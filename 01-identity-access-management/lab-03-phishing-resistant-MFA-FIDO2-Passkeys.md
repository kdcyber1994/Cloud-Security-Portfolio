# Lab 03: Phishing-Resistant MFA via FIDO2 Passkeys

## 🎯 Lab Objective
The goal of this lab is to move an Entra ID tenant away from standard MFA methods like push notifications and configure custom Authentication Strengths to enforce hardware-bound FIDO2 Passkeys. This setup addresses Adversary-in-the-Middle (AiTM) phishing risks by verifying user onboarding, registering mobile devices as cryptographic tokens, and testing cross-device authentication flows.

## ⚙️ Architectural Context
Traditional MFA options like SMS, voice calls, or standard authenticator app push notifications can be intercepted by reverse-proxy phishing kits like Evilnginx. They are also highly vulnerable to MFA fatigue attacks. 

FIDO2 passkeys solve this issue by binding the authentication handshake directly to the official login domain. When a user tries to authenticate, the local browser verifies the site domain. If there is a mismatch caused by an attacker proxy, the hardware enclave drops the request. This completely eliminates credential harvesting and session hijacking attempts.

---

## 🛠️ Execution Walkthrough

### Task 3: Enable Phishing-Resistant MFA for Login
Before creating granular access rules, the FIDO2 security key authentication method must be enabled globally across the tenant.
1. Navigated to **Identity > Protection > Authentication methods > Policies** within the Entra admin center.
2. Selected **Passkey (FIDO2)** and toggled the state to **Enable** for the target users.

![Enabling FIDO2 Policy Globally](./images/Authentication-Policies.png)
![Authentication Methods Main Policy Dashboard](./images/Authentication-Methods.png)

#### Subtask 1 — Create a Custom Passkey Authentication Strength
1. From the **Authentication methods** menu, switched to the **Authentication strengths** tab and selected **+ New authentication strength**.

![Authentication Strengths Landing Page](./images/Authentication-Strength-Summary.png)

2. Named the configuration `Ignite phishing resistant MFA` and checked **Passkeys (FIDO2)** from the available methods list.

![Creating Custom Strength Profile](./images/New-Authentication-Strength.png)

3. Opened the **Advanced options** item directly under Passkeys (FIDO2). Marked the **Microsoft Authenticator** setting to enforce corporate app attestation via approved AAGUIDs, blocking unmanaged or consumer-grade password managers.

![Advanced FIDO2 Configuration Enclave](./images/Advanced-Options-FIDO2.png)
![Authentication Method Summary Verification](./images/Authentication-Method-Summary.png)

#### Subtask 2 — Add Authentication Strength to the Conditional Access Policy
1. Navigated to **Protection > Conditional Access > Policies** to review active access rules.

![Conditional Access Policies Dashboard](./images/Conditional-Access-Policies.png)

2. Opened the target pilot policy, which originally relied on the basic, legacy "Require multifactor authentication" grant rule.

![Legacy Policy Assessment](./images/MFA-Grant.png)

3. Modified the Access Controls under the **Grant** section: unmarked the generic MFA checkbox, marked **Require authentication strength**, and selected the custom `Ignite phishing resistant MFA` profile.

![Enforcing Modern Strengths](./images/MFA-Grant-Setup.png)

---

### Subtask 3: Configure an Android Passkey for User Login
Because creating a passkey requires a direct Bluetooth proximity handshake between the workstation and the mobile device, this workflow was executed using a dedicated test identity outside of a virtualized context.
1. Initiated a fresh login session. The account was stopped by a directory-enforced "More information required" prompt to force security registration.

![Directory Onboarding Intercept](./images/More-Info-Needed-Prompt.png)

2. Approved a baseline number-matching push notification to securely establish the initial onboarding session context.

![MFA Baseline Verification](./images/Authenticator-Request.png)
![Initial MFA Session Entry](./images/MFA-Try.png)

3. When prompted to create a passkey in Microsoft Authenticator, selected **Having trouble?** followed by **Create your passkey a different way** to explicitly trigger the cross-device registration flow. Selected **Next**.

![Hardware Onboarding Portal](./images/Android-Authenticator-Setup.png)

4. Initialized the local enrollment handler, named the hardware token `Android` for asset tracking, and used the phone's camera to scan the onboarding QR code to establish the secure Bluetooth link.

![Cryptographic Token Labeling](./images/Android-Authenticator-Setup-Naming.png)
![MFA Authenticator Baseline Registration](./images/MFA-Authenticator-Added.png)

5. Completed the biometric confirmation on the physical phone to finalize the enrollment and write the key to the hardware enclave.

![Registration Confirmation](./images/Android-Passkey-Created.png)

---

### Subtask 4: Log in with Phishing-Resistant MFA
1. Opened a new unauthenticated window and entered the target identity credentials. The browser intercepted the session with a request for a hardware credential.

![Proximity Challenge](./images/Choose-Passkey-Prompt.png)

2. Selected the Android device option to display the localized passkey login QR code.

![QR-Code Handshake](./images/Sign-In-With-Passkey.png)

3. Scanned the QR code and provided biometric verification on the handset. This executes a localized Bluetooth proximity check to verify the physical device is in front of the terminal.

![Biometric Verification Checkpoint](./images/Sign-In-With-Passkey-Success.png)

4. Confirmed successful portal entry as a zero-privilege user without sending a single password string or standard push notification over the wire.

![Successful Authenticated Session](./images/Logged-In-View-TestUser.png)

5. Audited the user's profile security info from the administrative portal to verify that the device-bound passkey was successfully registered alongside legacy options and marked as the primary high-assurance factor.

![Security Info State Verification](./images/Security-Info-TestUser.png)

---

## 🔍 Defensive Verification & Tactical Takeaways

* **Frictionless & Phishing-Resistant:** This method uses our Android device's camera to capture the QR code, which then instantly brings up our security key. This completely removes the need to type passwords, enter codes, or match numbers on a screen like traditional Authenticator push approvals.
* **Phishing Resistance Works:** Binding the Conditional Access rules to an explicit authentication strength prevents the account from authenticating via weaker, phishable code channels.
* **AAGUID Restrictions:** Restricting the accepted passkeys to the Microsoft Authenticator App stops users from registering unmanaged personal or consumer-grade password managers to store enterprise keys.
* **No More MFA Fatigue:** Because the validation relies on origin checks and local proximity, external threat actors cannot generate blind push spam notifications to force an employee into an accidental approval.
