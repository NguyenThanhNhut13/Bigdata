# Chào thầy, em tên Nguyễn Văn A, em xin tiếp tục demo phần Scaling và Load Balancing trong hệ thống Docker Swarm.

Hệ thống của chúng ta hiện tại bao gồm **15 worker services**, **10 rng services**, **10 hasher services**, **5 webui services** và **2 redis services**, được triển khai trên **Docker Swarm** với các manager node đã thiết lập **High Availability (HA)** để đảm bảo tính sẵn sàng cao. Giả sử hệ thống đang chạy ổn định, nhưng vào giờ cao điểm, lượng người dùng truy cập tăng đột biến, khiến **CPU và RAM** của các node bị quá tải. Làm sao để hệ thống tự động mở rộng và phân phối tải hiệu quả?

## Phần 1: Scaling
Docker Swarm hỗ trợ **manual scaling** thông qua lệnh `docker service scale`, nhưng trong thực tế, chúng ta có thể **tự động hóa** quy trình này bằng các công cụ như **Prometheus** và **AlertManager** để theo dõi tài nguyên và tự động điều chỉnh số lượng replicas khi cần.

Để minh họa, đầu tiên, em sẽ **scale service webui xuống còn 1 replica** để mô phỏng tình huống hệ thống chạy với tài nguyên hạn chế:

```bash
sudo docker service scale webui=1
```

Tiếp theo, giả sử lượng truy cập tăng đột biến, em sẽ **scale service webui lên 5 replicas** để đáp ứng nhu cầu:

```bash
sudo docker service scale webui=5
```

Sau khi chạy lệnh này, Docker Swarm sẽ **tự động tạo thêm các replicas mới** cho `webui` và phân bổ chúng trên các node trong swarm. Để kiểm tra tổng quan số lượng replicas hiện tại, em sử dụng lệnh:

```bash
sudo docker service ls
```

Như thầy thấy, cột `REPLICAS` cho service `webui` hiển thị `5/5`, nghĩa là **5 replicas đã được triển khai thành công**. Điều này giúp hệ thống mở rộng linh hoạt khi tài nguyên bị quá tải.

## Phần 2: Load Balancing
Sau khi scaling, làm sao để hệ thống **phân phối request đồng đều** đến các replicas để tránh quá tải một node? Docker Swarm sử dụng **Routing Mesh** để giải quyết vấn đề này. Routing Mesh cho phép tất cả các node trong swarm **lắng nghe trên một cổng cụ thể** (ví dụ: cổng 80 cho webui) và **chuyển tiếp request đến các replicas** của service tương ứng, **bất kể replicas đó chạy trên node nào**.

Để kiểm tra cách các replicas được phân bố, em sử dụng lệnh:

```bash
sudo docker service ps webui --filter "desired-state=running"
```

Như thầy thấy, **5 replicas của webui được phân bổ trên các node khác nhau** trong swarm. Docker Swarm **tự động tối ưu hóa việc phân bố** này để đảm bảo sử dụng tài nguyên hiệu quả và duy trì tính sẵn sàng cao.

## Phần 3: Minh họa Load Balancing qua hệ thống giả lập đào coin
Để trực quan hóa load balancing, em đã thiết lập một hệ thống giả lập **đào coin**:

1. **15 worker services** gửi yêu cầu lấy dữ liệu ngẫu nhiên từ **10 rng services**.
2. Dữ liệu sau đó được gửi đến **10 hasher services** để băm thành mã hash.
3. Kết quả được lưu vào **2 redis services**.
4. **5 webui services** hiển thị các mã hash này trên giao diện web, cập nhật mỗi giây.

Bây giờ, em sẽ truy cập vào hai địa chỉ IP:

- Một **worker node không chạy service webui**.
- Một **node chạy service webui**.

[Thực hiện demo]

Như thầy thấy, khi truy cập vào **worker node không chạy webui**, em vẫn xem được giao diện web với các mã hash cập nhật liên tục. Điều này chứng tỏ **Routing Mesh đã chuyển tiếp request** đến một trong 5 replicas của webui trên các node khác. Khi truy cập vào **node chạy webui**, tải vẫn được **phân phối đều**, không có node nào bị quá tải. Đây là cách **load balancing hoạt động trong Docker Swarm**.

→ Kết quả: Node chạy WebUI sẽ **xử lý ít mã hash hơn**, vì Docker Swarm đã kích hoạt **load balancing**, giúp phân bổ request đều hơn giữa các nodes.

## Kết luận
✅ **Scaling** giúp hệ thống mở rộng số lượng replicas khi tài nguyên bị quá tải, đảm bảo đáp ứng nhu cầu người dùng.

✅ **Load Balancing với Routing Mesh** phân phối request đồng đều giữa các replicas, ngăn chặn tình trạng quá tải trên một node cụ thể.

Cả hai tính năng này đều là yếu tố quan trọng để xây dựng **một hệ thống có khả năng mở rộng và chịu tải cao trong Docker Swarm**.

Để hiểu rõ hơn về **cách giám sát hệ thống này hoạt động**, em xin mời bạn [tên bạn tiếp theo] trình bày phần tiếp theo về **giám sát và logging**. Em xin hết.
