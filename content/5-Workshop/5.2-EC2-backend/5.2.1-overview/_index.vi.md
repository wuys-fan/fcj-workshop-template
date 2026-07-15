---
title: "Tổng quan Workshop"
weight: 1
chapter: false
pre: " <b> 5.2.1 </b> "
---

# Tổng quan Workshop Backend

#### Bạn sẽ xây dựng gì

Trong phần này, bạn sẽ triển khai backend **RESTful API bằng Java Spring Boot** cho nền tảng **Pet Resort & Care System**. Ứng dụng sẽ được triển khai trực tiếp lên cụm máy chủ **Amazon EC2**, kết nối với Database và được phân luồng tự động qua **Application Load Balancer (ALB)**. 

Hệ thống API bao gồm **Swagger UI** để dễ dàng tài liệu hóa và kiểm thử các chức năng:

- 🏠 **Quản lý Dịch vụ & Sản phẩm** - Lấy danh sách sản phẩm, dịch vụ Spa, tối ưu tốc độ bằng Cache.
- 📅 **Hệ thống Đặt lịch (Booking)** - Xử lý logic đặt lịch hẹn an toàn và chính xác.
- 👤 **Quản lý Người dùng** - Xác thực và phân quyền (Khách hàng, Nhân viên, Admin).
- 🖼️ **Xử lý Media** - API hỗ trợ upload và lấy đường dẫn ảnh từ Amazon S3.
- 📝 **Swagger UI** - Giao diện trực quan để tương tác và test trực tiếp các endpoint API.

#### Tổng quan Kiến trúc

```text
┌─────────────────────────────────────────────────────┐
│           ReactJS Frontend (CloudFront / S3)        │
│          (https://d3uvhesft661gl.cloudfront.net/)  │
└──────────────────┬──────────────────────────────────┘
                   │ Gọi HTTPS API
                   ▼
┌─────────────────────────────────────────────────────┐
│       Application Load Balancer (ALB)               │
│         - Kiểm tra sức khỏe (Health Check)          │
│         - Phân luồng Traffic (Routing)              │
└──────────────────┬──────────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
┌──────────────┐      ┌──────────────┐
│ EC2 Instance │      │ EC2 Instance │
│(Auto Scaling)│      │(Auto Scaling)│
│              │      │              │
│ Spring Boot  │      │ Spring Boot  │
│ REST API     │      │ REST API     │
│ + Swagger    │      │ + Swagger    │
└──────┬───────┘      └──────┬───────┘
       │                     │
       └──────────┬──────────┘
        ┌─────────┴─────────┐
        ▼                   ▼
┌──────────────┐    ┌──────────────┐
│ ElastiCache  │    │  Amazon RDS  │
│  (Redis)     │    │   (MySQL)    │
│- Caching Data│    │- Lưu dữ liệu │
└──────────────┘    └──────────────┘
```

#### Công nghệ chính

| Công nghệ | Mục đích | Tại sao sử dụng? |
|-----------|----------|----------|
| **Java Spring Boot** | Framework Backend | Mạnh mẽ, bảo mật cao, chuẩn doanh nghiệp |
| **Swagger UI** | Tài liệu API | Tự động tạo Docs, dễ dàng test giao tiếp API |
| **Amazon EC2 & ALB** | Compute & Routing | Kiểm soát hoàn toàn máy chủ, tự động cân bằng tải |
| **Amazon RDS (MySQL)** | Cơ sở dữ liệu quan hệ | Ổn định, hỗ trợ Backup & Multi-AZ tự động |
| **Amazon ElastiCache** | In-memory Cache | Giảm tải cho Database, tăng tốc độ truy xuất dữ liệu |

#### Ước tính Chi phí

Hệ thống được thiết kế linh hoạt để có thể chạy trên môi trường thử nghiệm (Free Tier) hoặc mở rộng quy mô cho môi trường doanh nghiệp (Production).

**Môi trường thực hành (Free Tier):**
- EC2 `t3.micro`: 750 giờ/tháng (Miễn phí 1 instance)
- RDS `db.t3.micro`: 750 giờ/tháng (Miễn phí)
- ElastiCache `cache.t2.micro`: 750 giờ/tháng (Miễn phí)
- Chi phí duy nhất là Application Load Balancer: ~$16/tháng

**Môi trường Production (Theo cấu hình High-Performance đã thiết lập):**
- EC2 Instances kết hợp Auto Scaling: Trả theo lưu lượng sử dụng thực tế.
- RDS `db.m7g.large` & ElastiCache `cache.r7g.large`: Phục vụ hàng chục ngàn Request/giây (~$100 - $600/tháng).