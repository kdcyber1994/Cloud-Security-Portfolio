# Enterprise Cloud Security & Architecture Portfolio

Welcome to my enterprise cloud security repository. This space serves as a live engineering runbook and technical portfolio documenting my implementation, hardening, and governance of hybrid and cloud infrastructure specifically focusing on the Microsoft Security Stack (AZ-500/SC-500 tracks).

## 🛠️ Portfolio Focus Areas
* **Identity & Access Management (IAM):** Hybrid synchronization, Entra ID governance, and Zero Trust access controls.
* **Infrastructure & Perimeter Security:** Network security groups, cloud workload protection, and boundary hardening.
* **Data Protection & Compliance:** Information protection, logging/monitoring configuration, and control mapping (NIST 800-53/NIST CSF).

## 📈 Active Certification Tracks
* **Microsoft Certified: Azure Security Engineer Associate (AZ-500)**
* **Microsoft Certified: Cloud and AI Security Engineer Associate (SC-500 Track)**

## 🧪 Hands-On Engineering Exercises

### Identity & Access Management (IAM)
* **[Lab 01: Cloud-Native User Provisioning](./01-identity-access-management/lab-01-user-provisioning.md)** — Setting up a fresh cloud identity in a test tenant and configuring basic user profile parameters.
* **[Lab 02: Policy-Driven MFA via Conditional Access](./01-identity-access-management/lab-02-conditional-access-mfa.md)** — Disabling generic security defaults to build conditional access policies. Enforces MFA for specific groups while keeping admin accounts safe from lockout.
* **[Lab 03: Phishing-Resistant MFA via FIDO2 Passkeys](./01-identity-access-management/lab-03-phishing-resistant-MFA-FIDO2-Passkeys.md)** — Setting up a phishing-resistant FIDO2 passkey on an Android device and creating a custom authentication strength to enforce it.
* **[Lab 04: Just-in-Time Access Controls via Microsoft Entra PIM](./01-identity-access-management/lab-04-Configuring-Just-in-Time-Access-Conrols-via-Microsoft-Entra-PIM.md)** — Configured Just-in-Time (JIT) access and hardened the setup by forcing activation requests to go through a manual administrator approval workflow. 

### Infrastructure & Data Security
* **[Lab 05: Restricting Key Vault Public Perimeters and Implementing Multi-Layer Access Validation](./lab-05-key-vault-perimeter-hardening.md)** — Closing internet-facing exposure on an Azure Key Vault by restricting access to a specific trusted public IP and implementing modern Azure RBAC to separate vault management from secret visibility.

---
*Maintained by Kevin Dunn — Security Engineer*
