# Lab 03: Phishing-Resistant MFA via Entra ID FIDO2 Passkeys

## 🎯 Lab Objective
The goal of this lab is to move an Entra ID tenant away from standard MFA methods like push notifications and configure custom Authentication Strengths to enforce hardware-bound FIDO2 Passkeys. This setup addresses Adversary-in-the-Middle (AiTM) phishing risks by verifying user onboarding, registering mobile devices as cryptographic tokens, and testing cross-device authentication flows.

## ⚙️ Architectural Context
Traditional MFA options like SMS, voice calls, or standard authenticator app push notifications can be intercepted by reverse-proxy phishing kits like Evilnginx. They are also highly vulnerable to MFA fatigue attacks. 

FIDO2 passkeys solve this issue by binding the authentication handshake directly to the official login domain. When a user tries to authenticate, the local browser verifies the site domain. If there is a mismatch caused by an attacker proxy, the hardware enclave drops the request. This completely eliminates credential harvesting and session hijacking attempts.

---

## 🛠️ Execution Walkthrough

### Step 1: Initial Identity Onboarding Interception
Unconfigured identities must be forced through a controlled enrollment loop before they can register high-assurance credentials.
1. Attempted to log into the administrative portal using `testuser@kdunncloudlabs.onmicrosoft.com`.
2. The account was stopped by a directory-enforced "More information required" prompt to force security registration.

![Directory Onboarding Intercept](./images/More-Info-Needed-Prompt.jpg)

3. Clicked Next and completed a standard number-matching push notification to establish a verified baseline session.

![MFA Baseline Verification](./images/Authenticator-Request.jpg)

### Step 2: Enrolling the Device-Bound FIDO2 Passkey
With a secure session established, the next step is to bind a mobile device passkey to the account.
1. Navigated to the registration portal at `https://mysignins.microsoft.com/register` to trigger the passkey configuration workflow.

![Hardware Onboarding Portal](./images/Android-Authenticator-Setup.jpg)

2. Launched the WebAuthn client handler and named the new hardware token `Android` to keep track of the registered asset.

![Cryptographic Token Labeling](./images/Android-Authenticator-Setup-Naming.jpg)

3. Completed the biometric enrollment on the physical phone to finalize the key creation screen.

![Registration Confirmation](./images/Android-Passkey-Created.jpg)

4. Audited the profile security settings to verify that the device-bound passkey was successfully registered alongside legacy options.

![Security Info State Verification](./images/Security-Info-TestUser.png)

### Step 3: Upgrading Conditional Access to Authentication Strengths
After registering the key, the tenant policy needs to change from accepting generic MFA to strictly requiring phishing-resistant credentials.
1. Opened the existing "MFA Pilot" Conditional Access policy which originally relied on the basic "Require multifactor authentication" grant rule.

![Legacy Policy Assessment](./images/MFA-Grant.png)

2. Modified the policy controls to select "Require authentication strength" and mapped it to the custom `Ignite phishing resistant MFA` profile. This explicitly blocks standard push notifications or numeric codes.

![Enforcing Modern Strengths](./images/MFA-Grant-Setup.png)

### Step 4: End-to-End Cross-Device Authentication Verification
The final step is testing the updated login path from a fresh, unauthenticated context.
1. Opened a separate browser window and initiated a login attempt. The browser intercepted the session with a Windows Security window asking for a passkey from a mobile device.

![Proximity Challenge](./images/Choose-Passkey-Prompt.jpg)

2. Selected the mobile device option to generate a localized WebAuthn broker QR code. This establishes an out-of-band proximity connection over Bluetooth.

![QR-Code Handshake](./images/Sign-In-With-Passkey.jpg)

3. Scanned the QR code and provided biometric confirmation on the phone, causing the browser to immediately authorize the device connection.

![Biometric Verification Checkpoint](./images/Sign-In-With-Passkey-Success.jpg)

4. Confirmed successful tenant entry as a zero-privilege user without sending a single password string across the wire.

![Successful Authenticated Session](./images/Logged-In-View-TestUser.png)

---

## 🔍 Defensive Verification & Tactical Takeaways
