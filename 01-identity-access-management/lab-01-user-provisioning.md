# Lab 01: Cloud-Native User Provisioning & Identity Governance

## 🎯 Lab Objective
To establish a secure baseline for cloud-native user identity management within Microsoft Entra ID. This exercise simulates provisioning an enterprise employee identity, defining initial role-based placement properties, and ensuring structural visibility within a sandbox tenant.

## ⚙️ Architectural Context
In a pure cloud-native or hybrid environment, user creation triggers downstream governance cycles. Proper initial assignment of attributes (Job Title, Department, Usage Location) ensures that **Dynamic Group memberships** and **Conditional Access policies** evaluate correctly to enforce a least-privilege operational model.

---

## 🛠️ Execution Walkthrough

### Step 1: Navigating the Core Directory Portal
1. Authenticated to the **Microsoft Entra admin center** (`entra.microsoft.com`).
2. Expanded the left-hand navigation pane to target **Identity** > **Users** > **All Users**.

### Step 2: Defining Initial Identity Profiles
1. Selected **New user** > **Create new user**.
2. Structured the directory parameters under the **Basics** panel:
   * **User Principal Name (UPN):** Assigned the explicit login suffix aligned with the sandbox tenant domain boundary (`TestUser@kdunn1994outlook.onmicrosoft.com`).
   * **Display Name:** Defined using standardized enterprise formats (`Tester`).
   * **Password Rules:** Initial password established with account activation controls toggled on (`Account enabled: Yes`).

[Microsoft Entra ID Basic User Configuration](./images/user-basics-setup.png)

---

### Step 3: Identity Assignment & Administrative Boundary Verification
1. Advanced through the **Properties** and **Assignments** workflows to inspect directory placement opportunities.
2. Verified that the platform allows structural governance injections at ingestion, including:
   * **Administrative Units (AUs):** For segmenting regional or departmental scope boundaries.
   * **Group Memberships:** To apply role-based resource access mappings.
   * **Directory Roles:** To grant granular administrative permissions.
3. Executed the final structural audit on the **Review + create** summary screen before committing the user instantiation write operation to the cloud graph database.

[Microsoft Entra ID Review and Create Validation Screen](./images/user-review-create.png)

---

### Step 4: Verification of Active Directory State
1. Returned to the main directory **All users** dashboard canvas.
2. Confirmed successful tenant object generation, verifying that the new identity is active, designated as a standard account type (`User type: Member`), and strictly isolated as a cloud-only object (`On-premises sync enabled: No`).


[Microsoft Entra ID Activated Directory State Verified](./images/entra-users-list.png)

---

## 🛡️ Defensive Engineering Takeaways
* **Password Complexity Defenses:** By default, cloud-only accounts are subjected to Microsoft Entra ID Smart Lockout and global banned password lists. In production environments, this step should be coupled with a requirement for user-driven change at initial authentication.
* **Attribute Hygiene Enforcement:** Leaving properties like *Department* or *Usage Location* blank breaks automation pipelines. If a Conditional Access policy requires a country code to trigger an MFA challenge, an unhygienic user profile can create an implicit security bypass vulnerability.

---

## 🔒 Post-Lab Hardening & Incident Lifecycle Actions
* **Credential Deactivation:** Following the completion of this provisioning verification, the `Tester` account was immediately updated to a disabled state (`Account enabled: No`) within the Entra ID portal. This action mitigates risk against automated repository harvesting scripts and enforces strict identity lifecycle management boundaries within public-facing sandbox documentation.
