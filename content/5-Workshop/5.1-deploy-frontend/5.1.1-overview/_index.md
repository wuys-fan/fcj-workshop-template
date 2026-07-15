---
title: "Workshop Overview"
weight: 1
chapter: false
pre: " <b> 5.1.1 </b> "
---

# Workshop Overview

#### Workshop Objectives

Upon completing this workshop, you will manually build a system on AWS with the following skills:
- ✅ Build a highly scalable web system that withstands high traffic without crashing.
- ✅ Segment the network (VPC, Subnets) to hide source code and data, preventing hacker access.
- ✅ Utilize ElastiCache (Redis) to make the website load smoother and faster.
- ✅ Store images and static UI at minimal cost on Amazon S3.
- ✅ Configure automated Email alerts when the server encounters issues.

#### Pet Resort & Care System - Core Features

In this workshop, we will deploy the Pet Resort & Care System platform, encompassing these modules:
- 🏠 **Home & Products**: Display catalogs of pet products and services.
- 📅 **Booking System**: Secure and accurate Spa appointment scheduling.
- 🖼️ **Media Management**: A secure repository for staff to upload and store pet images.

#### High-Level Deployment Architecture

Below is the system flow, detailing the journey from a customer clicking on the website to the data being securely stored:

```text
┌──────────────────────────────┐
│             User             │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│      CloudFront & WAF        │ (Firewall & Load Acceleration)
└──────┬───────────────┬───────┘
       │               │
       ▼               ▼
┌────────────┐   ┌─────────────┐
│ S3 Frontend│   │ Load Balancer (Traffic Router)
│ (Web UI)   │   └──────┬──────┘
└────────────┘          │
                        ▼
                 ┌─────────────┐
                 │ EC2 Backend │ (Code Execution & Auto Scaling)
                 └─┬────┬────┬─┘
                   │    │    │
          ┌────────┘    │    └────────┐
          ▼             ▼             ▼
  ┌────────────┐ ┌─────────────┐ ┌───────────────┐
  │ ElastiCache│ │  Amazon RDS │ │   S3 Media    │
  │ (Cache)    │ │  (Database) │ │(Internal Media)
  └────────────┘ └─────────────┘ └───────────────┘
```

#### Security & Flow

- **Reception & Protection:** Customer requests pass through the WAF filter to block malicious access, followed by CloudFront to accelerate website loading.
- **Routing:** The website UI is fetched from the S3 bucket. Operations like "Booking" or "Purchasing" are routed through the Load Balancer to distribute the workload evenly among internal servers.
- **Processing:** The EC2 server receiving the request will prioritize finding data in the Cache (Redis) for speed. If unavailable, it queries the Database (RDS MySQL) to read or save new data.
- **Secure Image Storage:** Uploading pet images routes through an internal network "tunnel" (VPC Endpoint) directly into the S3 Media bucket, bypassing the public Internet to ensure security.
- **Monitoring:** CloudWatch continuously monitors the system. If any server fails, it automatically sends an Email alert to the administration team.

#### Estimated Costs

By cleverly leveraging the AWS Free Tier, the maintenance cost of this system is reduced to a minimum:

- ✅ **EC2 Servers & Caching (Redis):** Free 750 hours/month (Free Tier).
- ✅ **RDS Database:** Free 750 hours/month (Free Tier).
- ✅ **S3 Storage & CloudFront:** Within free quotas.

**Actual Estimated Cost:** Approximately **$50 - $70/month** for the practice environment. This cost is primarily to maintain 2 networking services: NAT Gateway and Load Balancer. The full Multi-AZ architecture (as described in the Proposal) costs about **~$168/month**.

{{% notice tip %}}
💡 **Tip:** During practice, if you don't need a production-ready setup, you can temporarily turn off the NAT Gateway and run the Database in Single-AZ mode to maximize savings on your card.
{{% /notice %}}

#### Next Steps

Start with [Network Setup (VPC & Subnets)](#) to build the outermost defense layer for your system.