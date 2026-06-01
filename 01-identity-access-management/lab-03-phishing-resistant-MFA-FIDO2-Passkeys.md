# Lab 03: Phishing-Resistant MFA via FIDO2 Passkeys

## 🎯 Lab Objective
The goal of this lab is to move an Entra ID tenant away from standard MFA methods like push notifications and configure custom Authentication Strengths to enforce hardware-bound FIDO2 Passkeys. This setup stops modern proxy-based phishing attacks by securely onboarding the user, turning their phone into a passkey, and testing the login flow between devices.

## ⚙️ Architectural Context
Standard MFA options like SMS, voice calls, or basic authenticator app push notifications are no longer enough to stop modern attacks. Attackers can easily bypass these methods using phishing kits to clone login pages, or they can just spam a user with notifications until they accidentally click approve.

FIDO2 passkeys fix this security gap by tying your login directly to the actual website domain. When you try to log in, your browser and your phone talk to each other to verify the exact site you are on. If a hacker tries to trick you using a look-alike phishing link, the passkey recognizes the fake domain and refuses to authenticate. This completely cuts off credential harvesting, session hijacking, and push notification fatigue.

---

## 🛠️ Execution Walkthrough

### Enable Phishing-Resistant MFA for Login
Before creating granular access rules, the FIDO2 security key authentication method must be enabled globally across the tenant.
1. Navigated to **Identity > Protection > Authentication methods > Policies** within the Entra admin center.
2. Selected **Passkey (FIDO2)** and toggled the state to **Enable** for the target users.

![Enabling FIDO2 Policy Globally](./images/Authentication-Policies.png)


#### Subtask 1 — Create a Custom Passkey Authentication Strength
1. From the **Authentication methods** menu, switched to the **Authentication strengths** tab and selected **+ New authentication strength**.

![Authentication Strengths Landing Page](./images/Authentication-Strength-Summary.png)

2. Named the configuration `Ignite phishing resistant MFA` and checked **Passkeys (FIDO2)** from the available methods list.

![Creating Custom Strength Profile](./images/New-Authentication-Strength.png)

3. Opened the Advanced options link under Passkeys (FIDO2) and checked the Microsoft Authenticator option. This enforces strict app restrictions so users can only use the official app, blocking unmanaged or personal password managers.

![Advanced FIDO2 Configuration Enclave](./images/Advanced-Options-FIDO2.png)
![Authentication Method Summary Verification](./images/Authentication-Method-Summary.png)

#### Subtask 2 — Add Authentication Strength to the Conditional Access Policy
1. Navigated to **Protection > Conditional Access > Policies** to review active access rules.

![Conditional Access Policies Dashboard](./images/Conditional-Access-Policies.png)

2. Opened the target MFA Pilot policy, which originally relied on the basic, legacy "Require multifactor authentication" grant rule.

![Legacy Policy Assessment](./images/MFA-Grant.png)

3. Modified the Access Controls under the **Grant** section: unmarked the generic MFA checkbox, marked **Require authentication strength**, and selected the custom `Ignite phishing resistant MFA` profile.

![Enforcing Modern Strengths](./images/MFA-Grant-Setup.png)

---

### Subtask 3: Configure an Android Passkey for User Login
Since setting up a passkey requires a direct Bluetooth connection between your computer and phone, this workflow was done on a local PC instead of a virtual machine. 
1. When logging in for the first time, Entra paused the session with a "More information required" prompt to force the security registration, selected **Next**.

![Directory Onboarding Intercept](./images/More-Info-Needed-Prompt.png)

2. When prompted to create a passkey in Microsoft Authenticator, selected **Next**.

![Hardware Onboarding Portal](./images/Android-Authenticator-Setup.png)

3. Named the hardware key `Android` for asset tracking, and used the phone's camera to scan the onboarding QR code to establish the secure Bluetooth link.

![Cryptographic Token Labeling](./images/Android-Authenticator-Setup-Naming.png)

4. Completed the biometric scan on the Android phone to finish the setup and securely save the passkey to the device.

![Registration Confirmation](./images/Android-Passkey-Created.png)

---

### Subtask 4: Log in with Phishing-Resistant MFA
1. Opened a new unauthenticated window and entered the target identity credentials. The browser intercepted the session with a request for a hardware credential.

![Proximity Challenge](./images/Choose-Passkey-Prompt.png)

2. Selected the Android device option to display the localized passkey login QR code.

![QR-Code Handshake](./images/Sign-In-With-Passkey.png)

3. Scanned the QR code and used biometrics on the phone, which triggers a quick Bluetooth check to make sure the device is physically next to the computer.

![Biometric Verification Checkpoint](./images/Sign-In-With-Passkey-Success.png)

4. Confirmed successful entry into the portal as a standard test user without typing a single password or dealing with standard push notifications.

![Successful Authenticated Session](./images/Logged-In-View-TestUser.png)

5. Checked the user's security info in the admin portal to confirm the new passkey was successfully registered alongside old methods and set as the main high-security option.

![Security Info State Verification](./images/Security-Info-TestUser.png)

---

## 🔍 Defensive Verification & Tactical Takeaways

* **Frictionless & Phishing-Resistant:** This method uses our Android device's camera to capture the QR code, which then instantly brings up our security key. This completely removes the need to type passwords, enter codes, or match numbers on a screen like traditional Authenticator push approvals.
* **Phishing Resistance Works:** Binding the Conditional Access rules to an explicit authentication strength prevents the account from authenticating via weaker, phishable code channels.
* **AAGUID Restrictions:** Restricting the accepted passkeys to the Microsoft Authenticator App stops users from registering unmanaged personal or consumer-grade password managers to store enterprise keys.
* **No More MFA Fatigue:** Because the validation relies on origin checks and local proximity, external threat actors cannot generate blind push spam notifications to force an employee into an accidental approval.
