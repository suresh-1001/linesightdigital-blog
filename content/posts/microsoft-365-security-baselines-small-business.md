---
title: "Microsoft 365 Security Baselines for Small Businesses — What Actually Matters"
date: 2026-03-15
lastmod: 2026-03-15
summary: "Most small businesses have Microsoft 365 configured out of the box. Out of the box is not secure. Here's what to fix first."
description: "A practical guide to Microsoft 365 security baselines for Bay Area startups and small businesses — covering MFA, Conditional Access, Entra ID, and email authentication."
draft: false
image: "/images/m365-security-baselines.png"
slug: "microsoft-365-security-baselines-small-business"
tags:
  - Microsoft 365
  - Entra ID
  - MFA
  - Conditional Access
  - Cybersecurity
  - Email Security
categories:
  - Microsoft 365 & Security
---

Most small businesses that come to us have Microsoft 365 up and running. Email works. Teams works. SharePoint mostly works.

**What doesn't work: the security configuration.**

Out-of-the-box Microsoft 365 is not secure. The defaults are designed for broad compatibility, not protection. This post covers the baselines we implement for every client — and why they matter for startups and small teams.

<!--more-->

## Why M365 Security Configuration Gets Skipped

Small business owners set up M365 through a reseller, an IT generalist, or a step-by-step guide. The guide covers licensing, email setup, and maybe Teams. It rarely covers security hardening.

The result: a fully licensed tenant that is wide open to credential stuffing, phishing, and account takeover.

These aren't edge cases. They're the most common incidents we see in Bay Area small businesses.

## 1. Multi-Factor Authentication (MFA) — Non-Negotiable

The single highest-impact change you can make.

Without MFA, a stolen password = a compromised account. With MFA, a stolen password alone is useless.

**What to configure:**

- Enable MFA for every user — no exceptions, including the admin account
- Use Microsoft Authenticator (app-based), not SMS codes
- Disable legacy authentication protocols (SMTP Auth, POP, IMAP) that bypass MFA

If you only do one thing from this post, do this.

## 2. Conditional Access Policies

Conditional Access is how you enforce *when and where* users can authenticate. It's available on Microsoft 365 Business Premium and above.

**Key policies to implement:**

- Require MFA for all users on all apps
- Block legacy authentication
- Require compliant devices for access to sensitive data
- Block sign-ins from high-risk locations or anonymous IP addresses

Conditional Access turns MFA from a checkbox into an actual access control layer.

## 3. Entra ID (Azure AD) Hardening

Entra ID is the identity backbone of your M365 tenant. Misconfigured, it's your biggest attack surface.

**What we check and fix:**

- Disable user consent for third-party OAuth apps (prevents consent phishing)
- Enable Entra ID Identity Protection if licensed
- Review admin role assignments — most tenants have too many Global Admins
- Enable Privileged Identity Management (PIM) for just-in-time admin access
- Configure Self-Service Password Reset (SSPR) with proper authentication methods

## 4. Email Authentication: SPF, DKIM, DMARC

Email spoofing is trivially easy without these three DNS records. With them, it's nearly impossible.

- **SPF** — defines which servers are allowed to send email on your behalf
- **DKIM** — cryptographically signs outgoing mail
- **DMARC** — tells receiving servers what to do when SPF/DKIM checks fail (report, quarantine, or reject)

A DMARC policy of `p=none` means you're monitoring but not blocking. Move to `p=quarantine` or `p=reject` once you've validated your sending sources.

Not having these configured also hurts deliverability. Your legitimate email is more likely to land in spam.

## 5. Exchange Online Protection Configuration

Exchange Online Protection (EOP) is included with every M365 plan. Most tenants leave it at defaults.

**Settings worth reviewing:**

- Enable Safe Links and Safe Attachments (requires Defender for Office 365)
- Configure anti-phishing policies with impersonation protection
- Tune the spam confidence level (SCL) thresholds
- Enable mailbox auditing (it should be on by default, but verify)

## 6. Data Loss Prevention (DLP) Basics

If your team handles any sensitive data — client information, financial records, healthcare data — DLP policies help prevent accidental or intentional data leaks.

Basic DLP for small businesses:

- Block or alert on sharing files containing Social Security Numbers or credit card numbers externally
- Restrict sensitive SharePoint sites to specific groups
- Review OneDrive sharing defaults (disable "Anyone with the link" if you don't need it)

## What a Security Baseline Review Looks Like

When we onboard a new M365 client, we run through approximately 40 configuration checkpoints across:

- Identity and authentication (Entra ID, MFA, Conditional Access)
- Email security (EOP, Defender, SPF/DKIM/DMARC)
- Data and file sharing (SharePoint, OneDrive, Teams)
- Device management (Intune enrollment and compliance policies)
- Admin controls and audit logging

Most tenants have 15–25 items that need attention. A few have more.

## Final Thoughts

Microsoft 365 is excellent software. Its default security posture is not excellent for small businesses.

The good news: most of this is configurable, not a product limitation. It just requires someone who knows what to look for.

**[LineSight Digital](https://linesightdigital.com/#contact)** offers a free IT assessment that includes a review of your Microsoft 365 security configuration. We'll tell you where you stand — no obligation.

### Related Posts

- [5 Signs Your Startup Needs a Fractional IT Director](/posts/5-signs-startup-needs-fractional-it-director/)
- [Backup and Disaster Recovery for Bay Area Small Businesses](/posts/backup-disaster-recovery-small-business/)
