# aws-cloudwatch-observability-pipeline
"An end-to-end cloud monitoring solution that utilizes the Amazon CloudWatch Agent to securely ingest Apache web server logs from an EC2 instance, transforming raw text into real-time operational metrics and visual dashboards."
# AWS Cloud Observability Pipeline

An end-to-end cloud monitoring solution that utilizes the Amazon CloudWatch Agent to securely ingest Apache web server logs from an EC2 instance, transforming raw text into real-time operational metrics and visual dashboards.

## 📐 Architecture Diagram

This pipeline bridges the gap between infrastructure deployment and active systems observability, establishing a secure data flow from raw text files to high-level visualizations.
### 🔄 The Data Lifecycle
1. **Traffic Generation:** Users hit the public EC2 IP address, prompting the Apache Web Server (`httpd`) to log local infrastructure access data to `/var/log/httpd/access_log`.
2. **Secure Log Ingestion:** The Amazon CloudWatch Agent, executing under an isolated `cwagent` system account with explicit directory execute permissions, securely streams log payloads to the cloud.
3. **Identity Verification:** The payload is securely signed and authorized using a granular, least-privilege IAM Role attached to the EC2 instance.
4. **Metric Transformation:** Raw log lines enter CloudWatch Log Groups, where a custom Metric Filter scans and converts incoming log text into time-series data (`InboundTraffic`).
5. **Operational Visualization:** CloudWatch Dashboards dynamically plots the numerical count metrics onto a continuous line graph widget for instant health monitoring.

---

## 🛠️ AWS Services & Technologies Used

* **AWS EC2 (Elastic Compute Cloud):** Hosted the web application instance running Red Hat Enterprise Linux (RHEL).
* **Apache HTTP Server (`httpd`):** Served the frontend application and generated local system access records.
* **Amazon CloudWatch Agent:** Installed locally on the server to bridge the data gap and ship logs securely out of the host OS.
* **AWS IAM (Identity and Access Management):** Enforced least-privilege access using a dedicated role to authorize metrics publishing.
* **AWS CloudWatch Logs / Metrics / Dashboards:** Acted as the core telemetry suite for centralized log storage, data filtering, and continuous visual monitoring.

---

## 🔬 Project Progression & Visual Proof

### 1. The Starting Point: Provisioning & Blind Operations
The project began by launching an isolated EC2 instance and building out the Apache environment. While the server successfully handled ingress web traffic, the engineering team suffered from total system blindness—possessing no centralized logs or operational telemetry.

| SSH Access Verified | Apache Web Server Live | Initial CloudWatch Blindness |
|---|---|---|
| ![SSH Success](screenshots/01-ssh-terminal-success.png) | ![Apache Live](screenshots/02-apache-landing-page.png) | ![Empty Logs](screenshots/03-cloudwatch-empty-logs.png) |

### 2. The Solution: Engineering the Telemetry Pipeline
To solve the visibility gap, a least-privilege IAM profile was coupled to the instance to authorize monitoring delivery. The CloudWatch Agent was then successfully configured and deployed to watch the restricted access files.

| Secure IAM Role Attached | Centralized Log Group Established |
|---|---|
| ![IAM Role Profile](screenshots/04-iam-role-attached.png) | ![Log Group Active](screenshots/05-cloudwatch-log-group.png) |

### 3. Operational Visibility: Live Monitoring & Analytics
By implementing a custom metric filter to intercept raw log lines, traffic data was successfully extracted, aggregated, and mapped to an executive-facing dashboard showing real-time, active traffic patterns.

| Operational Control Dashboard | Real-Time Time-Series Analytics |
|---|---|
| ![CloudWatch Dashboard](screenshots/06-cloudwatch-dashboard.png) | ![Metrics Spike Visualization](screenshots/07-cloudwatch-metrics.png) |

---

## ⚡ Key Engineering Challenges Overcome

### 🔒 Linux Directory Permission Masking
* **The Problem:** The CloudWatch agent reported healthy execution statuses but quietly failed to deliver logs to AWS, resulting in completely empty log streams.
* **The Resolution:** Diagnosed a system user mismatch. The CloudWatch agent runs natively under a restricted system account (`cwagent`), while Apache restricts its parent logging directory to `root`. Executed granular permission modifications (`sudo chmod +x /var/log/httpd`) to open the directory path exclusively for traversal, allowing the agent to read log lines without compromising host system security.

### 🌐 Dynamically Reassigned Cloud IPs
* **The Problem:** Re-initiating the compute layer broke test tracking links due to AWS dynamically reallocating Public IPs on standard instance launches. 
* **The Resolution:** Audited the infrastructure layer to ensure network configuration shifts were accounted for, matching the internal agent stream configurations to target the newly updated endpoints seamlessly.
