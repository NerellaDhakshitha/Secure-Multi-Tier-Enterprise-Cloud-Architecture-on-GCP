# Secure Multi-Tier Enterprise Cloud Architecture on GCP

## Project Overview

This project focuses on building a segmented and secure cloud environment using Google Cloud Platform (GCP). The objective was to isolate network resources, keep the backend application server private, enable secure administrative access, and manage access to cloud storage through IAM.

I configured a custom VPC, separate subnets for the bastion host and private application server, Cloud NAT for outbound connectivity, and service-account-based access to Google Cloud Storage. I also deployed a simple webpage on the private VM to validate the web server configuration.

This hands-on project helped me understand core GCP networking, access management, Compute Engine, and cloud storage concepts.

---

## Architecture Diagram

![Secure Multi-Tier Enterprise Cloud Architecture on GCP](Architecture.png)

### Architecture Components

- **Custom VPC (`enterprise-vpc`):** Provides the network environment for the project.
- **Bastion Subnet (`10.0.1.0/24`):** Contains the bastion host used as an administrative entry point to the private application VM.
- **Private Subnet (`10.0.2.0/24`):** Contains the backend application server, configured without an external IP address.
- **Cloud Router and Cloud NAT:** Provide outbound connectivity for eligible private resources, including software package updates.
- **Google Cloud Storage (GCS):** Stores test files and uses IAM-controlled access.

---

## Google Cloud Services Used

| Service | Purpose |
|---|---|
| Virtual Private Cloud (VPC) | Network isolation and resource organization |
| VPC Subnets | Separate network address ranges |
| Compute Engine | Hosts the bastion and application VMs |
| Identity-Aware Proxy (IAP) | Secure administrative access |
| Cloud Router and Cloud NAT | Outbound connectivity for private resources |
| Firewall Rules | Control permitted network traffic |
| Identity and Access Management (IAM) | Manage permissions for cloud resources |
| Cloud Storage | Store and upload files using authorized access |

---

## Implementation Details

### 1. Custom VPC and Network Segmentation

Created a custom VPC named `enterprise-vpc` and configured separate subnet ranges for the bastion host and backend application server.

- Bastion subnet: `10.0.1.0/24`
- Private subnet: `10.0.2.0/24`

This design separates the administrative entry point from the private application workload.

### 2. Bastion Host and Secure Administrative Access

Configured a Compute Engine VM named `bastion-host` to provide an administrative entry point to the private application server.

Used Identity-Aware Proxy (IAP) for secure SSH access and an internal SSH connection from the bastion host to the private VM. This avoids exposing the backend VM directly to the public internet.

### 3. Private Application Server and Webpage Deployment

Configured a Debian-based Compute Engine VM named `private-app-server` in the private subnet.

- **Internal IP:** `10.0.2.2`
- **External IP:** None
- **Web server:** Apache
- **Web port:** TCP 80

Deployed a simple HTML webpage titled **Secure Multi-Tier Enterprise Cloud Architecture** to demonstrate that the web server was running.

Verified the webpage locally using `curl` and checked the process listening on port 80. Since the VM has no external IP, I also used an IAP SSH tunnel through Google Cloud Shell and Cloud Shell Web Preview to view the webpage from a browser.

### 4. Cloud NAT and Outbound Connectivity

Configured Cloud NAT with Cloud Router to support outbound internet connectivity for eligible private resources.

This allows the private application VM to access external resources, such as software package repositories, without requiring an external IP address. Cloud NAT is a managed service associated with Cloud Router; it is not a VM hosted inside the public subnet.

### 5. Cloud Storage and IAM Integration

Configured service-account permissions for access to a Google Cloud Storage bucket.

Tested uploading a file using the Google Cloud CLI, demonstrating how a Compute Engine workload can access cloud storage without requiring a hardcoded service-account key file.

The configured `Storage Object Admin` role provides object-management permissions. For a production workload, permissions should be narrowed to the minimum required operations.

---

## Testing and Execution Proofs

### 1. Verify the Web Server

Run the following command on the application VM:

```bash
curl -I http://localhost
```

This checks whether the local web server responds to HTTP requests.

### 2. Identify the Service Listening on Port 80

```bash
sudo ss -ltnp | grep ':80'
```

This helps identify the process listening on the web server port.

### 3. Upload a Test File to Google Cloud Storage

```bash
gcloud storage cp test.txt gs://enterprise-app-bucket-dhakshith-2026/
```

**Observed result:**

```text
Copying file://test.txt to gs://enterprise-app-bucket-dhakshith-2026/test.txt
Operation completed.
```

### 4. Access the Private Webpage

The application VM does not have an external IP address. To view the webpage, I established an IAP SSH tunnel from Google Cloud Shell and used **Web Preview → Preview on port 8080**.

This provided a way to validate the webpage without exposing the application server directly to the public internet.

---

## Key Learnings

- Configuring a custom VPC and subnet address ranges
- Understanding private Compute Engine instances and external IP addresses
- Using a bastion host and IAP for administrative access
- Configuring Cloud NAT for outbound connectivity
- Managing network traffic through firewall rules
- Working with IAM permissions and service accounts
- Deploying and troubleshooting an Apache web server
- Uploading files to Cloud Storage using the Google Cloud CLI
