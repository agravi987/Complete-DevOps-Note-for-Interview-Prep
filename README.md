# 🚀 DevOps 80/20 Interview Notes

![DevOps 80/20 Roadmap](assets/devops-80-20-roadmap.svg)

![DevOps](https://img.shields.io/badge/DevOps-Interview%20Prep-2563eb?style=for-the-badge)
![Linux](https://img.shields.io/badge/Linux-Servers-f59e0b?style=for-the-badge&logo=linux&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containers-0ea5e9?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-326ce5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-IaC-7c3aed?style=for-the-badge&logo=terraform&logoColor=white)

📘 Minimal, practical DevOps notes for interviews, hands-on practice, and real-world workflow revision.

> **80/20 rule:** learn the 20% of concepts that explain 80% of daily DevOps work.

Each topic now starts with a **🖼️ Quick Visual Summary** and a **⚡ 80/20 Summary** for fast revision.

## 🎯 What To Master First

| Area | 80/20 Concepts | Must Practice |
| --- | --- | --- |
| 🐧 Linux | Filesystem, permissions, processes, logs, networking commands | `ls`, `chmod`, `ps`, `systemctl`, `journalctl`, `grep`, `find` |
| 🌐 Networking | DNS, HTTP/HTTPS, ports, firewalls, routing, load balancing | `ping`, `curl`, `dig`, `nslookup`, `ss`, `traceroute` |
| 🐳 Docker | Images, containers, volumes, networks, Compose | Build, run, debug, push, clean up |
| ☸️ Kubernetes | Pods, Deployments, Services, ConfigMaps, Secrets, rolling updates | `kubectl get/describe/logs/exec/apply` |
| 🔁 CI/CD | Git flow, pipelines, artifacts, runners, approvals | Build-test-deploy pipeline |
| ☁️ Cloud | VPC, IAM, EC2, S3, security groups, high availability | Deploy a small app securely |
| 🏗️ Terraform | Providers, state, variables, modules, lifecycle | Create and update infra safely |
| 📊 Monitoring | Metrics, logs, alerts, dashboards, SLO basics | Prometheus + Grafana basics |

## 🧭 Learning Path

```text
01 Linux
   ↓
02 Networking for DevOps
   ↓
03 Docker
   ↓
04 Kubernetes
   ↓
05 CI/CD
   ↓
06 Cloud
   ↓
07 Terraform
   ↓
08 Monitoring
```

## 📂 Notes Index

| Module | Notes |
| --- | --- |
| 🐧 [01-Linux](01-Linux) | Linux architecture, permissions, processes, networking commands |
| 🌐 [02-Networking](02-Networking) | DevOps networking fundamentals, DNS/HTTP, ports, firewalls, troubleshooting |
| 🐳 [03-Docker](03-Docker) | Docker images, containers, volumes, networking, Compose |
| ☸️ [04-Kubernetes](04-Kubernetes) | Architecture, Pods, ReplicaSets, Deployments, Services, updates |
| 🔁 [05-CI-CD](05-CI-CD) | Git, Jenkins, GitHub Actions |
| ☁️ [06-Cloud](06-Cloud) | AWS infrastructure, VPC, EC2, S3, IAM |
| 🏗️ [07-Terraform](07-Terraform) | State, HCL, variables, meta-arguments |
| 📊 [08-Monitoring](08-Monitoring) | Prometheus architecture, Grafana dashboards |

## ⚡ Interview Revision Checklist

- ✅ Can you explain the problem each tool solves?
- ✅ Can you draw the basic architecture from memory?
- ✅ Can you debug the most common failure?
- ✅ Can you write the core command without searching?
- ✅ Can you connect it to a real deployment workflow?

## 🧪 Practice Flow

1. Read the **Overview** and **Why It Matters**.
2. Memorize the **Core Concepts**.
3. Run the **Commands**.
4. Break something on purpose and fix it.
5. Review the **Interview Q&A** before interviews.

## 💡 Best Use

Use this repo as a fast revision map, not a textbook. Learn the concept, run the command, understand the failure, then move to the next layer.
