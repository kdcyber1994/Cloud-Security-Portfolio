# Lab 02: Policy-Driven MFA via Microsoft Entra Conditional Access

## Objective
This lab demonstrates the migration of an enterprise cloud tenant from static, unconfigurable global Security Defaults to an agile, context-aware Zero Trust identity boundary using **Microsoft Entra Conditional Access**. The implementation establishes an explicit identity-verification checkpoint, targeting scoped user groups to enforce multifactor authentication (MFA) while mitigating administrative lockout risks and reducing post-lab attack surfaces.

---

## Architectural Workflow
To implement a resilient identity boundary, the configuration follows a deterministic validation path: 
Provisioning -> Boundary Grouping -> Tenant Posture Transition -> Rule Engineering -> Adversarial Verification.

---

## Technical Implementation Steps

### Step 1: Target Identity Provisioning
To isolate testing from production administrators, a dedicated non-privileged identity was created via **Identity > Users > All Users**.
* **Display Name:** `Test User`
* **User Principal Name (UPN):** `testuser@KdunnCloudLabs.onmicrosoft.com`
* **Administrative Roles:** None (Standard User)

*File Reference: `Screenshot 2026-05-21 113035.png`*

---

### Step 2: Directory Group Security Architecture
To implement scalable access control, permissions were mapped to a security group rather than an individual account. Navigated to **Identity > Groups > All Groups** and initialized the directory object:
* **Group Type:** Security
* **Group Name:** `MFA-Test-Group`
* **Membership Type:** Assigned
* **Direct Members:** `testuser`

*File References: `Screenshot 2026-05-21 113356.png`, `Screenshot 2026-05-21 113527.png`, `Screenshot 2026-05-21 113600.png`*

---

### Step 3: Tenant Posture Hardening & Transition
Microsoft Entra tenants ship with **Security Defaults** enabled out-of-the-box. While secure for basic setups, Security Defaults apply a broad policy block that restricts the use of granular Conditional Access logic. 

1. Navigated to **Identity > Overview > Properties**.
2. Selected **Manage security defaults** at the base of the blade.
3. Toggled the configuration to **Disabled**, selecting *'My organization is using Conditional Access'* as the technical justification.

This step safely transitioned the tenant control plane to an advanced enterprise posture, allowing custom evaluation rule blocks to take effect.

---

### Step 4: Engineering the Conditional Access Policy
Navigated to **Identity > Protection > Conditional Access** and initialized a new policy block designated as **"MFA Pilot"** utilizing the following technical specifications:

| Policy Blade Section | Parameter Configuration | Technical Justification |
| :--- | :--- | :--- |
| **Assignments: Users** | Include: `MFA-Test-Group` | Scopes policy strictly to test objects; eliminates global administrative lockout risks. |
| **Target Resources** | Include: **All resources** *(formerly 'All cloud apps')* | Ensures comprehensive coverage across all portals, APIs, and data management planes. |
| **Access Controls: Grant** | Control: **Require multifactor authentication** | Intercepts tokens to mandate secondary cryptographic proof before issuing a SAML/OIDC claim. |
| **Enable Policy** | State: **On** | Bypasses *Report-only* mode to immediately enforce live evaluation. |

*File References: `Screenshot 2026-05-21 114232.png`, `Screenshot 2026-05-21 115213.png`, `Screenshot 2026-05-21 115437.png`, `Screenshot 2026-05-21 120038.png`*

---

## Defensive Verification & Adversarial Emulation

To validate that the policy engine accurately intercepts out-of-compliance authentication loops, an adversarial sign-in test was executed from an unauthenticated context:

1. Initialized an isolated browser session (**Chrome Incognito Mode**) to clear pre-existing token caches and session cookies.
2. Navigated directly to the Microsoft Entra admin center (`portal.azure.com`) and entered the UPN for `testuser`.
3. **Verification Result:** The authentication handshake successfully processed the first-factor password validation, parsed the user's directory metadata, matched their membership within `MFA-Test-Group`, and triggered the **MFA Pilot** policy interception layer. 
4. The user was blocked from accessing the portal and rerouted to the mandatory `Keep your account secure` enrollment workflow to register a modern verification token.

*File References: `Screenshot 2026-05-21 120447.jpg`, `Screenshot 2026-05-21 120553.jpg`*

---

## Identity Lifecycle Decommissioning
> 🔒 **Security Best Practice:** Following successful functional validation of the identity checkpoint, the `testuser` account status was explicitly modified within the Entra directory to **Account Status: Disabled**. In production cloud architecture, leaving unmonitored test accounts active provides opportunistic actors a vector for password spraying and initial access. Defensive cleanup reduces the tenant's persistent attack surface.
