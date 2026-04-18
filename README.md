```md id="tcp-ns3-report-22521"
# ĐÁNH GIÁ HIỆU NĂNG TCP NEWRENO VÀ TCP CUBIC TRONG MẠNG CÓ NGHẼN SỬ DỤNG NS-3

## 1. Giới thiệu

Trong mạng máy tính, hiện tượng tắc nghẽn xảy ra khi lưu lượng dữ liệu vượt quá khả năng xử lý của mạng, dẫn đến mất gói, tăng độ trễ và giảm hiệu năng truyền tải.

TCP sử dụng các thuật toán điều khiển tắc nghẽn (Congestion Control) để điều chỉnh tốc độ gửi dữ liệu. Hai thuật toán phổ biến là:

- TCP NewReno: cải tiến từ Reno, xử lý tốt multiple packet loss
- TCP Cubic: tối ưu cho mạng tốc độ cao, sử dụng hàm cubic

Đề tài này sử dụng NS-3 để mô phỏng và so sánh hiệu năng của hai thuật toán trong điều kiện mạng có bottleneck.

---

## 2. Mô hình mô phỏng

### 2.1 Kiến trúc mạng

Mô hình được xây dựng như sau:

```

Sender1 
Sender2 ----> Router1 ----> Router2 ----> Receiver
...      /

````id="topo-ns3"

- N sender tạo nhiều luồng TCP
- Router1 ↔ Router2 là bottleneck
- Receiver nhận dữ liệu

---

### 2.2 Cấu hình mạng trong code

#### 2.2.1 Node

```cpp
NodeContainer senders, routers, receiver;
senders.Create(numFlows);
routers.Create(2);
receiver.Create(1);
````

* `numFlows`: số luồng TCP (có thể thay đổi bằng CLI)

---

#### 2.2.2 Link

**Access link (không nghẽn):**

```cpp
100 Mbps, 2 ms
```

**Bottleneck link:**

```cpp
rate = bottleneckRate (default: 5Mbps)
delay = bottleneckDelay (default: 50ms)
```

---

#### 2.2.3 Địa chỉ IP

* Sender → Router1: `10.1.x.0/24`
* Bottleneck: `10.1.100.0/24`
* Router2 → Receiver: `10.1.200.0/24`

---

### 2.3 Cấu hình TCP

```cpp
Config::SetDefault("ns3::TcpL4Protocol::SocketType",
    TypeIdValue(TypeId::LookupByName("ns3::" + tcpVariant)));
```

Có thể chọn:

* TcpNewReno
* TcpCubic

---

## 3. Sinh lưu lượng

### 3.1 Ứng dụng gửi

```cpp
BulkSendHelper src("ns3::TcpSocketFactory", ...);
src.SetAttribute("MaxBytes", UintegerValue(0));
```

* Gửi dữ liệu **liên tục (infinite)**
* Bắt đầu từ 1s → 20s

---

### 3.2 Ứng dụng nhận

```cpp
PacketSinkHelper sink(...)
```

* Nhận toàn bộ dữ liệu TCP

---

## 4. Thu thập dữ liệu

Sử dụng:

```cpp
FlowMonitorHelper fm;
```

Các chỉ số được tính:

---

### 4.1 Throughput

```cpp
throughput = (rxBytes * 8) / time
```

(đơn vị Mbps)

---

### 4.2 Delay

```cpp
delay = delaySum / rxPackets
```

---

### 4.3 Packet Loss

```cpp
lostPackets
```

---

### 4.4 Fairness Index (Jain)

Công thức:

Fairness = (Σxi)^2 / (n * Σxi^2)

Trong code:

```cpp
double fairness = (sum * sum) / (throughputs.size() * sumSq);
```

---

## 5. Kịch bản thí nghiệm

Chạy chương trình với các tham số:

```bash
./ns3 run "scratch/tcp-comparison --tcpVariant=TcpNewReno --rate=5Mbps --delay=50ms --flows=2"
```

Các biến thay đổi:

* TCP variant: NewReno / Cubic
* Bottleneck bandwidth: 5Mbps, 10Mbps
* Delay: 20ms – 100ms
* Số flows: 1, 2, 5

---

## 6. Phân tích code quan trọng

### 6.1 Bottleneck tạo nghẽn

```cpp
PointToPointHelper bottleneck;
bottleneck.SetDeviceAttribute("DataRate", StringValue(bottleneckRate));
```

👉 Đây là nơi **cố tình tạo nghẽn mạng**

---

### 6.2 Multi-flow competition

```cpp
senders.Create(numFlows);
```

👉 Các luồng cạnh tranh băng thông → đánh giá fairness

---

### 6.3 Lọc flow không hợp lệ

```cpp
if (t.sourceAddress == Ipv4Address("10.1.200.2"))
    continue;
```

👉 Loại bỏ flow từ receiver

---

### 6.4 Tính throughput từng flow

```cpp
double throughput = (flow.second.rxBytes * 8.0 / time) / 1024 / 1024;
```

---

## 7. Kết quả và nhận xét

### 7.1 Throughput

* TCP Cubic thường đạt throughput cao hơn
* TCP NewReno tăng trưởng chậm hơn

---

### 7.2 Delay

* Cubic có delay cao hơn khi aggressive
* NewReno ổn định hơn

---

### 7.3 Packet Loss

* Cubic dễ gây mất gói khi tăng nhanh
* NewReno giảm cwnd mạnh → ít loss hơn

---

### 7.4 Fairness

* NewReno → công bằng hơn
* Cubic → có thể chiếm băng thông

---

## 8. Kết luận

* TCP NewReno:

  * Ổn định
  * Fairness tốt
  * Phù hợp mạng nhỏ

* TCP Cubic:

  * Throughput cao
  * Phù hợp mạng tốc độ cao
  * Trade-off delay và loss

---

## 9. Hạn chế

* Chưa xét:

  * Queue discipline (RED, CoDel)
  * TCP BBR
  * Mạng thực tế

---

## 10. Hướng phát triển

* Thêm TCP BBR
* So sánh nhiều thuật toán hơn
* Áp dụng ML để adaptive congestion control

---

## 11. Tài liệu tham khảo

* NS-3 Documentation
* GeeksforGeeks - TCP Congestion Control
* RFC 5681

```



