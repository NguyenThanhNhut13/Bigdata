
# Triển khai Scaling và Load Balancing trong Docker Swarm

## Giới thiệu
Chào thầy, em tên **Nguyễn Văn A**, em xin tiếp tục demo phần **Scaling** và **Load Balancing** trong hệ thống Docker Swarm.

Giả sử hệ thống đang chạy ổn định, nhưng đột nhiên có một lượng lớn người dùng truy cập, CPU và RAM tăng cao. Làm sao để hệ thống có thể tự động mở rộng?

Docker Swarm hỗ trợ **manual scaling** mặc định, nhưng có thể kết hợp với các công cụ như **Prometheus + AlertManager** hoặc tự động bằng **script**.

---

## 1. Scaling
### Giảm số lượng replicas
Đầu tiên, em sẽ scale service **webui** xuống **1**:

```bash
sudo docker service scale webui=1
```

### Tăng số lượng replicas
Sau đó, giả sử có lượng lớn người dùng truy cập thì em sẽ scale service **webui** lên **5**:

```bash
sudo docker service scale webui=5
```

Sau khi chạy lệnh này, hệ thống sẽ tự động tạo ra các **replica** của service **webui**.

### Kiểm tra số lượng replicas
Để kiểm tra lại số lượng replica sau khi scale, em sử dụng lệnh:

```bash
sudo docker service ps webui
```

Kết quả hiển thị số lượng replica của **webui** đã tăng lên **5**.

> ✅ **Scaling giúp mở rộng số lượng replicas khi tài nguyên bị quá tải.**

---

## 2. Load Balancing
Sau khi scaling, làm sao hệ thống có thể phân phối request đồng đều đến các container để tránh quá tải một node?

Docker Swarm có **Routing Mesh**, giúp tự động phân phối request đến tất cả các **replicas** đang chạy.

### Kiểm tra phân phối của Load Balancer
Sau khi scale **webui** lên **5**, em sẽ xem nó phân bổ thế nào đến các node bằng lệnh:

```bash
sudo docker service ps webui --filter "desired-state=running"
```

Như thầy đã thấy, Docker Swarm **tự động phân phối** các replica của **webui** đến các node.

### Mô phỏng hệ thống Load Balancing
Để trực quan hóa **Load Balancing**, em sẽ tạo một hệ thống giả lập **đào coin**, trong đó:

- **Worker service** lấy dữ liệu random từ **service rng**
- **Service hasher** băm dữ liệu thành mã **hash** và lưu vào **Redis**
- **WebUI** hiển thị mã **hash** theo từng giây

### Kiểm tra trên các worker nodes
Em sẽ truy cập vào 2 địa chỉ IP:
- **Worker node không chạy webui**
- **Worker node chạy webui**

Như thầy thấy, **node chạy webui nhận ít mã hash hơn**, vì **Docker Swarm đã kích hoạt Load Balancing**.

> ✅ **Load Balancing đảm bảo request được phân phối đều giữa các replicas.**

---

## Kết luận
- **Scaling** giúp hệ thống mở rộng khi có lượng lớn người dùng truy cập.
- **Load Balancing** giúp cân bằng tải giữa các **replicas** để tránh quá tải cho một node cụ thể.

Để giám sát hệ thống này hoạt động thế nào, em xin mời bạn tiếp theo trình bày ạ. Em xin hết. 🙏
