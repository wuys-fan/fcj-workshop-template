---
title: "Deploy to Amazon EC2"
weight: 3
chapter: false
pre: " <b> 5.2.3 </b> "
---

# Deploy Spring Boot to Amazon EC2 & ALB

Now, we will deploy the `.jar` file built in the previous step onto an Amazon EC2 server and configure an Application Load Balancer to route external traffic.

#### Step 1: Networking & NAT Gateway Setup (Security)

For security purposes, the backend EC2 server (hosting business logic and DB connections) is placed within a **Private Subnet** (without a Public IP). To allow the EC2 instance to download necessary packages (like Java) from the Internet, we must configure a NAT Gateway in the Public Subnet.

![VPC Resource Map](/images/1-Worklog/vpc_resource_map.png)

Verify that your NAT Gateway is successfully created and associated with the public subnet:
![NAT Gateway Configuration](/images/1-Worklog/nat_gateway_pending.png)

#### Step 2: Provision the EC2 Instance

1. Navigate to the **AWS Console** → Select **EC2** → Click **Launch instance**.
2. **Name:** `petshop-backend-server`
3. **AMI (OS):** Choose `Amazon Linux 2023` (or Ubuntu).
4. **Instance type:** Select `t3.micro` (Free Tier eligible).
5. **Network settings:** 
   - Select VPC `Pet-Shop-vpc`
   - Select a Private Subnet
   - Security Group: Allow Inbound port `8080` (Spring Boot default port) and port `22` (SSH).

![Security Groups](/images/1-Worklog/security_groups.png)

Additionally, set up a KMS Customer Managed Key to encrypt database/backend parameters and secrets:
![KMS Key Creation](/images/1-Worklog/kms_key.png)

Click **Launch instance** and wait until the status changes to `Running`.

#### Step 3: Install Java 17 on EC2

Connect to your EC2 instance via **AWS Systems Manager (Session Manager)** or SSH. Then, run the following commands to install Java 17:

```bash
# Update system packages
sudo dnf update -y

# Install Amazon Corretto 17 (Java 17)
sudo dnf install java-17-amazon-corretto -y

# Verify installation
java -version
```

#### Step 4: Upload the JAR file to the Server

The simplest method on AWS is to upload the `pet-resort-api-0.0.1-SNAPSHOT.jar` file to an **Amazon S3** bucket first, then pull it down from the EC2 instance.

```bash
# Download the file from S3 bucket to the EC2 instance
aws s3 cp s3://your-bucket-name/pet-resort-api-0.0.1-SNAPSHOT.jar ./pet-resort-api.jar
```

#### Step 5: Run the Spring Boot Application

Launch the application in the background using the `nohup` command:

```bash
nohup java -jar pet-resort-api.jar > app.log 2>&1 &
```
At this point, your API is running on port `8080` attached to its Private IP (e.g., `10.0.1.15:8080`).

#### Step 6: Configure Application Load Balancer (ALB)

Since the EC2 instance resides in a Private Subnet, it cannot be accessed directly. We use an ALB in the Public Subnet to receive requests from the Frontend and forward them to the EC2 instance.

1. In the EC2 Console → Scroll down to **Load Balancers** → **Create Load Balancer**.
2. Select **Application Load Balancer**.
3. Scheme: **Internet-facing**
4. Mapping: Select the `Pet-Shop-vpc` and at least 2 Public Subnets.
5. **Listeners and routing:** 
   - HTTP: 80 or HTTPS: 443
   - Forward traffic to a Target Group containing the `petshop-backend-server` EC2 instance on port `8080`.

![Target Group Configuration](/images/5-Workshop/target-group.png)

#### Step 7: Configure CORS for the Frontend

To ensure the Frontend (ReactJS) can successfully call the API without browser blockages, you must configure CORS in your Spring Boot source code (typically in the `WebConfig` or `SecurityConfig` class):

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins(
                    "http://localhost:3000", 
                    "https://d3uvhesft661gl.cloudfront.net" // Real CloudFront domain
                )
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true);
    }
}
```
*(If updated, you must rebuild the `.jar` file and redeploy it).*

#### Common Issues

**Issue: API Timeout (504 Gateway Time-out)**
- The EC2 instance hasn't fully started, or the application crashed.
- Verify that the EC2 Security Group allows traffic on port 8080 from the ALB.
- Check the ALB Target Group to ensure the Health Check status is `Healthy`.

**Issue: Database connection error upon startup**
- Double-check the Database URL, username, and password in your `application.properties`.
- Ensure the RDS Security Group allows Inbound traffic from the EC2 instance IP on port `3306`.

**Issue: CORS Errors in the Browser Console**
- Update the `allowedOrigins` array in your Spring Boot code.
- Ensure you are using the correct protocol (`http` vs `https`).

#### Next Steps

The backend server is ready! Now, we will integrate it with our Database and Cache in the **[Database & ElastiCache Connection](../5.2.4-testing/)** section!