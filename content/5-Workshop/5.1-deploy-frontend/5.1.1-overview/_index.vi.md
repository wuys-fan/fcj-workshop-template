---
title: "Tổng quan Workshop"
weight: 1
chapter: false
pre: " <b> 5.1.1 </b> "
---

# Tổng quan Workshop

#### Mục tiêu Workshop

Sau khi hoàn thành workshop này, bạn sẽ tự tay xây dựng được một hệ thống trên AWS với các kỹ năng:
- ✅ Xây dựng hệ thống web chịu tải tốt, không bị sập khi đông khách.
- ✅ Chia tách mạng (VPC, Subnet) để giấu kín mã nguồn và dữ liệu, chống hacker.
- ✅ Dùng ElastiCache (Redis) để web tải mượt và nhanh hơn.
- ✅ Lưu trữ ảnh và giao diện tĩnh với chi phí cực rẻ trên Amazon S3.
- ✅ Cài đặt tự động gửi Email cảnh báo khi server gặp sự cố.

#### Pet Resort & Care System - Tính năng cơ bản

Trong workshop này, chúng ta sẽ triển khai nền tảng Pet Resort & Care System bao gồm các module:
- 🏠 **Trang chủ & Sản phẩm**: Hiển thị danh mục sản phẩm, dịch vụ thú cưng.
- 📅 **Hệ thống Booking**: Chức năng đặt lịch Spa an toàn, chính xác.
- 🖼️ **Quản lý Media**: Nơi nhân viên tải lên và lưu trữ hình ảnh của thú cưng.

#### Kiến trúc triển khai tổng quan

Dưới đây là luồng đi của hệ thống từ lúc khách hàng bấm vào website cho đến khi dữ liệu được lưu lại:

```text
┌──────────────────────────────┐
│           Người dùng         │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│      CloudFront & WAF        │ (Tường lửa & Tăng tốc độ tải)
└──────┬───────────────┬───────┘
       │               │
       ▼               ▼
┌────────────┐   ┌─────────────┐
│ S3 Frontend│   │ Load Balancer (Bộ phân luồng)
│ (Giao diện)│   └──────┬──────┘
└────────────┘          │
                        ▼
                 ┌─────────────┐
                 │ EC2 Backend │ (Xử lý code & Tự động tăng giảm máy chủ)
                 └─┬────┬────┬─┘
                   │    │    │
          ┌────────┘    │    └────────┐
          ▼             ▼             ▼
  ┌────────────┐ ┌─────────────┐ ┌───────────────┐
  │ ElastiCache│ │  Amazon RDS │ │   S3 Media    │
  │ (Cache)    │ │  (Database) │ │(Lưu ảnh nội bộ)
  └────────────┘ └─────────────┘ └───────────────┘
```

#### Quy trình xử lý và Bảo mật (Security & Flow)

- **Tiếp nhận & Bảo vệ:** Yêu cầu từ khách hàng sẽ đi qua màng lọc WAF để chặn các truy cập xấu, sau đó CloudFront sẽ giúp tải trang web nhanh hơn.
- **Phân luồng:** Giao diện website được lấy từ kho S3. Còn các thao tác như "Đặt lịch" hay "Mua hàng" sẽ được đưa qua cổng Load Balancer để chia đều việc cho các máy chủ bên trong.
- **Xử lý:** Máy chủ EC2 nhận yêu cầu sẽ ưu tiên tìm dữ liệu trong Cache (Redis) cho nhanh. Nếu không có, nó mới xuống Database (RDS MySQL) để đọc hoặc lưu dữ liệu mới.
- **Lưu ảnh an toàn:** Thao tác tải ảnh thú cưng lên sẽ đi qua một "đường hầm" mạng nội bộ (VPC Endpoint) để vào kho S3 Media, không đi ra ngoài Internet để đảm bảo an toàn.
- **Giám sát:** Dịch vụ CloudWatch sẽ liên tục theo dõi hệ thống. Nếu có máy chủ nào bị lỗi, nó sẽ tự động gửi Email báo cho đội ngũ quản trị.

#### Chi phí dự kiến

Nhờ tận dụng khéo léo gói AWS Free Tier, chi phí duy trì hệ thống này được giảm xuống mức tối thiểu:

- ✅ **Máy chủ EC2 & Caching (Redis):** Miễn phí 750 giờ/tháng (Free Tier).
- ✅ **Database RDS:** Miễn phí 750 giờ/tháng (Free Tier).
- ✅ **Lưu trữ S3 & CloudFront:** Nằm trong định mức miễn phí.

**Chi phí ước tính thực tế:** Khoảng **$50 - $70/tháng** cho môi trường thực hành. Chi phí này chủ yếu để duy trì 2 dịch vụ mạng: NAT Gateway và Load Balancer. Kiến trúc Multi-AZ đầy đủ (như mô tả trong Proposal) có chi phí khoảng **~$168/tháng**.

{{% notice tip %}}
💡 **Mẹo:** Trong lúc thực hành, nếu chưa cần chạy thật, bạn có thể tạm thời tắt NAT Gateway và chạy Database ở chế độ máy đơn (Single-AZ) để tiết kiệm tối đa tiền trong thẻ.
{{% /notice %}}

#### Bước tiếp theo

Bắt đầu với [Thiết lập Mạng (VPC & Subnets)](#) để xây dựng lớp bảo vệ ngoài cùng cho hệ thống.