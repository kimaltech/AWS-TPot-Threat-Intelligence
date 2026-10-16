# Cloud Threat Intelligence & Honeypot Cluster (AWS + T-Pot)

A cloud-native multi-service honeypot deployed on AWS EC2 to capture, parse, and analyze automated reconnaissance, credential spraying, and exploit attempts from public internet traffic.

---

## Architecture Overview

- **Cloud Platform:** Amazon Web Services (AWS)
- **Compute Instance:** EC2 `m7i-flex.large` (2 vCPUs, 8 GB RAM)
- **Operating System:** Ubuntu 24.04 LTS
- **Storage:** 100 GB `gp3` EBS volume (resized via `growpart` and `resize2fs`)
- **Container Framework:** T-Pot (Docker & Docker Compose running 20+ microservices)
- **SIEM / Telemetry Pipeline:** ELK Stack (Elasticsearch, Logstash, Kibana) + Suricata NIDS

---

## Network Architecture & Operational Security (OpSec)

To balance adversary exposure with administrative isolation, network ingress was segmented into two distinct security profiles within the AWS Security Group:

1. **Trap / Decoy Ingress (`0.0.0.0/0`):** Exposing common target services globally to invite automated scanning noise:
   - Port 21 (FTP)
   - Port 22 (SSH - routed to Cowrie)
   - Port 80 / 443 (HTTP/HTTPS)
   - Port 445 (SMB - routed to Dionaea)
   - Port 3389 (RDP)
   - Port 3306 (MySQL)
2. **Management Plane Isolation:** 
   - Management SSH remapped from port 22 to TCP `64295`.
   - Web management dashboard remapped to TCP `64297`.
   - Both management ports strictly restricted to an authorized management IP.

![AWS Security Group Configuration](images/aws-security-group.png)

---

## Deployment & System Engineering

1. **Host Provisioning:** Deployed an EC2 instance sized to handle Elasticsearch JVM memory requirements.
2. **Storage Management:** Expanded the root volume to 100 GB `gp3` to handle high-frequency index writes without disk exhaustion:
   ```bash
   sudo growpart /dev/nvme0n1 1
   sudo resize2fs /dev/nvme0n1p1
