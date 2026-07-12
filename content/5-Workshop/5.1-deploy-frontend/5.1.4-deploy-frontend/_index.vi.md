---
title: "Deploy Frontend lên S3 & CloudFront"
weight: 4
chapter: false
pre: " <b> 5.1.4 </b> "
---

# Deploy Frontend lên S3 & CloudFront

Theo đúng kiến trúc đã thiết kế, nhóm sẽ không sử dụng các dịch vụ đóng gói sẵn. Thay vào đó, chúng ta sẽ deploy giao diện tĩnh lên **Amazon S3**, sử dụng **Amazon CloudFront** làm CDN để tăng tốc độ phân phối toàn cầu và tích hợp **AWS WAF** để bảo mật.

## 1. Build mã nguồn (Local)

Trước khi đưa lên AWS, chúng ta cần đóng gói (build) mã nguồn ReactJS thành các file tĩnh (HTML, CSS, JS).

1. Mở Terminal tại thư mục `frontend`:
```bash
cd frontend
npm run build
```
2. Sau khi chạy xong, một thư mục `build` (hoặc `dist`) sẽ được tạo ra. Đây chính là thư mục chúng ta sẽ upload lên AWS.

![Thành quả build ReactJS](/images/1-Worklog/react_build_output.png)

---

## 2. Tạo và cấu hình Amazon S3 Bucket

### 2.1 Tạo Bucket
1. Truy cập AWS Console → Tìm kiếm **S3**
2. Click **Create bucket**
3. **Bucket name**: `pet-resort-frontend-prod` (Tên phải là duy nhất)
4. **AWS Region**: `ap-southeast-1`
5. Bỏ check phần **Block all public access** (Vì đây là web public).
6. Click **Create bucket**.

![Create S3 Bucket](/images/5-Workshop/s3-create.png)

### 2.2 Upload mã nguồn và Bật Website Hosting
1. Truy cập vào Bucket vừa tạo.
2. Click **Upload** → Kéo thả toàn bộ nội dung trong thư mục `build` (hoặc `dist`) ở bước 1 vào đây → Click **Upload**.
3. Chuyển sang tab **Properties**.
4. Kéo xuống phần **Static website hosting** → Click **Edit**.
5. Chọn **Enable**.
6. Index document: `index.html`.
7. Click **Save changes**.

{{% notice info %}}
📝 **Lưu ý**: Cần cấp quyền đọc public cho Bucket Policy để người dùng có thể truy cập nội dung.
{{% /notice %}}

---

## 3. Cấu hình Amazon CloudFront & WAF

Để web load nhanh hơn và an toàn hơn, chúng ta đặt CloudFront chắn trước S3.

### 3.1 Tạo Distribution
1. Truy cập AWS Console → Tìm kiếm **CloudFront** → **Create Distribution**.
2. **Origin domain**: Dán link **S3 Static Website Hosting Endpoint** (Ví dụ: `pets-hop-frontend.s3-website-ap-southeast-1.amazonaws.com`) của bucket vừa tạo.
   * *Lưu ý*: Không chọn endpoint gợi ý tự động của S3 API để tránh gặp lỗi phân quyền hoặc định tuyến.
3. **Viewer protocol policy**: Chọn **Redirect HTTP to HTTPS** (Bắt buộc để bảo mật).
4. **Allowed HTTP methods**: Chọn **GET, HEAD**.
5. **Cache policy**: Chọn **CachingOptimized**.
6. **Web Application Firewall (WAF)**: Chọn **Enable security protections** để tự động chặn các request độc hại.
7. Click **Create distribution**.

![Tạo CloudFront Distribution](/images/5-Workshop/cloudfront-create.png)

### 3.2 Lấy URL truy cập
Sau khoảng 3-5 phút, CloudFront sẽ deploy xong. 
Bạn copy đường dẫn tại cột **Distribution domain name** (Ví dụ: `d1x2y3z4.cloudfront.net`) và truy cập trên trình duyệt để kiểm tra web.

---

## 4. Xử lý CI/CD tự động bằng GitHub Actions (Tuỳ chọn)

Để không phải upload tay mỗi lần sửa code, nhóm thiết lập **GitHub Actions** tự động đẩy code lên S3 và xóa cache CloudFront.

Tạo file `.github/workflows/deploy.yml` trong repo:

```yaml
name: Deploy Frontend to S3
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Build
      run: |
        cd frontend
        npm install
        npm run build
    - name: Deploy to S3
      run: aws s3 sync frontend/build/ s3://pet-resort-frontend-prod --delete
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 5. Xử lý sự cố (Troubleshooting)

### Vấn đề 1: Lỗi 403 Access Denied khi truy cập
**Giải pháp**: Kiểm tra lại S3 Bucket Policy. Đảm bảo bạn đã thêm đoạn policy cho phép `s3:GetObject` từ mọi nguồn (`*`) hoặc cấu hình CloudFront OAC chính xác.

### Vấn đề 2: Lỗi 404 khi F5 (Refresh) trang hoặc vào link con
Vì ReactJS là Single Page Application (SPA), AWS S3 không hiểu các route ảo.
**Giải pháp**: 
1. Vào CloudFront → Chọn Distribution → Tab **Error pages**.
2. Click **Create custom error response**.
3. HTTP error code: `404: Not Found`.
4. Customize error response: `Yes`.
5. Response page path: `/index.html`.
6. HTTP Response code: `200: OK`.

![CloudFront Error Pages](/images/5-Workshop/cloudfront-error.png)

---

## Bước tiếp theo

Tiếp tục với [Kiểm thử Ứng dụng (Testing)](../5.1.5-testing/).