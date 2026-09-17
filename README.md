# BÀI TẬP 1: KHỞI TẠO NỀN TẢNG NON-BLOCKING VÀ KẾT NỐI KAFKA CLUSTER

## 1. Mục tiêu
- Chuyển đổi ứng dụng từ mô hình chặn luồng (Blocking) sang mô hình phản ứng (Reactive) với Spring WebFlux.
- Thiết lập kết nối thành công tới cụm Apache Kafka đóng vai trò Event Broker.

## 2. Báo cáo cấu hình và Xử lý lỗi

### 2.1. Cấu hình WebClient
Đã khởi tạo Bean `WebClient` dùng chung trong ứng dụng (tại `WebClientConfig.java`), có cấu hình tự động ngắt kết nối (`Timeout`) nếu sau 5 giây không nhận được phản hồi.

### 2.2. BUG-01: Ứng dụng chạy Tomcat thay vì Netty
**Nguyên nhân:** File `pom.xml` có chứa thư viện `spring-boot-starter-webmvc`, dẫn đến Spring Boot tự động tải Tomcat lên thay thế cho Netty.
**Cách xử lý:** Đã tiến hành gỡ bỏ dependency `spring-boot-starter-webmvc` ra khỏi `pom.xml`. Chỉ giữ lại `spring-boot-starter-webflux`. Ứng dụng giờ đây sẽ chạy trên máy chủ Netty (port 8080).

### 2.3. BUG-02: Cấu hình Serializer cho Kafka Value
**Nguyên nhân:** Nếu sử dụng `StringSerializer` cho Value, việc truyền tải dữ liệu có cấu trúc phức tạp (Object Java) sẽ gặp khó khăn và phải chuyển đổi thủ công.
**Cách xử lý:** Đã cấu hình tường minh thuộc tính `value-serializer` thành `org.springframework.kafka.support.serializer.JsonSerializer` trong file `application.yml`:
```yaml
spring:
  kafka:
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

## 3. Hướng dẫn chạy
```bash
./mvnw spring-boot:run
```
Ứng dụng sẽ khởi động Netty server ở port `8080` và sẵn sàng nhận request theo chuẩn Reactive.
