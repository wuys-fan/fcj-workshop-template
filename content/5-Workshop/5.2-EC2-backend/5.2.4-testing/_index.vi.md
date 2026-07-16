---
title: "Kiểm tra và Tích hợp"
weight: 4
chapter: false
pre: " <b> 5.2.4 </b> "
---

# Kiểm tra API và Tích hợp Hệ thống

Bây giờ chúng ta sẽ kiểm thử backend Spring Boot vừa triển khai, xác minh kết nối cơ sở dữ liệu và tích hợp API với Frontend (ReactJS).

## Bước 1: Lấy URL của Application Load Balancer (ALB)

1. Truy cập **AWS Console** → **EC2** → Chọn **Load Balancers** ở menu trái.
2. Chọn ALB `petshop-alb` mà bạn đã tạo ở bài trước.
3. Trong tab **Description**, copy đường dẫn ở mục **DNS name** (Ví dụ: `petshop-alb-123456789.ap-southeast-1.elb.amazonaws.com`).

![ALB DNS Name](/images/5-Workshop/alb-dns.png)

## Bước 2: Kiểm tra API qua Swagger UI

Mở trình duyệt và truy cập vào đường dẫn Swagger mặc định của Spring Boot thông qua ALB:
```text
http://<DNS-name-của-ALB>/swagger-ui/index.html
```

Bạn sẽ thấy giao diện **Swagger UI** xuất hiện với đầy đủ các Endpoints của Pet Resort (như Products, Services, Bookings, Users). Thử thực thi (Execute) một API `GET /api/products` để đảm bảo API trả về mã `200 OK`.

![Swagger UI Backend](/images/5-Workshop/swagger-ui.png)

## Bước 3: Xác minh kết nối Database & Cache

Trong quá trình thiết lập cơ sở dữ liệu **Amazon RDS (MySQL)**, bạn sẽ thấy bảng tóm tắt chi phí ước tính hàng tháng:
![RDS Database Estimated Costs](/images/1-Worklog/rds_estimated_costs.png)

Sau khi khởi tạo, kiểm tra xem cơ sở dữ liệu có chuyển sang trạng thái **Creating** và sau đó là **Available** hay không:
![RDS Instance Launching](/images/1-Worklog/rds_creating.png)

Đảm bảo cấu hình kết nối của database được thiết lập chính xác với các thông tin endpoint và subnet nội bộ:
![RDS Connectivity Details](/images/1-Worklog/rds_connectivity.png)

Để tăng tốc độ truy vấn bằng cache, thiết lập và xem lại thông số của **Amazon ElastiCache (Valkey)**:
![ElastiCache Valkey Settings](/images/1-Worklog/elasticache_settings.png)

Xác nhận rằng cụm cache ElastiCache đã được khởi tạo thành công và chuyển sang trạng thái khả dụng:
![ElastiCache Valkey Status](/images/1-Worklog/elasticache_creating.png)

Thay vì chỉ tin vào API, chúng ta sẽ kiểm tra trực tiếp xem dữ liệu đã thực sự được ghi vào **Amazon RDS (MySQL)** hay chưa.

1. Vào AWS Console → **Systems Manager** → **Session Manager**.
2. Start session để chui vào bên trong máy chủ EC2 `petshop-backend-server`.
3. Dùng lệnh kết nối MySQL để kiểm tra các bảng dữ liệu đã được Hibernate/JPA tự động tạo ra.

```bash
mysql -h petshop-database-1.xxxx.ap-southeast-1.rds.amazonaws.com -u admin -p
SHOW DATABASES;
USE petshop_db;
SHOW TABLES;
```

Kết quả trả về chính xác các bảng nghiệp vụ của hệ thống như `bookings`, `pets`, `spa_services` chứng tỏ kết nối từ EC2 đến RDS đã thành công mỹ mãn!

## Bước 4: Tích hợp API vào Frontend (ReactJS)

Bây giờ bạn cần cập nhật URL Backend mới cho Frontend.

1. Mở mã nguồn ReactJS (Frontend) trên máy tính bằng VS Code.
2. Mở file biến môi trường (ví dụ `.env` hoặc `.env.production`).
3. Cập nhật đường dẫn `VITE_API_URL` trỏ về DNS Name của ALB:

```env
VITE_API_URL=http://petshop-alb-123456789.ap-southeast-1.elb.amazonaws.com/api
```

4. Cấu hình lại file biến môi trường, lưu file, commit code và push lên GitHub để hệ thống CI/CD tự động deploy bản cập nhật lên S3/CloudFront.

## Bước 5: Kiểm tra lỗi CORS & Mixed Content

Mở trang web Frontend đang host trên CloudFront của bạn, nhấn `F12` mở tab **Network** và **Console** để theo dõi:

- ✅ **Thành công (200 OK):** Dữ liệu sản phẩm, dịch vụ được load lên giao diện từ ALB.
- ❌ **Lỗi CORS:** Backend chặn request do khác domain. Bạn cần vào lại code Spring Boot (class `CorsConfig`), thêm domain CloudFront vào danh sách `allowedOrigins` và deploy lại.
- ❌ **Lỗi Mixed Content (HTTP/HTTPS):** Frontend chạy trên `HTTPS` nhưng ALB lại dùng `HTTP`. 
  - **Cách khắc phục thực tế:** Cấu hình Listener `HTTPS (443)` cho ALB và đính kèm chứng chỉ SSL miễn phí từ AWS Certificate Manager (ACM). 

## Giám sát Ứng dụng qua CloudWatch

Nếu API không phản hồi, hãy đọc Logs:
1. Vào **AWS Console** → **CloudWatch** → **Log groups**.
2. Tìm Log Group của EC2 (nếu đã cài CloudWatch Agent) hoặc kết nối SSH vào EC2 đọc file `app.log`.
3. Kiểm tra các lỗi liên quan đến OutOfMemory hoặc kết nối RDS/ElastiCache bị từ chối do Security Group.