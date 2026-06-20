# Lab 05: Restricting Key Vault Public Perimeters and Implementing Multi-Layer Access Validation

## 🎯 Lab Objective
The goal of this lab is to shut down internet-facing exposure on an Azure Key Vault. By setting up a strict network firewall and combining it with Azure RBAC, this setup locks down the vault so it can only be reached by an authorized engineer from a trusted home network.

## ⚙️ Architectural Context
By default, a new Key Vault is completely open to the public internet. Even though users still need valid credentials to log in, leaving the endpoint exposed means anyone can attempt to scan it or launch brute-force attacks. 

To secure sensitive secrets and meet standard compliance rules, we need a layered security approach. First, the network firewall needs to drop all traffic by default except for specific whitelisted IP addresses. Second, we need to use modern Azure RBAC instead of legacy access policies. Legacy access policies are a major security risk because anyone with a basic Contributor role on the vault can rewrite the policy and grant themselves full access to the secrets. Azure RBAC stops this by separating vault management from actual data access.

---

## 🛠️ Execution Walkthrough

### Provision a Hardened Key Vault with Deletion Defenses
The first phase focuses on building a secure vault from scratch and locking down settings that cannot be easily changed later.
1. Navigated to the Azure Portal and started a new Key Vault deployment under the name **`kv-cyberlab-secure`**. Selected the **Premium SKU** to make sure cryptographic keys are backed by high-level hardware security modules.

![Initializing Key Vault Provisioning](./images/Create_Key_Vault.png)

2. Opened the recovery tabs and set the soft-delete retention to **7 days** for the purposes of this lab. Checked the box to **Enable purge protection** to ensure that no one, not even a subscription owner, can permanently wipe out secrets before the retention period is up.

![Reviewing Completed Configuration Profile](./images/Create_Key_Vault_Summary.png)

3. Reviewed the final parameters and created the resource inside the `rg-cyberlabs` resource group.

![Confirming Completed Infrastructure Deployment](./images/Key_Vault_Overview.png)

---

### Subtask 1: Enforce Governance at Scale with Azure Policy
To make sure no other teams can spin up unsecured vaults without deletion protection, an organization-wide Azure Policy was put in place.
1. Opened the **Policy** dashboard, went to **Definitions**, and searched for Key Vault rules.

![Assessing Available Key Vault Built-In Definitions](./images/Policy_Definitions.png)

2. Selected the rule **`Key vaults should have deletion protection enabled`** and clicked **Assign policy**. Set the scope to apply to the entire subscription.

![Configuring Assignment Target Scope](./images/Key_Vault_Assign_Policy.png)

3. On the Parameters tab, changed the default setting from Audit to **Deny**. This will actively block any future vault deployments if they do not have purge protection turned on.

![Modifying Policy Enforcement Parameters to Deny](./images/Key_Vault_Assign_Policy_Parameters_Cont_Deny.png)

4. Saved the policy assignment and confirmed it was active on the dashboard.

![Verifying Live Policy Definition Baseline](./images/Key_Vault_Assign_Policy_Summary.png)

5. **Testing the Policy Guardrail:** Ran a test by trying to deploy a separate vault named `Compliance-Test` with purge protection turned off. The Azure engine immediately caught it, failed the validation check, and blocked the deployment from happening.

![Validating Scale-Level Policy Enforcement Block](./images/Key_Vault_Failed_Validation_Check_Error.png)

---

### Subtask 2: Lock Down the Network Firewall
With global policy locked down, the next step was restricting the network perimeter on the live production vault.
1. Opened `kv-cyberlab-secure`, went to **Networking**, and switched the main setting to **Allow public access from specific virtual networks and IP addresses**.
2. Looked up the home router public WAN IP address. Entered the IP range into the firewall rule box. Note that standard private local IPs like `10.x.x.x` cannot be used here.

![Configuring Firewall IP Whitelist Boundary](./images/Key_Vault_Firewall_IP_Setup.png)

3. Left the box checked for **Allow trusted Microsoft services to bypass this firewall** so that background tasks like Azure Backup do not break. Clicked Apply to save the network boundary.

---

### Subtask 3: Test the Security Order of Operations
Before assigning permissions, a baseline test was run from an unapproved network to see how Azure handles unauthorized requests.
1. Turned on a commercial VPN on a mobile device to route traffic through an untrusted IP address out of Detroit.

![Establishing Adversarial VPN Network Tunnel](./images/VPN_Connected.jpg)

2. Logged into the Azure Portal on the mobile device and went to the key vault console.
3. Tried to click into the **Secrets** menu. The portal immediately blocked access with an **`RBAC Permission Failure`** message.

![Observing Initial Identity Authorization Block](./images/RBAC_Block.jpg)

*Takeaway:* This proves that Azure always checks identity and permissions before it looks at the network firewall. Because this account did not have data permissions yet, it was stopped at the identity layer.

---

### Subtask 4: Assign Azure RBAC and Validate the Firewall
To finish the setup, permissions were assigned to pass the identity check so the network firewall could be properly tested.
1. Switched back to the home computer, went to the vault access configuration settings, and verified the permission model was set to **Azure role-based access control**.
2. Navigated to **Access control (IAM)** and clicked **Add role assignment**.

![Initializing Granular Role Assignment Workflow](./images/Key_Vault_IAM.png)

3. Selected the **Key Vault Secrets Officer** role, which allows full management of secrets without giving away control over the infrastructure itself.

![Selecting Target Data Plane Secrets Officer Role](./images/Key_Vault_IAM_Role_Assignment.png)

4. Assigned the role directly to the primary user account and saved it.

![Finalizing Direct Identity Mapping](./images/Key_Vault_Role_Assignment_Cont.png)

5. **Testing the Firewall Boundary:** Went back to the mobile phone on the VPN and refreshed the session. 

![Confirming Active Portal Session via Untrusted Network](./images/Login_Confirmed.jpg)

6. Tried to open the secrets tab again. With the identity check now passing, the network layer stepped in. The vault successfully blocked the connection and displayed a **`Firewall is turned on`** error message. This proves that an attacker cannot steal secrets even if they manage to compromise the user account.

![Confirming Network Perimeter Hardening Success](./images/Firewall_Block.jpg)

7. Disconnected the VPN on the phone to go back to the whitelisted home network. Refreshed the page and full access to the secrets menu was restored instantly with no errors.

![Verifying Seamless Authorized Egress Access](./images/KeyVault_Without_VPN.jpg)

---

## 🔍 Defensive Verification & Tactical Takeaways

* **Perimeter Protection:** Turning on the Key Vault firewall means the data plane is hidden from the open web. Unauthorized internet traffic cannot connect to search for secrets.
* **Credential Protection:** If user login credentials are leaked, the network firewall acts as a critical second line of defense. The attacker is blocked unless they are connecting from the whitelisted corporate IP block.
* **Separation of Duties:** Using Azure RBAC ensures that platform engineers can handle network settings or tags without automatically getting access to read sensitive passwords or encryption keys.
* **Future Recommendations:** While IP firewalls work well for static engineering workstations, production environments should move to **Azure Private Link**. A Private Endpoint removes the public endpoint entirely and gives the vault a private internal IP address inside a secure Virtual Network.
