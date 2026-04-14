---
title: "Your Employees Are Working From Coffee Shops. Here's Why That Should Terrify You."
date: 2026-04-14
lastmod: 2026-04-14
summary: "MFA alone isn't enough anymore. If you're not controlling where, when, and from what device your users can access company data, you're leaving the door wide open — regardless of how strong their passwords are."
description: "A practical guide to Microsoft Conditional Access for Bay Area startups and small businesses — how to move beyond MFA checkboxes to real zero-trust access control."
draft: false
image: "/images/conditional-access-zero-trust-smb.webp"
slug: "conditional-access-zero-trust-smb"
tags:
  - Conditional Access
  - Microsoft 365
  - Entra ID
  - Zero Trust
  - MFA
  - Cybersecurity
categories:
  - Microsoft 365 & Security
---

Your sales rep is logged into Salesforce from a coffee shop in the Mission. Your developer is accessing the code repository from a hotel in Austin. Your CFO just approved a wire transfer from their personal iPad at home.

All of them passed MFA. All of them are in.

Here's the problem: so would an attacker with a stolen session token, a compromised device, or a credential bought on the dark web — because MFA alone doesn't ask *where* you are, *what device* you're on, or *whether that device is safe*.

Conditional Access is what fixes that.

<!--more-->

## Why MFA Alone Isn't Enough

Multi-factor authentication is essential. It should be the first thing you configure in any M365 tenant, and we've covered that in detail in our [M365 Security Baselines post](/posts/microsoft-365-security-baselines-small-business/). But MFA has a fundamental limitation: it only verifies identity at the moment of login.

Once a user is authenticated, the session token is issued. That token can last hours or days. And tokens can be stolen.

**Token theft attacks** — specifically adversary-in-the-middle (AiTM) phishing — are now one of the most common techniques used against M365 environments. The attacker sits between the user and Microsoft's login page, captures the session token after MFA is completed, and uses it to access the account directly. No password needed. MFA bypassed entirely.

In 2023, Microsoft's own threat intelligence reported that AiTM phishing kits were responsible for compromising hundreds of thousands of accounts — the majority of which had MFA enabled.

This isn't a reason to skip MFA. It's a reason to add Conditional Access on top of it.

---

## What Conditional Access Actually Does

Conditional Access is Microsoft's policy engine for access control. It evaluates a set of signals every time a user tries to access a resource, and then decides what to do — allow, block, or require additional verification.

The signals it evaluates include:

- **Who** is signing in (user, group, role)
- **What** they're trying to access (specific app, all apps, admin portal)
- **Where** they're signing in from (country, IP address, named location)
- **What device** they're using (compliant, domain-joined, unmanaged)
- **What risk level** is associated with the sign-in or user (from Entra ID Identity Protection)

Based on those signals, it enforces a grant or block condition:

- Grant access (with or without MFA)
- Require MFA
- Require compliant device
- Require password change
- Block access entirely

The power is in combining signals. "Require MFA" is a Conditional Access policy. But so is "Block access from any country except the United States, unless the device is Intune-compliant, in which case allow with MFA." That's where it gets genuinely useful.

---

## The Licensing Reality

Conditional Access requires **Microsoft Entra ID P1** or above. This is included in:

- Microsoft 365 Business Premium
- Microsoft 365 E3 / E5
- Entra ID P1 / P2 standalone

If you're on Microsoft 365 Business Basic or Business Standard, you don't have Conditional Access. You have Security Defaults — which is better than nothing but not configurable.

For most growing Bay Area startups, Business Premium is the right license tier. The jump from Business Standard to Business Premium adds Conditional Access, Intune device management, and Defender for Business — the three things that actually matter for security. At roughly $4-6 more per user per month, it's the most defensible security spend in the M365 stack.

---

## The Policies That Actually Matter

### Policy 1 — Require MFA for All Users on All Apps

This is the baseline. Every user, every application, every sign-in.

```
Assignments:
  Users: All users
  Cloud apps: All cloud apps
  Conditions: (none)

Grant controls:
  Require multi-factor authentication
```

If you have Security Defaults enabled and nothing else, this is roughly what you have — but without the flexibility to exclude emergency access accounts or service accounts, which Security Defaults doesn't handle well.

**Create proper exclusions:** Always exclude at least one emergency access (break-glass) account from MFA policies. If your Conditional Access policies have a configuration error, you need a way back in that doesn't depend on MFA.

---

### Policy 2 — Block Legacy Authentication

Legacy authentication protocols — SMTP AUTH, POP3, IMAP, older Office clients — don't support modern authentication and can't complete MFA challenges. Attackers specifically target these protocols because they bypass MFA entirely.

```
Assignments:
  Users: All users
  Cloud apps: All cloud apps
  Conditions:
    Client apps: Exchange ActiveSync clients, Other clients

Grant controls:
  Block access
```

Before enabling this, run a sign-in log audit to identify any legitimate legacy auth traffic. Some line-of-business applications, printers, and shared mailbox automations still use basic auth. You'll need to migrate or exclude those before blocking.

```powershell
# Find legacy auth sign-ins in the last 30 days
Connect-MgGraph -Scopes "AuditLog.Read.All"

Get-MgAuditLogSignIn -Filter "clientAppUsed eq 'IMAP' or 
  clientAppUsed eq 'POP3' or 
  clientAppUsed eq 'SMTP'" -Top 100 |
  Select UserDisplayName, ClientAppUsed, AppDisplayName, CreatedDateTime
```

Any user or app showing up in that output needs to be remediated before you flip the block policy on.

---

### Policy 3 — Require Compliant Device for Sensitive Apps

This policy requires that devices accessing specific high-value applications are enrolled in Intune and marked compliant — meaning they meet your defined security baseline (encryption enabled, OS up to date, screen lock configured, etc.).

```
Assignments:
  Users: All users (or specific groups)
  Cloud apps: SharePoint Online, Exchange Online, specific LOB apps
  Conditions: (none)

Grant controls:
  Require device to be marked as compliant
  OR
  Require Hybrid Azure AD joined device
```

This is the policy that catches the "employee accessing company data from their personal, unmanaged laptop" scenario. An unmanaged device — even with a valid user credential and completed MFA — gets blocked.

For a BYOD environment, pair this with Intune's device enrollment for personal devices and an app protection policy, rather than requiring full device compliance.

---

### Policy 4 — Block High-Risk Sign-Ins

This requires Entra ID P2 (included in M365 E3/E5 or Business Premium with the right add-on). It uses Microsoft's Identity Protection ML models to evaluate each sign-in for risk signals — impossible travel, unfamiliar location, anonymous IP, password spray patterns.

```
Assignments:
  Users: All users
  Cloud apps: All cloud apps
  Conditions:
    Sign-in risk: High

Grant controls:
  Block access
```

For medium-risk sign-ins, require MFA rather than blocking — this catches the cases where the risk signal is elevated but not conclusive.

```
Conditions:
  Sign-in risk: Medium

Grant controls:
  Require multi-factor authentication
```

This is one of the most effective policies for catching account compromise in progress. If a stolen credential is used from an unusual location or via a proxy, Identity Protection flags it and Conditional Access acts on it — before the attacker gets in.

---

### Policy 5 — Protect the Admin Portal

Admin accounts are the highest-value targets in any M365 tenant. A compromised Global Admin can do essentially anything — delete data, disable MFA, create backdoor accounts, exfiltrate everything.

```
Assignments:
  Users: Directory roles → Global Administrator, 
         Exchange Administrator, SharePoint Administrator,
         Conditional Access Administrator (and any other admin roles)
  Cloud apps: Microsoft Admin Portals

Grant controls:
  Require multi-factor authentication
  Require compliant device (recommended)
```

Go further: require Privileged Identity Management (PIM) for admin role activation, so no one is permanently a Global Admin — they elevate when needed and the elevation is time-limited and audited.

---

### Policy 6 — Block Access from Unexpected Countries

If your team is entirely US-based and nobody travels internationally for work, there's no reason for sign-ins from Romania, Nigeria, or China. Block them.

```
Assignments:
  Users: All users
  Cloud apps: All cloud apps
  Conditions:
    Locations: Any location
    Exclude: United States (or whatever your approved countries are)

Grant controls:
  Block access
```

Create Named Locations for any countries your team legitimately operates in, and include those in the exclusion. For companies with international employees or contractors, adjust the approved country list accordingly.

This policy alone blocks a significant percentage of opportunistic credential stuffing attacks, which overwhelmingly originate from automated infrastructure in specific geographic regions.

---

## Reading the Sign-In Logs

Conditional Access is only as good as your visibility into what it's doing. The Entra ID sign-in logs are where you verify policies are working as intended — and catch what they're missing.

In the Entra admin center: Identity → Monitoring & Health → Sign-in logs

Filter by:
- **Conditional Access** column → look for "Failure" or "Not applied"
- **Client app** → identify legacy auth traffic
- **Location** → spot unexpected countries
- **Risk level** → see what Identity Protection is flagging

```powershell
# Pull failed Conditional Access evaluations
Get-MgAuditLogSignIn -Filter "conditionalAccessStatus eq 'failure'" -Top 50 |
  Select UserDisplayName, AppDisplayName, ConditionalAccessStatus, 
         IPAddress, Location, CreatedDateTime |
  Format-Table -AutoSize
```

Run this weekly when you're first rolling out policies. You'll catch misconfigured exclusions, legitimate users getting blocked, and policy gaps before they become incidents.

---

## The Rollout Order That Doesn't Break Things

Conditional Access mistakes can lock users out of their accounts entirely. Roll out in this order:

**Week 1:** Enable all policies in **Report-only mode**. This logs what would have happened without enforcing anything. Review the sign-in logs daily and identify anything that would have been incorrectly blocked.

**Week 2:** Enable Policy 1 (Require MFA for all users) in enforcement mode. Communicate to users beforehand. Handle any MFA registration issues.

**Week 3:** Enable Policy 2 (Block legacy auth) after confirming no legitimate legacy auth traffic remains.

**Week 4:** Enable Policy 6 (Block unexpected countries) and Policy 5 (Protect admin portal).

**Month 2:** Enable Policies 3 and 4 after Intune device enrollment is in place and Identity Protection is configured.

Report-only mode is your safety net. Use it every time you create or modify a policy before flipping it to enforcement.

---

## What a Conditional Access Audit Looks Like

When we inherit an M365 tenant, here's what we check on day one:

- Are any Conditional Access policies enabled? (Many tenants: none)
- Is Security Defaults still on? (Can't coexist with custom CA policies)
- Are legacy auth protocols blocked?
- Are admin accounts protected with MFA and device compliance?
- Are there emergency access accounts excluded from MFA policies?
- Are sign-in risk policies configured?
- Are Named Locations defined for the company's offices and approved countries?
- Are sign-in logs being reviewed regularly?

Most tenants have zero of these in place. Some have MFA enabled via Security Defaults and nothing else. A few have partial Conditional Access configurations with gaps that an attacker could walk through.

The gap between "we have MFA" and "we have proper Conditional Access" is the gap between a basic lock and a proper security system.

---

## How LineSight Digital Can Help

Conditional Access configuration requires getting the policy logic right, testing it without locking users out, and maintaining it as your team and tools evolve. It's one of the highest-leverage things you can do for M365 security — and one of the easiest to misconfigure.

LineSight Digital helps Bay Area startups and small businesses implement and maintain Conditional Access as part of a complete M365 security posture:

- **Conditional Access audit** — review existing policies, identify gaps, and document what's protecting you and what isn't
- **Policy design and implementation** — all six policies above, configured for your specific environment and user base
- **Report-only mode review** — we analyze sign-in logs before enforcement to ensure no legitimate users get blocked
- **Intune integration** — device compliance policies that Conditional Access can actually enforce against
- **Ongoing monitoring** — regular sign-in log review and policy tuning as your environment changes
- **Managed IT** — full M365 administration including identity, security, and Conditional Access as a standard service

**[Schedule a free IT assessment](https://linesightdigital.com/#contact)** — we'll review your current Conditional Access configuration and tell you exactly where the gaps are.

---

### Related Posts

- [Microsoft 365 Security Baselines for Small Businesses — What Actually Matters](/posts/microsoft-365-security-baselines-small-business/)
- [The Offboarding Checklist That Prevents Your Ex-Employee From Walking Out With Your Data](/posts/employee-offboarding-checklist-m365/)
- [Why Your Startup's Emails Are Landing in Spam — And How to Fix It with SPF, DKIM, and DMARC](/posts/email-authentication-audit/)

---

*Suresh Chand is the Founder & Managing Consultant of [LineSight Digital](https://linesightdigital.com), an IT managed services and consulting firm serving startups and small businesses in the San Jose / Silicon Valley / Bay Area.*
