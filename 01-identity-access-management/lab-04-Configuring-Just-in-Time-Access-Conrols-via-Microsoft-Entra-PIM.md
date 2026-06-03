# Lab 04: Implementing Just-in-Time Access and Approval Workflows with Entra PIM

## 🎯 Lab Objective
The goal of this lab is to eliminate standing administrative privileges within an Entra ID tenant. By leveraging Microsoft Entra Privileged Identity Management (PIM), this setup transitions high-impact roles into Just-in-Time (JIT) entitlements, shifting the tenant from a default auto-activation state to a secure, administrator-approved activation pipeline.

## ⚙️ Architectural Context
Leaving permanent, 24/7 administrative privileges active on user accounts is one of the highest identity risks an organization can take. If a support engineer or administrator account is compromised, attackers instantly inherit those full permissions to move laterally, exfiltrate data, or deploy backdoors without triggering additional roadblocks.

Privileged Identity Management fixes this by introducing Just-in-Time access. Instead of holding active permissions constantly, accounts are granted an *eligibility* status. When an admin needs to do a task, they must explicitly request to activate the role for a limited window. For critical roles, this activation can be bound to an approval pipeline, meaning a second authorized administrator must review the business justification before any permissions are live. This drastically shrinks the tenant's attack surface and generates an immutable audit trail for compliance tracking.

---

## 🛠️ Execution Walkthrough

### Configure the Baseline Eligible Assignment
The first phase focuses on assigning a high-privilege role as an eligible entitlement rather than a permanent active assignment.
1. Navigated to **Identity Governance > Privileged Identity Management** from the Entra admin center menu.

![PIM Quick Start Dashboard](./images/PIM_Quick_Start.png)

2. Selected **Microsoft Entra roles** under the Manage menu and opened the **Roles** catalog to view the directory offerings.

![Roles Catalog Assessment](./images/Roles_Overview.png)

3. Selected the **User Administrator** role, clicked **Assignments > + Add assignments**, and set the scope type to **Directory**. This ensures permissions apply tenant-wide only when explicitly activated.

![Configuring Target Assignment Scope](./images/Add_Role_Assignments_TestUser.png)

4. Switched to the Setting tab, verified the Assignment type was set to **Eligible**, and marked the duration as **Permanently eligible** for the lab account.

![Setting Assignment Eligibility Profile](./images/Add_Role_Assignments_TestUser1.png)

5. Confirmed the configuration saved successfully by checking the **Eligible assignments** dashboard for the targeted role.

<img src="./images/Resource_Audit_Corrected.png" width="100%" style="image-rendering: -webkit-optimize-contrast; image-rendering: crisp-edges;">

---

### Subtask 1 — Test Default Auto-Activation
Before tightening security policies, a baseline test was run to see how the user-side workflow handles a default activation request.
1. Logged into the Entra portal as `TestUser` and navigated to **My roles** within Privileged Identity Management. The User Administrator role properly appeared under the eligible assignments tab.

![User Role Catalog View](./images/My_Roles_TestUser_Overview.png)

2. Clicked **Activate**, which opened the prompt panel. By default, the policy only required a written reason. Entered a realistic helpdesk ticketing justification ("*Activating role to handle user account provisioning...*") and clicked the main action button.

![Submitting Activation Justification](./images/PIM_Activation_Justification.png)

3. Checked the progress panel as the system verified the submission stages and auto-refreshed the browser session.

![Processing Activation Cycle](./images/PIM_Activation_Underway.png)

4. Checked the **Active assignments** tab to verify that the account successfully held full User Administrator permissions without requiring any secondary approval.

![Auto Activation Verification](./images/Active_Assignments.png)

---

### Subtask 2 — Harden the Role Settings to Enforce Approval
To elevate the security posture, the role configuration was modified to force all activation requests through a strict administrative gatekeeper.
1. Logged back in as the Global Administrator, returned to the PIM role settings for User Administrator, and clicked **Edit**.

![Reviewing Default Role Settings Baseline](./images/User_Admin_Summary.png)

2. Under the Activation tab, checked the box to **Require approval to activate**. Clicked the plus icon and explicitly assigned the Global Admin account (`Kevin Dunn`) as the designated approver.

![Enforcing Administrative Approval Policies](./images/Edit_User_Admin_Role_Settings.png)

3. Moved to the Assignment tab to review active guardrails, ensuring that MFA and written justifications remain mandatory for any role actions.

![Reviewing Additional Assignment Guardrails](./images/Edit_User_Admin_Role_Settings_Continued.png)

4. Saved the updates and reviewed the revised **Role setting details** matrix to confirm that the activation approval workflow was officially live.

![Confirming Hardened Policy Baseline](./images/Role_Settings_Summary_After.png)

---

### Subtask 3 — Validate the Administrative Approval Pipeline
With the tightened security controls live, a new activation request was triggered to verify the end-to-end approval workflow.
1. Attempted to activate the role again as `TestUser`. The system blocked immediate entry and held the request in a pending status. 
2. Logged into the admin portal as the Global Admin, navigated to PIM, and opened the **Approve requests** queue. The inbound request from the lab user was captured properly, showing their target role and exact request time.

![Reviewing Incoming Pending Requests](./images/Approval_Requests.png)

3. Selected the pending line item to open the configuration enclave, reviewed the user's business justification, entered an administrative audit note ("*Approved request*"), and clicked **Submit**.

![Authorizing Active Elevation Request](./images/Approval_Request_Continued.png)

4. Switched back to the user's view and refreshed the console. The role moved successfully to **Active assignments**, showing a time-bound window authorized by the admin.

![Reviewing Post Approval Active Role State](./images/Active_Assignments_After_Approval.png)

5. Checked the central **Resource audit** logs to confirm that every phase of the identity lifecycle—from the baseline assignment to the user's request and the final admin approval—was tracked with clear cryptographic timestamps.



---

## 🔍 Defensive Verification & Tactical Takeaways

* **Eliminating Standing Access:** Moving the User Administrator role to an eligible state means the account operates with standard, non-privileged permissions during routine work, drastically shrinking the tenant's daily attack surface.
* **Dual-Custody Gatekeeping:** Enforcing an explicit approval pipeline ensures that no single user account can elevate its own privileges to manipulate directory accounts without a secondary administrator reviewing and authorizing the action.
* **Blast Radius Mitigation:** If the `TestUser` account is ever compromised, an attacker cannot immediately misuse directory control to reset passwords or create persistent backdoor access. The attack is entirely stalled by the mandatory approval barrier.
* **Immutable Compliance Auditing:** The PIM resource audit engine ensures every single elevation request, written justification, and administrative sign-off generates a clear event log, making it simple to track privileged actions during security incidents or compliance reviews.
