---
title: "Đóng gói Ứng dụng"
weight: 2
chapter: false
pre: " <b> 5.2.2 </b> "
---

# Đóng gói Ứng dụng Spring Boot để Triển khai

Trước khi đưa mã nguồn lên máy chủ Amazon EC2, chúng ta cần biên dịch (build) và đóng gói toàn bộ ứng dụng Java Spring Boot thành một file thực thi duy nhất (Executable JAR).

#### Bước 1: Mở Terminal trong VS Code

Mở project backend của bạn bằng **Visual Studio Code**. 
Sử dụng tổ hợp phím `` Ctrl + ` `` (hoặc vào menu **Terminal** → **New Terminal**) để mở giao diện dòng lệnh tích hợp ngay bên trong VS Code.

Đảm bảo terminal đang trỏ đúng vào thư mục gốc của dự án (nơi chứa file `pom.xml`).

#### Bước 2: Build dự án bằng Maven

Tại màn hình Terminal của VS Code, gõ lệnh sau để dọn dẹp các bản build cũ, biên dịch mã nguồn mới và đóng gói. Chúng ta sử dụng cờ `-DskipTests` để bỏ qua các bài test cục bộ, giúp quá trình build diễn ra nhanh chóng hơn:

```bash
mvn clean package -DskipTests
```

Sau khi nhấn Enter, Maven sẽ tải các thư viện cần thiết và tiến hành đóng gói. Khi bạn nhìn thấy dòng chữ **[INFO] BUILD SUCCESS** màu xanh lá, quá trình đã hoàn tất thành công!

![Terminal Build Success trong VS Code](/images/5-Workshop/build-backend-2.png)

#### Deployment Package Sẵn sàng! 📦

Lệnh trên đã tự động tạo ra một thư mục có tên là `target` nằm trong cây thư mục bên trái của VS Code. Trong hệ sinh thái Spring Boot, bạn chỉ cần quan tâm đến file quan trọng nhất:

- ✅ `petshop-backend-1.0.0.jar` - File Fat JAR đã sẵn sàng để upload lên máy chủ EC2.
- ✅ Tích hợp sẵn máy chủ Web (Embedded Tomcat).
- ✅ Tất cả dependencies (thư viện) đã được bao gồm đầy đủ bên trong.
- ✅ Cấu hình từ `application.properties` đã được ánh xạ chuẩn xác.

**Vị trí File:**

![Thư mục target chứa tệp JAR](/images/1-Worklog/maven_build_output.png)

#### Các Vấn đề Thường gặp

**Vấn đề:** Báo lỗi `mvn: command not found`
**Giải pháp:** 
- Máy tính của bạn chưa cài đặt Maven hoặc chưa cấu hình biến môi trường (Environment Variables) cho Maven. Cần cài đặt và thêm đường dẫn thư mục `bin` của Maven vào biến `PATH`.

**Vấn đề:** Ứng dụng báo lỗi thiếu phiên bản Java (ví dụ: cần Java 17)
**Giải pháp:**
- Kiểm tra lại phiên bản Java trong file `pom.xml`: `<java.version>17</java.version>`.
- Đảm bảo môi trường EC2 trên AWS sắp tới cũng được cài đặt đúng Java 17.

**Vấn đề:** Lỗi kết nối Database khi chạy lệnh build
**Giải pháp:**
- Nếu bạn quên thêm cờ `-DskipTests`, Spring Boot sẽ cố kết nối với Database ở localhost để chạy Unit Test. Hãy chắc chắn bạn đã dùng cờ này trong lệnh build.

#### Bước tiếp theo

Bây giờ chúng ta đã có file thực thi `.jar` hoàn chỉnh. Bước tiếp theo là cấu hình mạng, khởi tạo **[Amazon EC2 và chạy Server](../5.2.3-deploy-ec2/)**!