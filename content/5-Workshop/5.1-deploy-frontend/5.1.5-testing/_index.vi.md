---
title: "Kiểm tra và xác minh"
weight: 5
chapter: false
pre: " <b> 5.1.5 </b> "
---

# Kiểm tra và xác minh

## 1. Kiểm tra chức năng

### 1.1 Kiểm tra tất cả các trang

- ✅ Trang chủ Pet Resort tải đúng
- ✅ Menu hiển thị danh sách Sản phẩm & Dịch vụ Spa
- ✅ Form đăng nhập / Đặt lịch hiển thị
- ✅ Navigation (thanh điều hướng) hoạt động
- ✅ Footer hiển thị

### 1.2 Kiểm tra Routing

Kiểm tra các URLs (thay bằng CloudFront Domain của bạn):
```text
[https://d3uvhesft661gl.cloudfront.net/](https://d3uvhesft661gl.cloudfront.net/)
[https://d3uvhesft661gl.cloudfront.net/services/tam-spa-thu-cung](https://d3uvhesft661gl.cloudfront.net/services/tam-spa-thu-cung)
[https://d3uvhesft661gl.cloudfront.net/login](https://d3uvhesft661gl.cloudfront.net/login)
```

Refresh mỗi trang con → Không bị lỗi 404 ✅

## 2. Kiểm tra CI/CD 

### 2.1 Kiểm tra Auto-Deploy

1. Thay đổi một dòng code text ở trang chủ trên Local
2. Commit và push code lên nhánh `main` bằng Github Desktop (hoặc Terminal)
3. Đợi báo Success, refresh lại trang web để thấy thay đổi mới

## 3. Dọn dẹp (Tùy chọn)

Nếu bạn chỉ đang thực hành và muốn xóa để tránh phát sinh chi phí:

1. Vào AWS Console → **CloudFront** → Disable Distribution và Delete.
2. Vào AWS Console → **S3** → Empty Bucket (Xóa hết file) và Delete Bucket.
3. Xác nhận xóa.