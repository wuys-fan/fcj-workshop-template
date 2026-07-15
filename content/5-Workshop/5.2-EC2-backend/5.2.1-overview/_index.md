---
title: "Workshop Overview"
weight: 1
chapter: false
pre: " <b> 5.2.1 </b> "
---

# Backend Workshop Overview

#### What you will build

In this section, you will deploy a production-ready **Java Spring Boot RESTful API** for the **Pet Resort & Care System**. The application will be deployed directly onto **Amazon EC2** instances, connected to a relational database, and traffic will be managed through an **Application Load Balancer (ALB)**.

The API includes **Swagger UI** for easy documentation and testing:

- 🏠 **Product & Service Management** - Retrieve pet products and Spa services, optimized with Caching.
- 📅 **Booking System** - Secure and accurate appointment scheduling logic.
- 👤 **User Management** - Authentication and role authorization (Customer, Staff, Admin).
- 🖼️ **Media Handling** - API support for uploading and fetching image URLs from Amazon S3.
- 📝 **Swagger UI** - Interactive interface for API documentation and testing.

#### Architecture Overview

```text
┌─────────────────────────────────────────────────────┐
│           ReactJS Frontend (CloudFront / S3)        │
│           (https://d3uvhesft661gl.cloudfront.net/) │
└──────────────────┬──────────────────────────────────┘
                   │ HTTPS API Calls
                   ▼
┌─────────────────────────────────────────────────────┐
│       Application Load Balancer (ALB)               │
│         - Health Check mechanism                    │
│         - Traffic Routing                           │
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
│- Caching Data│    │- Store data  │
└──────────────┘    └──────────────┘
```

#### Core Technologies

| Technology | Purpose | Why? |
|-----------|----------|----------|
| **Java Spring Boot** | Backend Framework | Robust, secure, and enterprise-standard |
| **Swagger UI** | API Documentation | Auto-generated docs, interactive testing |
| **Amazon EC2 & ALB** | Compute & Routing | Full server control, automatic load balancing |
| **Amazon RDS (MySQL)**| Relational Database | Stable, automated backups & Multi-AZ support |
| **Amazon ElastiCache**| In-memory Cache | Reduces database load, accelerates data retrieval |

#### Cost Estimation

The system is flexibly designed to run in a test environment (Free Tier) or scale up for an enterprise-level Production environment.

**Practice Environment (Free Tier):**
- EC2 `t3.micro`: 750 hours/month (Free for 1 instance)
- RDS `db.t3.micro`: 750 hours/month (Free)
- ElastiCache `cache.t2.micro`: 750 hours/month (Free)
- Only cost incurred is the Application Load Balancer: ~$16/month

**Production Environment (Based on High-Performance setup):**
- EC2 Instances with Auto Scaling: Pay-as-you-go based on actual traffic.
- RDS `db.m7g.large` & ElastiCache `cache.r7g.large`: Capable of handling tens of thousands of Requests/second (~$400 - $600/month).