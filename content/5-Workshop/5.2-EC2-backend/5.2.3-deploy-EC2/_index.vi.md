---
title: "Triển khai lên Amazon EC2"
weight: 3
chapter: false
pre: " <b> 5.2.3 </b> "
---

# Triển khai Spring Boot lên Amazon EC2 & ALB

Bây giờ chúng ta sẽ đưa file `.jar` đã build ở bước trước lên máy chủ Amazon EC2 và thiết lập bộ cân bằng tải (Application Load Balancer).

#### Bước 1: Thiết lập Mạng & NAT Gateway (Bảo mật)

Để đảm bảo bảo mật, máy chủ EC2 backend (chứa logic và kết nối DB) sẽ được đặt trong **Private Subnet** (không có IP Public). Để EC2 có thể tải các gói cài đặt (như Java) từ Internet về, chúng ta cần cấu hình NAT Gateway ở Public Subnet.

![VPC Resource Map](/images/1-Worklog/vpc_resource_map.png)

Xác minh rằng NAT Gateway đã được khởi tạo thành công và liên kết với public subnet:
![NAT Gateway Configuration](/images/1-Worklog/nat_gateway_pending.png)

#### Bước 2: Khởi tạo EC2 Instance

1. Truy cập **AWS Console** → Chọn dịch vụ **EC2** → Click **Launch instance**.
2. **Name:** `petshop-backend-server`
3. **AMI (Hệ điều hành):** Chọn `Amazon Linux 2023` (hoặc Ubuntu).
4. **Instance type:** Chọn `t3.micro` (Đủ điều kiện Free Tier).
5. Network settings:
    * Chọn VPC: `Pet-Shop-vpc`
    * Chọn Subnet: **Private Subnet**
    * Security Group: Chọn `SG_Backend` (Lưu ý: Inbound rules của SG này chỉ mở port `8080` cho phép luồng dữ liệu từ `SG_ALB` đi vào. Tuyệt đối không mở port 22, chúng ta sẽ quản trị server qua AWS Systems Manager).

![Security Groups](/images/1-Worklog/security_groups.png)

Đồng thời, tạo một KMS Customer Managed Key để mã hóa các thông số và bí mật của hệ thống:
![KMS Key Creation](/images/1-Worklog/kms_key.png)

Click **Launch instance** và đợi trạng thái chuyển sang `Running`.

#### Bước 3: Cài đặt môi trường Java 17 trên EC2

Kết nối vào EC2 thông qua **AWS Systems Manager (Session Manager)** hoặc SSH. Sau đó chạy lệnh để cài đặt Java 17:

```bash
# Cập nhật hệ thống
sudo dnf update -y

# Cài đặt Amazon Corretto 17 (Java 17)
sudo dnf install java-17-amazon-corretto -y

# Kiểm tra phiên bản
java -version
```

#### Bước 4: Upload file JAR lên Server

Cách đơn giản nhất trên AWS là upload file `pet-resort-api-0.0.1-SNAPSHOT.jar` lên **Amazon S3** trước, sau đó từ EC2 kéo file đó về.

```bash
# Tải file từ S3 bucket xuống máy chủ EC2
aws s3 cp s3://your-bucket-name/pet-resort-api-0.0.1-SNAPSHOT.jar ./pet-resort-api.jar
```

#### Bước 5: Chạy ứng dụng Spring Boot

Khởi chạy ứng dụng chạy ngầm trên máy chủ bằng lệnh `nohup`:

```bash
nohup java -jar pet-resort-api.jar > app.log 2>&1 &
```
Lúc này, API của bạn đã chạy trên cổng `8080` của địa chỉ Private IP (Ví dụ: `10.0.1.15:8080`).
 
Bạn có thể kiểm tra xem ứng dụng đã khởi chạy thành công hay chưa bằng cách kiểm tra log (ví dụ: chạy lệnh `cat app.log` hoặc chạy trực tiếp lệnh Java để quan sát log xuất ra màn hình console):

![Ứng dụng Spring Boot khởi chạy thành công trên cổng 8080](/images/5-Workshop/backend-run.png)

#### Bước 6: Tạo Target Group (Nhóm đối tượng đích)

Trước khi cấu hình bộ cân bằng tải Application Load Balancer, chúng ta cần định nghĩa nhóm đối tượng đích nhận traffic backend.

1. Truy cập **EC2 Console** → Chọn **Target Groups** ở menu trái → Click **Create target group**.
2. **Target type**: Chọn **Instances**.
3. **Target group name**: `ALB-target-group`.
4. **Protocol**: `HTTP` | **Port**: `80` (Nếu cấu hình port 80, đảm bảo rằng bạn override port nhận traffic của EC2 backend thành `8080` nơi Spring Boot đang chạy khi đăng ký target).
5. **VPC**: Chọn `Pet-Shop-vpc`.
6. Click **Next** → Chọn EC2 `petshop-backend-server` và đăng ký nó → Click **Create target group**.

![Cấu hình Target Group](/images/5-Workshop/target-group.png)

#### Bước 7: Cấu hình Application Load Balancer (ALB)

Vì EC2 Backend nằm ở Private Subnet không ai truy cập được, chúng ta dùng ALB đặt ở Public Subnet để "đứng mũi chịu sào" nhận request công khai từ Frontend rồi đẩy an toàn vào EC2 ở subnet nội bộ.

1. Vào EC2 Console → Kéo xuống mục **Load Balancers** → **Create Load Balancer**.
2. Chọn **Application Load Balancer**.
3. Scheme: **Internet-facing**
4. Mapping: Chọn VPC `Pet-Shop-vpc` và ít nhất 2 Public Subnets.
5. **Listeners and routing**:
   - HTTP: 80 hoặc HTTPS: 443.
   - **Default action**: Chọn **Forward to** và trỏ tới Target Group `ALB-target-group` vừa tạo ở bước trên.
6. Click **Create load balancer**.

![Cấu hình Load Balancer](/images/5-Workshop/alb-dns.png)

#### Bước 8: Cấu hình CORS cho Frontend

Để Frontend (ReactJS) có thể gọi được API mà không bị chặn, cần cấu hình CORS trong mã nguồn Spring Boot (thường nằm ở class `WebConfig` hoặc `SecurityConfig`):

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins(
                    "http://localhost:3000", 
                    "https://d3uvhesft661gl.cloudfront.net" // Domain thật của CloudFront Frontend
                )
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true);
    }
}
```
*(Nếu cập nhật, bạn cần build lại file `.jar` và deploy lại).*

#### Bước 9: Thiết lập Launch Template & Auto Scaling Group cho Backend (Tối ưu hóa khả dụng)

Để hệ thống hoạt động ổn định ở quy mô lớn, tự động phục hồi khi máy chủ gặp sự cố và phân phối tải đều giữa các Private Subnets, chúng ta tạo bản sao hình ảnh máy ảo (AMI) từ EC2 Backend hiện tại, sau đó tạo Launch Template và thiết lập Auto Scaling Group.

1. **Tạo Amazon Machine Image (AMI) từ EC2 Backend:**
   - Vào **EC2 Console** → Chọn **Instances** → Tích chọn instance `petshop-backend-server`.
   - Chọn **Actions** → **Image and templates** → **Create image**.
   - **Image name**: `pet-shop-backend-AMI`.
   - Click **Create image**. Trạng thái của AMI sẽ chuyển sang `Available` sau khi quá trình tạo hoàn tất.
   
   ![Tạo Amazon Machine Image](/images/5-Workshop/backend-ami.png)

2. **Tạo Launch Template (Bản mẫu cấu hình khởi chạy):**
   - Vào **EC2 Console** → Chọn **Launch Templates** ở menu trái → Click **Create launch template**.
   - **Launch template name**: `petshop-launch-template`.
   - **Application and OS Images (Amazon Machine Image)**: Ở mục **My AMIs**, chọn `pet-shop-backend-AMI` vừa tạo ở trên.
   - Chọn Instance type (`t3.micro`), Key pair, và Security Group `SG_Backend` giống như đã thiết lập thủ công ở Bước 2.
   - Nhấp **Create launch template**.
   
   ![Cấu hình Launch Template](/images/5-Workshop/launch-template.png)

3. **Cấu hình Auto Scaling Group (ASG):**
   - Vào **EC2 Console** → Chọn **Auto Scaling Groups** ở menu trái → Click **Create Auto Scaling group**.
   - **Auto Scaling group name**: `petshop-backend-autoscaling`.
   - **Launch template**: Chọn `petshop-launch-template` vừa tạo.
   - **Network**: Chọn VPC `Pet-Shop-vpc` và chỉ chọn các **Private Subnets** (Private Subnet 1 & Private Subnet 2) để đảm bảo máy chủ không bị lộ ra ngoài Internet.
   - **Load balancing**: Tích hợp với Load Balancer hiện tại, chọn Target Group `ALB-target-group` để tự động đăng ký các instance mới tạo.
   - **Group size (Kích thước nhóm)**: Cấu hình tải cơ bản với **Desired capacity: 2**, **Minimum capacity: 2**, và **Maximum capacity: 4** nhằm đảm bảo luôn duy trì tối thiểu 2 máy chủ chạy song song.
   - Hoàn tất các bước và chọn **Create Auto Scaling group**.

   ![Cấu hình Auto Scaling Group](/images/5-Workshop/auto-scaling.png)

#### Các Vấn đề Thường gặp

**Vấn đề: API bị Timeout (504 Gateway Time-out)**
- EC2 chưa khởi động xong hoặc ứng dụng bị lỗi.
- Kiểm tra Security Group của EC2 đã mở port 8080 cho ALB chưa.
- Kiểm tra ALB Target Group xem trạng thái Health Check có đang là `Healthy` không.

**Vấn đề: Lỗi kết nối Database khi chạy app**
- Kiểm tra lại cấu hình URL, username, password của Database trong file `application.properties`.
- Đảm bảo Security Group của RDS cho phép IP của EC2 truy cập vào cổng `3306`.

**Vấn đề: CORS Errors trên Console của trình duyệt**
- Cập nhật lại danh sách `allowedOrigins` trong code Spring Boot.
- Đảm bảo sử dụng giao thức chính xác (`http` vs `https`).

#### Bước tiếp theo

Máy chủ Backend đã sẵn sàng! Bây giờ chúng ta sẽ kết nối máy chủ này với Cơ sở dữ liệu và Cache trong bài **[Kết nối Database & ElastiCache](../5.2.4-testing/)**!