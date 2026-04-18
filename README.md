# ĐÁNH GIÁ VÀ SO SÁNH HIỆU NĂNG CÁC THUẬT TOÁN TCP CONGESTION CONTROL (NEWRENO VÀ CUBIC) TRONG MÔI TRƯỜNG MẠNG CÓ NGHẼN SỬ DỤNG NS-3

---

## 1. Giới thiệu

### 1.1 Bối cảnh

Trong các hệ thống mạng hiện đại, đặc biệt là Internet, việc truyền tải dữ liệu giữa các thiết bị đầu cuối phụ thuộc rất lớn vào giao thức TCP (Transmission Control Protocol). TCP đảm bảo truyền dữ liệu tin cậy thông qua cơ chế kiểm soát lỗi, kiểm soát luồng và đặc biệt là **điều khiển tắc nghẽn (Congestion Control)**.

Tắc nghẽn mạng xảy ra khi lưu lượng dữ liệu vượt quá khả năng xử lý của các thiết bị trung gian (router, switch), dẫn đến:
- Mất gói (packet loss)
- Tăng độ trễ (delay)
- Giảm thông lượng (throughput)

Do đó, việc thiết kế các thuật toán điều khiển tắc nghẽn hiệu quả là yếu tố then chốt để tối ưu hiệu năng mạng.

---

### 1.2 Lý do chọn đề tài

Hiện nay tồn tại nhiều thuật toán TCP Congestion Control khác nhau, mỗi thuật toán có cách tiếp cận và hiệu năng khác nhau trong các điều kiện mạng khác nhau.

Cụ thể:
- TCP NewReno: cải tiến từ Reno, phổ biến trong hệ thống truyền thống
- TCP Cubic: được sử dụng mặc định trong Linux, tối ưu cho mạng tốc độ cao

Tuy nhiên, chưa có cái nhìn trực quan về:
- Sự khác biệt hiệu năng giữa hai thuật toán
- Khả năng thích ứng trong môi trường có nghẽn
- Mức độ công bằng khi nhiều luồng cạnh tranh

Vì vậy, đề tài này được thực hiện nhằm làm rõ các vấn đề trên thông qua mô phỏng bằng NS-3.

---

### 1.3 Mục tiêu nghiên cứu

- Xây dựng mô hình mạng có liên kết nghẽn (bottleneck)
- Triển khai TCP NewReno và TCP Cubic
- Đánh giá các chỉ số:
  - Throughput
  - Delay
  - Packet loss
  - Fairness
- So sánh và đưa ra nhận xét

---

## 2. Cơ sở lý thuyết

---

### 2.1 Khái niệm tắc nghẽn mạng

Tắc nghẽn xảy ra khi:

Tổng lưu lượng vào > khả năng xử lý của router

Ví dụ thực tế:
- 10 máy cùng tải file qua 1 đường truyền 5Mbps
→ Router không xử lý kịp → queue đầy → drop packet

---

### 2.2 Nguyên lý TCP Congestion Control

TCP sử dụng biến:

- **cwnd (Congestion Window)**: số lượng dữ liệu tối đa có thể gửi

---

#### Các pha chính:

### a. Slow Start
- cwnd tăng theo cấp số nhân
- mục tiêu: nhanh chóng tận dụng băng thông

Ví dụ:
- cwnd = 1 → 2 → 4 → 8 → 16

---

### b. Congestion Avoidance
- cwnd tăng tuyến tính
- tránh gây nghẽn

---

### c. Khi phát hiện mất gói

TCP coi mất gói là dấu hiệu tắc nghẽn:

- Giảm cwnd
- Giảm tốc độ gửi

---

## 2.3 TCP NewReno

### Đặc điểm

- Cải tiến từ TCP Reno
- Xử lý tốt multiple packet loss

---

### Cơ chế hoạt động

Khi nhận duplicate ACK:
- Retransmit packet
- Giảm cwnd (thường chia đôi)

---

### Ví dụ

Giả sử:
- cwnd = 16

Khi mất gói:
→ cwnd = 8

→ tăng lại từ từ

---

### Ưu điểm

- Ổn định
- Fairness tốt

### Nhược điểm

- Tăng trưởng chậm
- Không tận dụng tốt mạng tốc độ cao

---

## 2.4 TCP Cubic

### Ý tưởng chính

Thay vì tăng tuyến tính, Cubic dùng hàm bậc ba:

cwnd(t) = C(t - K)^3 + Wmax

---

### Ý nghĩa

- Khi xa điểm nghẽn → tăng nhanh
- Gần điểm nghẽn → tăng chậm lại

---

### Ví dụ trực quan

- NewReno: tăng đều từng bước
- Cubic: tăng nhanh → chậm lại → ổn định

---

### Ưu điểm

- Tận dụng tốt băng thông lớn
- Phù hợp mạng backbone, data center

---

### Nhược điểm

- Có thể gây mất gói nhiều hơn
- Fairness không cao

---

## 3. Ứng dụng thực tiễn

---

### 3.1 TCP NewReno

Được sử dụng trong:
- Hệ thống cũ
- Mạng có độ ổn định cao
- Môi trường yêu cầu fairness

Ví dụ:
- Mạng doanh nghiệp nội bộ
- Hệ thống truyền dữ liệu nhỏ

---

### 3.2 TCP Cubic

Được sử dụng trong:
- Linux (mặc định)
- Data center
- Cloud computing

Ví dụ:
- Truyền video (YouTube, Netflix)
- Download file lớn
- Backup dữ liệu

---

### 3.3 Ý nghĩa so sánh

Giúp:
- Chọn thuật toán phù hợp
- Tối ưu hệ thống mạng
- Hiểu rõ hành vi TCP

---

## 4. Mô hình mô phỏng (NS-3)

---

### 4.1 Kiến trúc mạng
``
Multiple Senders → Router1 → Router2 → Receiver
``
- Bottleneck nằm giữa Router1 và Router2

---

### 4.2 Ý nghĩa bottleneck

Giả lập:

- Đường truyền yếu
- Router quá tải
- Điều kiện mạng thực tế

---

### 4.3 Cấu hình từ code

- Access link: 100Mbps, 2ms
- Bottleneck: 5Mbps, 50ms
- Số flow: thay đổi

---

### 4.4 Sinh lưu lượng

- BulkSend → gửi liên tục
- PacketSink → nhận dữ liệu

---

## 5. Phương pháp đánh giá

---

### 5.1 Throughput

Lượng dữ liệu nhận được:

Throughput = (rxBytes × 8) / time

---

### 5.2 Delay

Delay trung bình:

Delay = delaySum / packets

---

### 5.3 Packet Loss

Số gói bị mất trong quá trình truyền

---

### 5.4 Fairness (Jain Index)

Fairness = (Σxi)^2 / (n Σxi^2)

---

### Ví dụ

3 flow:
- 5 Mbps, 5 Mbps, 5 Mbps → fairness = 1 (tốt)
- 10 Mbps, 0 Mbps, 0 Mbps → fairness thấp

---

## 6. Phân tích hoạt động hệ thống

---

### 6.1 Khi số flow tăng

- Các flow cạnh tranh băng thông
- Xuất hiện nghẽn tại bottleneck

---

### 6.2 Hành vi NewReno

- Giảm cwnd mạnh khi mất gói
- Tăng lại chậm

→ ổn định nhưng chậm

---

### 6.3 Hành vi Cubic

- Tăng nhanh
- Dễ chiếm băng thông

→ throughput cao nhưng có thể unfair

---

## 7. Kết quả kỳ vọng

---

### 7.1 Throughput

- Cubic > NewReno

---

### 7.2 Delay

- Cubic ≥ NewReno

---

### 7.3 Packet Loss

- Cubic ≥ NewReno

---

### 7.4 Fairness

- NewReno > Cubic

---

## 8. Kết luận

- Không có thuật toán tốt nhất tuyệt đối
- Lựa chọn phụ thuộc:
  - Loại mạng
  - Yêu cầu hệ thống

---

## 9. Hướng phát triển

- So sánh với TCP BBR
- Áp dụng AI tối ưu congestion control
- Thử nghiệm trên mạng thực

---

## 10. Tài liệu tham khảo

- NS-3 Official Documentation
- RFC 5681
- Linux TCP Cubic Paper
- GeeksforGeeks TCP Congestion Control
