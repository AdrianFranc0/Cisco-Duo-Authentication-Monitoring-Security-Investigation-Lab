# Cisco Duo Authentication Monitoring & Security Investigation Lab

## Overview

This project builds upon my previous **Active Directory + Cisco Duo MFA Deployment for Windows Logon** lab, where Cisco Duo was integrated with Active Directory and deployed to a Windows endpoint to enforce Multi-Factor Authentication (MFA).

After successfully deploying Duo MFA, I expanded the environment into a security-focused investigation lab to simulate how security analysts review authentication activity, identify access control risks, and investigate potential security findings related to identity and access management (IAM).

This project focuses on authentication monitoring, access reviews, user lifecycle management, and security investigations aligned with industry concepts from OWASP, NIST, and security operations best practices.

---

## Previous Project

This lab builds upon the environment created in my previous project:

🔗 **Active Directory + Cisco Duo MFA Deployment for Windows Logon**

https://github.com/AdrianFranc0/Active-Directory-Cisco-Duo-MFA-Deployment-for-Windows-Logon

The previous project covered:

- Active Directory user creation
- Cisco Duo integration
- Windows Logon MFA deployment
- User enrollment
- Successful Duo Push authentication validation

This project focuses on the monitoring, investigation, and analysis side of authentication security.

---

## Technologies Used

- Cisco Duo
- Active Directory
- Windows Server 2019
- Windows 10
- Duo Mobile
- Identity & Access Management (IAM)
- Multi-Factor Authentication (MFA)
- Authentication Log Analysis

---

## Lab Topology

**Description:**

The environment consists of a Windows Server 2019 Domain Controller, a Windows 10 client workstation, Active Directory, and Cisco Duo MFA for Windows Logon.

User authentication is validated through Active Directory while Cisco Duo provides the second authentication factor.

<p align="center">
Lab Topology<br/>
<img src="https://i.imgur.com/W04Vtlm.png" width="800" style="height:auto;" alt="Cisco Duo Authentication Monitoring Lab Topology"/>
<br /><br />
</p>

---

# Investigation 1 – MFA Fatigue Authentication Review

## Description

A review was conducted after observing multiple consecutive Duo Push notifications being generated for a user account.

Authentication logs showed several denied authentication requests followed by a successful approval. This behavior may be associated with MFA fatigue attacks, where repeated authentication requests are sent in an attempt to pressure a user into approving a login request they did not initiate.

The purpose of this investigation was to review authentication activity and identify indicators that may warrant additional analysis.

### Investigation Questions

- Were authentication requests expected by the user?
- Did the activity occur during normal business hours?
- How many requests occurred within a short time period?
- Were requests repeatedly denied before approval?
- Were there any unusual authentication patterns present?

### Findings

- Multiple consecutive authentication requests were observed.
- Several requests were denied before authentication was eventually approved.
- Authentication logs provided visibility into user behavior and authentication outcomes.
- Repeated authentication requests may warrant additional investigation depending on business context and user activity.

<p align="center">
MFA Fatigue Authentication Activity<br/>
<img src="https://i.imgur.com/bEiDTE9.png" width="800" style="height:auto;" alt="MFA fatigue authentication activity review"/>
<br /><br />
</p>

---

# Investigation 2 – Privilege Creep Access Review

## Description

An access review was performed to identify potential privilege creep within the environment.

Privilege creep occurs when users retain permissions, access, or group memberships from previous roles after transitioning to a different department or job function.

During this review, a user who had transitioned from an IT role to a Payroll role was examined. Although the user's organizational placement had changed, previous IT-related access assignments remained present. This included Active Directory group memberships that synced over to Duo, leaving the user mistakenly bound to privileged Duo groups and access policies intended only for IT personnel.
The account was also reviewed within Duo to verify that the user remained active and capable of authenticating into protected systems.

### Investigation Questions

- Does the user still require privileged access?
- Are current permissions aligned with the user's current role and the way they authenticate?
- Were legacy permissions removed following role transition?
- Does the account remain active and capable of authentication?

### Findings

- The user's organizational role had changed from IT to Payroll.
- Legacy IT-related access assignments remained present.
- The account remained active within Duo.
- Access reviews are important for maintaining least privilege and reducing unnecessary access exposure.

<p align="center">
Privilege Creep Review - Active Directory<br/>
<img src="https://i.imgur.com/ygLyLeu.png" width="800" style="height:auto;" alt="Privilege creep access review in Active Directory"/>
<br /><br />
</p>

<p align="center">
Privilege Creep Review - Duo Authentication Status = user is still authenticating and active<br/>
<img src="https://i.imgur.com/ssWJhQt.png" width="800" style="height:auto;" alt="Privilege creep review in Duo"/>
<br /><br />
</p>

---
## Industry Framework References

This project maps directly to industry-standard cybersecurity frameworks, demonstrating how the monitoring, investigation, and control of identity infrastructure align with compliance and threat-modeling benchmarks.

### 1. NIST SP 800-63B – Digital Identity Guidelines
The National Institute of Standards and Technology (NIST) guidelines outline requirements for robust authentication and identity verification. This lab directly demonstrates and validates the following controls:
* **IA-2: Identify Proofing & Authentication:** Implementing and enforcing Multi-Factor Authentication (MFA) via Cisco Duo to secure the Active Directory domain environment.
* **IA-5: Authenticator Management:** Restricting and monitoring out-of-band authenticators (Duo Mobile Push notifications) to ensure they are bound tightly to authorized users.
* **IA-7: Authentication Context:** Analyzing authentication telemetry (IP addresses, Device types, Time of authentication) within the Duo Admin Console to validate the trustworthiness of login events.

### 2. MITRE ATT&CK Framework
The investigations conducted in this lab simulate the detection and analysis of specific adversarial tactics and techniques:
* **T1621 – Multi-Factor Authentication Request Generation (MFA Fatigue):** Evaluated in *Investigation 1*. This maps directly to adversaries spamming a user's device with push notifications until they inadvertently hit "Approve."
* **T1078 – Valid Accounts:** Demonstrated across both investigations. Adversaries frequently attempt to exploit valid credentials to blend in with normal traffic; this lab shows how log analysis isolates anomalies from legitimate accounts.
* **T1136 – Create Account / Privilege Abuse:** Addressed in *Investigation 2*. Left unchecked, stale permissions from role transitions act as unintentional "backdoors" that malicious actors can abuse.

### 3. OWASP Authentication Failures (A07:2021)
The Open Web Application Security Project (OWASP) Top 10 highlights **Identification and Authentication Failures** as a critical vulnerability class. This project applies those principles to enterprise infrastructure:
* **Credential Misuse Mitigation:** Utilizing adaptive policies and active log monitoring to prevent unauthorized sessions resulting from leaked corporate credentials.
* **Excessive Trust & Over-Privilege:** *Investigation 2* specifically tackles the remediation of "Privilege Creep," ensuring that the security posture enforces strict Least Privilege modeling across the user lifecycle.

---

# Security Concepts Demonstrated

- Multi-Factor Authentication (MFA)
- Authentication Monitoring
- Identity & Access Management (IAM)
- Access Reviews
- Least Privilege
- Authentication Event Analysis
- Security Investigations
- User Lifecycle Management
- Authentication Log Analysis

---

## Control 1 – Access Governance Policy

**Description:**

Following the privilege creep investigation, an access governance control was implemented to prevent authentication attempts from accounts undergoing access review.

A dedicated Active Directory security group named **Restricted-Users** was used to simulate a user review process. Accounts placed into this group represent users whose permissions require validation due to role changes, department transfers, or other identity and access management concerns.

<p align="center">
User Assigned to Restricted Users Group<br/>
<img src="https://i.imgur.com/MXJszmM.png" width="800" style="height:auto;" alt="User assigned to Restricted Users group"/>
<br /><br />
</p>

---

### Restricted Users Group Added to Duo

**Description:**

To support centralized authentication governance, a corresponding **Restricted-Users** group was created within Duo. This allows security administrators to apply authentication controls to users undergoing access reviews.

<p align="center">
Restricted Users Group Added to Duo<br/>
<img src="https://i.imgur.com/hHQgbET.png" width="800" style="height:auto;" alt="Restricted Users group added to Duo"/>
<br /><br />
</p>

---

### Access Review Policy Implementation

**Description:**

A Duo policy named **Restricted Users Policy** was created and assigned to the Restricted Users group.

This policy was designed to prevent authentication attempts while user access requirements were reviewed. Implementing temporary authentication restrictions helps reduce the risk of unauthorized access resulting from privilege creep or stale permissions.

<p align="center">
Restricted Users Policy Configuration<br/>
<img src="https://i.imgur.com/zVxS8FY.png" width="800" style="height:auto;" alt="Restricted Users policy configuration"/>
<br /><br />
</p>

---

### Control Validation

**Description:**

After assigning the test user to the Restricted Users group and applying the associated Duo policy, authentication testing was performed.

The authentication attempt was successfully denied, confirming that the implemented control prevented access while the account remained under review.

This validation demonstrates how access governance controls can be used to reduce risk associated with privilege creep and excessive permissions.

<p align="center">
Authentication Denied Validation<br/>
<img src="https://i.imgur.com/wayz7Y5.png" width="800" style="height:auto;" alt="Authentication denied after policy enforcement"/>
<br /><br />
</p>

---

## Control 2 – Authentication Monitoring & MFA Fatigue Mitigation

**Description:**

During authentication log review, multiple denied Duo Push requests followed by a successful approval were observed.

While intentionally generated within the lab environment, similar patterns may be associated with MFA fatigue attacks, where repeated authentication prompts are sent in an attempt to obtain an accidental approval from a user.

Authentication monitoring provides security teams with visibility into user activity, authentication methods, timestamps, devices, and geographic information, allowing investigators to identify unusual authentication patterns and determine whether additional investigation is required.

<p align="center">
Authentication Monitoring and Log Review<br/>
<img src="https://i.imgur.com/kbamgFU.png" width="800" style="height:auto;" alt="Authentication monitoring and MFA fatigue indicators"/>
<br /><br />
</p>

---

### Recommended Control A – Authentication Threshold Monitoring

**Description:**

Organizations should establish monitoring thresholds for repeated authentication failures and abnormal authentication activity.

Repeated denied authentication requests occurring within a short timeframe may indicate MFA fatigue attempts, user confusion, or other authentication-related security concerns.

Example review criteria:

```text
5 denied authentication requests
within 10 minutes
=
Security Review Required
```

Monitoring authentication thresholds helps security teams identify patterns commonly associated with MFA fatigue attacks and other authentication-related threats.

---

### Recommended Control B – Geographic & Location Review

**Description:**

Authentication events should be reviewed for geographic anomalies that may indicate unauthorized access attempts.

Examples include:

- Foreign country logins
- Impossible travel scenarios
- Anonymous VPN or proxy usage
- Unrecognized locations
- Unexpected authentication patterns

Location analysis provides additional context during authentication investigations and assists security teams in validating user identity and access activity.

---

## Control 3 – Identity Lifecycle Management & Directory Synchronization

**Description:**

The privilege creep investigation identified a user who had transitioned from one department to another while still retaining access associated with a previous role.

This highlights the importance of identity lifecycle management processes and ongoing access reviews to ensure permissions remain aligned with current job responsibilities.

As part of the remediation process, Active Directory synchronization with Duo was evaluated as a method for improving identity lifecycle management.

Synchronizing user accounts, group memberships, and account status information between Active Directory and Duo helps ensure authentication policies remain aligned with the organization's identity source of truth.

Potential benefits include:

- Automatic synchronization of user accounts
- Group membership synchronization
- Faster removal of access during employee transfers
- Reduced risk of stale accounts
- Reduced risk of privilege creep
- Improved consistency between identity systems

---

# ✅ Project Outcome

This project demonstrated how authentication logs, identity systems, and access governance controls can be used together to identify, investigate, and remediate authentication and authorization risks.

Key concepts demonstrated:

- MFA Fatigue Detection
- Authentication Monitoring
- Privilege Creep Investigation
- Access Governance Controls
- Identity Lifecycle Management
- Security Control Validation
- Authentication Policy Enforcement
- Identity & Access Management (IAM)

---

# 🎯 Skills Demonstrated

- Identity & Access Management (IAM)
- Authentication Monitoring
- Multi-Factor Authentication (MFA)
- Access Governance
- Privilege Management
- Security Operations
- Security Investigations
- Identity Lifecycle Management
- Active Directory
- Cisco Duo Administration
- Authentication Policy Enforcement
- Security Control Validation
- Security Compliance Concepts

