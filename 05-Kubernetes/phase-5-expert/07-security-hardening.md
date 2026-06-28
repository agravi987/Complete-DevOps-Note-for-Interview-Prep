# 🛡️ Security Hardening

> **Hey Ravi! 👋** Did you know that the Tesla Kubernetes cluster was famously hacked in 2018 to mine cryptocurrency? They left the Kubernetes Dashboard open to the internet with no password! 🤦‍♂️ Security isn't just about Network Policies. It's about hardening every single layer of the stack. Let's make sure you never end up on the news! 🕵️‍♂️

---

## 🧠 The 4 C's of Cloud Native Security

Security must be applied at every layer (Russian Doll style):
1. **Cloud:** IAM, VPC boundaries, Network Security Groups.
2. **Cluster:** RBAC, API Server flags, etcd encryption.
3. **Container:** Image scanning, removing shell access.
4. **Code:** Static code analysis, dependency checking.

---

## 🏛️ Hardening the API Server (The Front Door)

The API server is the most dangerous target.

1. **Disable Anonymous Auth:** By default, K8s sometimes allows unauthenticated users to ping the API.
   *Flag:* `--anonymous-auth=false`
2. **Enable Audit Logging:** Keep a permanent record of who deleted that production pod!
   *Flag:* `--audit-log-path=/var/log/apiserver/audit.log`
3. **Node Restriction:** A compromised Kubelet (Worker Node) should only be able to read/write its *own* pods, not pods on other nodes!
   *Flag:* `--enable-admission-plugins=NodeRestriction`

---

## 📋 CIS Kubernetes Benchmark

You don't need to guess what flags to set. The **Center for Internet Security (CIS)** publishes a 100-page PDF checklist of exactly how every single K8s component should be configured securely.

**Kube-bench:** A popular open-source tool by Aqua Security that automatically scans your cluster and tells you exactly which CIS Benchmark rules you are failing! 💯

---

## 🕵️‍♂️ Runtime Security (Falco)

You secured the API, you locked the containers (Pod Security Standards), but what if a zero-day exploit still gets through? You need an alarm system! 🚨

**Falco** is the industry standard for Kubernetes threat detection. It sits in the Linux kernel (using eBPF) and watches for anomalous behavior in real-time!

*Falco will trigger an alert to Slack if it sees:*
- A shell (`bash`/`sh`) being executed inside a container!
- A container trying to read `/etc/shadow` (password files).
- A container making a weird outbound network connection to a crypto-mining pool.

---

## 📦 Supply Chain Security (Images)

Don't deploy garbage into your cluster.

1. **Image Scanning (Trivy/Clair):** Scans your Docker images in the CI/CD pipeline for known CVEs (Common Vulnerabilities and Exposures) *before* they are deployed.
2. **Admission Controller Block:** Write a Validating Webhook (OPA Gatekeeper) that says: "DENY any image that hasn't been signed and verified by our security team."
3. **SBOM (Software Bill of Materials):** A complete ingredient list of every single library inside your Docker image. Required by modern compliance standards!
4. **Cosign (Sigstore):** A tool to cryptographically sign your Docker images so K8s can verify they haven't been tampered with.

---

## 🎤 Interview Knockout Answers

**Q: How would you secure a Kubernetes cluster from a high-level perspective?**
> "I approach security using the 4 C's: Cloud, Cluster, Container, and Code. For the Cluster, I ensure strict RBAC (Principle of Least Privilege), enforce Network Policies (Default Deny), encrypt etcd at rest, and audit the API Server using the CIS Kubernetes Benchmark. For the Container, I enforce Pod Security Standards (running as non-root, read-only filesystems) and use runtime threat detection like Falco."

**Q: What is the CIS Kubernetes Benchmark and how do you use it?**
> "The CIS Benchmark is a prescriptive set of cybersecurity best practices for configuring a Kubernetes cluster. It covers API server flags, kubelet configs, and etcd settings. In practice, we run automated tools like `kube-bench` to constantly scan our clusters against these guidelines and generate compliance reports."

**Q: Explain how Falco enhances Kubernetes security.**
> "While admission controllers prevent bad configurations from entering the cluster, Falco provides Runtime Security. It hooks into the Linux kernel via eBPF to monitor system calls. If a container exhibits malicious behavior at runtime—like spawning a bash shell or mutating critical system files—Falco immediately detects the anomaly and triggers an alert, allowing us to respond to zero-day exploits."

---

## ⚡ 30-Second Revision

- 🏛️ **API Server:** Disable anonymous auth, enable audit logs, use NodeRestriction!
- 📋 **CIS Benchmark:** The industry standard checklist for secure cluster configuration.
- 🛠️ **kube-bench:** Automates checking the CIS Benchmark.
- 🚨 **Falco:** The alarm system. Detects malicious runtime behavior (like opening a bash shell).
- 📦 **Trivy:** Scans container images for known vulnerabilities.
- ✍️ **Cosign/SBOM:** Supply chain security to prove your images are safe and untampered.

---

> **BOOM! Phase 5 Complete! 🎉** You have reached the absolute peak of the mountain! You are a Kubernetes Expert, Ravi! 🏔️
> **Back to:** [Phase 5 README](README.md)
