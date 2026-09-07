# AuthGuard: Adaptive Multi-Factor Authentication & Single Sign-On Gateway

[![Security Standard](https://img.shields.io/badge/Security%20Standard-NIST%20SP%20800--63B-0A5C36.svg)](https://pages.nist.gov/800-63-3/sp800-63b.html)
[![OAuth 2.0](https://img.shields.io/badge/Protocol-OAuth%202.0%20%7C%20OIDC-2B5797.svg)](https://datatracker.ietf.org/doc/html/rfc6749)
[![PKCE](https://img.shields.io/badge/RFC%207636-PKCE%20Enforced-C0392B.svg)](https://datatracker.ietf.org/doc/html/rfc7636)
[![Token Algorithm](https://img.shields.io/badge/Token%20Signing-RS256%20Pinned-D35400.svg)](https://datatracker.ietf.org/doc/html/rfc7519)
[![Frontend](https://img.shields.io/badge/Frontend-React%2018%20SPAs-00D8FF.svg)](https://reactjs.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-F1C40F.svg)](https://opensource.org/licenses/MIT)

AuthGuard is an enterprise Identity & Access Management (IAM) and Single Sign-On (SSO) gateway engineered to secure the shared authentication boundary of a distributed, multi-portal academic system. Built on **Zero-Knowledge Credential Isolation** and **Defense-in-Depth**, AuthGuard completely decouples identity verification from frontend client applications.

The gateway integrates **Auth0 Universal Login**, **OAuth 2.0 with PKCE (RFC 7636)**, adaptive **Multi-Factor Authentication (WebAuthn/TOTP)**, and an active attack-protection layer to protect user sessions across independent origins while neutralizing brute-force attempts, credential stuffing, and session hijacking.

---



## System Architecture

```
                                  +-------------------------------------------------------------+
                                  |         Identity Provider (Hosted Universal Login)          |
                                  |                                                             |
                                  |  [Layered Attack Protection]                                |
                                  |  - Breached Password Shielding (Real-Time Hash Screening)   |
                                  |  - Account Lockout Engine (10 Consecutive Fails -> Lock)    |
                                  |  - Per-IP Rate Limiting & Bot Defense (reCAPTCHA v2 Always) |
                                  |                                                             |
                                  |  [Adaptive Multi-Factor Authentication]                     |
                                  |  - Hardware Biometrics & FIDO2 Keys (WebAuthn)              |
                                  |  - Time-Based One-Time Passwords (TOTP Authenticator)       |
                                  |                                                             |
                                  |  [Token Issuance & Lifespan Governance]                     |
                                  |  - Asymmetric RS256 Signature (Public Key via JWKS)         |
                                  |  - ID Token TTL: 900s (15 min) | Refresh Idle Timeout: 300s |
                                  +-------------------------------------------------------------+
                                                               ^
                                                               |
                     OAuth 2.0 + PKCE (RFC 7636) / Single Trusted SSO Session Boundary
                                                               |
           +---------------------------+-----------------------+-----------------------+---------------------------+
           |                           |                                               |                           |
           v                           v                                               v                           v
+----------------------+   +-----------------------+                       +-----------------------+   +-----------------------+
|     Main Portal      |   |  Course Registration  |                       |     Exams Portal      |   |     Events Portal     |
|    localhost:3000    |   |    localhost:3001     |                       |    localhost:3002     |   |    localhost:3003     |
| (Public SPA Client)  |   |  (Public SPA Client)  |                       |  (Public SPA Client)  |   |  (Public SPA Client)  |
+----------------------+   +-----------------------+                       +-----------------------+   +-----------------------+
           |                           |                                               |                           |
           +---------------------------+-----------------------------------------------+---------------------------+
                                                       |
                                                       v
                                     [ Centralized Single Logout (SLO) ]
                 (Terminating a session on any portal instantly destroys the central IdP session)
```

---

## Multi-Portal Ecosystem

AuthGuard orchestrates and protects four independent React Single-Page Applications (SPAs), each running on an isolated origin:

| Portal | Origin / Port | Academic Functional Scope | Security Client Model |
| :--- | :--- | :--- | :--- |
| **Main Portal** | `http://localhost:3000` | Central student dashboard, profile overview, and gateway entry. | Public Client (OAuth 2.0 + PKCE, Zero Secrets). |
| **Course Registration** | `http://localhost:3001` | Course schedule planning, add/drop workflows, and academic records. | Public Client (OAuth 2.0 + PKCE, Zero Secrets). |
| **Exams Portal** | `http://localhost:3002` | Examination schedules, seat allocations, and verified grades. | Public Client (OAuth 2.0 + PKCE, Zero Secrets). |
| **Events Portal** | `http://localhost:3003` | Campus seminars, hackathons, and activity registration. | Public Client (OAuth 2.0 + PKCE, Zero Secrets). |

---

## Core Security Pillars

* **Zero-Knowledge Credential Isolation:** Client portals never receive, inspect, or store raw user credentials. Authentication runs strictly on the hosted Identity Provider domain, containing blast radius in the event of frontend compromise.
* **OAuth 2.0 with PKCE (RFC 7636):** Eliminates static secrets in frontend runtimes. Cryptographically generated code verifiers and SHA-256 challenges prevent authorization code interception and replay attacks.
* **Adaptive Multi-Factor Authentication (MFA):** Passwords alone are treated as insufficient proof of identity. Secondary authentication is enforced via WebAuthn hardware biometrics (TouchID, Windows Hello, physical security keys) or TOTP authenticator apps.
* **Unified SSO & Single Logout (SLO):** A shared session cookie enables seamless cross-portal navigation without re-prompting credentials. Terminating a session from any portal immediately clears the central session across all four portals.
* **Cryptographic Token Integrity:** All tokens are signed using asymmetric RS256 with algorithm pinning to prevent key-confusion attacks. ID tokens expire in 900 seconds (15 minutes), and sessions terminate after 300 seconds of inactivity.

---

## Hardened Attack Protection

| Defensive Control | Configuration & Rule | Addressed Threat |
| :--- | :--- | :--- |
| **Breached Password Shielding** | Real-time screening against global leaked credential datasets during registration, login, and reset. | Credential Stuffing & Password Reuse. |
| **Account Lockout Engine** | Automatic account blocking after 10 consecutive failed logins across any IP; owner notified by email. | Distributed Brute-Force Attacks. |
| **Suspicious IP Throttling** | Maximum 5 failed logins (renewing at 50/day), 3 signups (20/day), and 10 token exchanges (144/day) per IP. | Distributed Denial of Service (DoS) & Scrapers. |
| **Bot Detection (reCAPTCHA v2)** | Enforced with the "Always" setting across database login, user signup, and password recovery endpoints. | Automated Bot Signups & Directory Flooding. |
| **NIST Password Complexity** | Minimum 8 characters requiring 3 of 4 types; sequential and identical repeats blocked. | Dictionary Attacks & Weak Credentials |
| **Anti-Enumeration Responses** | Uniform generic failure prompts that never disclose whether an account exists. | Account Harvesting & User Enumeration. |
| **Strict URL Whitelisting** | Whitelist limited strictly to `localhost:3000-3003` across callbacks, origins, and logout paths | Open Redirectors & Token Exfiltration. |
| **Authenticated Mail Relay** | System mail dispatched via private SMTP relay (`smtp.gmail.com:465`, implicit SSL/TLS). | Insecure Password Reset Exploitation. |

---

## Threat Modeling & STRIDE Matrix

| STRIDE Category | Threat Scenario | Countermeasure & Architectural Control |
| :--- | :--- | :--- |
| **Spoofing** | Adversary attempts account takeover via stolen or leaked credentials. | Mandatory WebAuthn/TOTP MFA renders harvested passwords insufficient; breached password screening active. |
| **Tampering** | Adversary alters JWT claims to escalate privileges or bypass restrictions. | Asymmetric RS256 signature verification with pinned algorithm; modifications break cryptographic validity. |
| **Repudiation** | User denies performing sensitive actions or causing account lockouts[cite: 1]. | Immutable audit logs capture timestamp, user ID, IP address, and outcome without logging sensitive secrets. |
| **Information Disclosure** | Credentials intercepted in transit or usernames harvested via verbose errors. | End-to-end TLS 1.2+ encryption, secret-less SPA design, and uniform generic failure prompts. |
| **Denial of Service** | Bot scripts flood endpoints to lock student accounts or exhaust quotas. | Enforced reCAPTCHA v2, per-IP rate limiting, and verified self-service mailbox recovery workflows. |
| **Elevation of Privilege** | Misconfigured portal connects to an unauthorized tenant, bypassing controls. | Single-tenant policy enforcement; local token verification of issuer, audience, and role claims. |

---

## System Workflows

### 1. Authentication Workflow
```
[ Open Any Portal ] ──> [ Redirect /authorize ] ──> [ Submit Credentials ]
                        (PKCE Challenge, State)     (Hosted Universal Login)
                                                               │
                                                               ▼
                                                    [ Attack Protection ]
                                                    - Lockout check (10 fails)
                                                    - reCAPTCHA v2 & IP rate limit
                                                               │
                                   ┌───────────────────────────┴───────────────────────────┐
                                   │ Valid                                                 │ 10 Failures
                                   ▼                                                       ▼
                        [ Challenge MFA Factor ]                                  [ Lock Account ]
                        - Hardware Key or TOTP                                    - Security email to owner
                                   │                                              - Audit event logged
                                   ▼
                        [ Issue RS256 Tokens ]
                        - ID Token TTL: 900s
                                   │
                                   ▼
                        [ Session Established ]
                        - Shared cookie active across all 4 portals (SSO)
```

### 2. Single Logout (SLO) Workflow
```
[ Any Authenticated Portal ] ──> [ User Clicks "Log Out" ]
                                             │
                                             ▼
                          [ Destroy Shared Session at IdP ]
                                             │
                                             ▼
                        [ All 4 Portals Unauthenticated ]
                        (Redirect to allow-listed logout URL)
```

---

## Auth0 Configuration Guide

To replicate the security configuration of this implementation, configure your Auth0 dashboard as follows:

### 1. Application Registration
* Create a **Single Page Web Application** in your Auth0 dashboard.
* **Allowed Callback URLs:**
  ```text
  http://localhost:3000, http://localhost:3001, http://localhost:3002, http://localhost:3003
  ```
* **Allowed Logout URLs:**
  ```text
  http://localhost:3000, http://localhost:3001, http://localhost:3002, http://localhost:3003
  ```
* **Allowed Web Origins (CORS):**
  ```text
  http://localhost:3000, http://localhost:3001, http://localhost:3002, http://localhost:3003
  ```
* **Token Algorithm:** Set strictly to `RS256`.

### 2. Multi-Factor Authentication
* Navigate to **Security > Multi-factor Auth**.
* Enable **One-time Password (TOTP)** and **WebAuthn with Device Biometrics**.
* Set MFA enforcement policy to **Always**.

### 3. Attack Protection
* Navigate to **Security > Attack Protection**:
  * **Breached Password Detection:** Toggle ON.
  * **Brute-force Protection:** Toggle ON; set maximum failed attempts to `10`.
  * **Suspicious IP Throttling:** Toggle ON; set limits to 5 failed logins, 3 signups, and 10 token requests.
  * **Bot Detection:** Select **reCAPTCHA v2** and set enforcement to **Always**.

---

## Installation & Deployment

### Prerequisites
* **Node.js:** `v18.x` or higher
* **npm:** `v9.x` or higher
* Configured Auth0 Application per the settings above

### 1. Clone & Configure
```bash
git clone [https://github.com/your-username/authguard-mfa-sso-gateway.git](https://github.com/your-username/authguard-mfa-sso-gateway.git)
cd authguard-mfa-sso-gateway
```

Create a `.env` file in the root directory:
```env
REACT_APP_AUTH0_DOMAIN=your-tenant-subdomain.auth0.com
REACT_APP_AUTH0_CLIENT_ID=your-shared-client-id
REACT_APP_AUTH0_AUDIENCE=[https://your-tenant-subdomain.auth0.com/api/v2/](https://your-tenant-subdomain.auth0.com/api/v2/)
```

### 2. Install & Run
```bash
# Install workspace root and portal dependencies
npm install

# Concurrently run all four applications
npm run start:all
```

The portals will launch concurrently:
* **Main Portal:** `http://localhost:3000`
* **Course Registration:** `http://localhost:3001`
* **Exams Portal:** `http://localhost:3002`
* **Events Portal:** `http://localhost:3003`

---

## Standards & Attribution

### Security Standards
* **NIST SP 800-63B:** Digital Identity Guidelines (Authentication & Lifecycle Management).
* **RFC 7636:** Proof Key for Code Exchange by OAuth Public Clients (PKCE).
* **RFC 7519:** JSON Web Token (JWT) Syntax and Validation.
* **OWASP ASVS v4.0.3:** Application Security Verification Standard (Authentication & Session Management).



## License
This repository is open-source software licensed under the [MIT License](LICENSE).
