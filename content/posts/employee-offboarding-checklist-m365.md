---
title: "The Offboarding Checklist That Prevents Your Ex-Employee From Walking Out With Your Data"
date: 2026-04-14
lastmod: 2026-04-14
summary: "Most companies disable the email account and call it done. Here's what a real offboarding process looks like — and all the access points most teams completely miss."
description: "A practical IT offboarding checklist for Bay Area startups and small businesses — covering Microsoft 365, Entra ID, Intune, SaaS apps, and data recovery before it's too late."
draft: false
image: "/images/employee-offboarding-checklist-m365.webp"
slug: "employee-offboarding-checklist-m365"
tags:
  - Microsoft 365
  - Entra ID
  - Intune
  - Offboarding
  - Identity Management
  - Cybersecurity
categories:
  - Microsoft 365 & Security
---

Someone just gave two weeks notice. Or didn't — they're gone today.

Either way, your IT environment has a clock ticking on it. Every hour that passes is another hour that person still has access to email, files, code repositories, customer data, and a dozen SaaS tools you may not even remember they were using.

Most companies disable the email account and call it done. That's not offboarding — that's the first step of a 20-step process that most small businesses skip entirely.

Here's what a complete offboarding actually looks like.

<!--more-->

## Why This Matters More Than You Think

The threat isn't always malicious. Sometimes a departing employee forwards their work emails to a personal account "just to finish a project." Sometimes they stay logged into a shared tool for months because nobody reset the credentials. Sometimes they genuinely don't realize they still have access.

And sometimes it is intentional. Salespeople export the CRM before they leave. Developers clone repositories. Executives forward financials to a personal drive.

According to the Ponemon Institute, insider threats — whether negligent or malicious — account for a significant portion of data breaches at small and mid-sized businesses. The common thread: inadequate offboarding.

The other risk is purely operational. A departed employee's account left active is an orphaned credential — a target for credential stuffing, phishing, and account takeover. Attackers regularly scan for accounts with no recent activity and known email patterns.

Disabling access isn't just about trust. It's basic hygiene.

---

## The Complete Offboarding Checklist

### Step 1 — Entra ID (Azure AD): Disable the Account Immediately

This is your identity control plane. Everything else flows from here.

```powershell
# Disable the account in Entra ID immediately
# Connect-MgGraph -Scopes "User.ReadWrite.All"

$user = Get-MgUser -Filter "userPrincipalName eq 'jane.doe@yourcompany.com'"
Update-MgUser -UserId $user.Id -AccountEnabled $false
```

Disabling the account in Entra ID cascades to M365 services — Exchange Online, SharePoint, Teams, OneDrive — because they all authenticate through the same identity. One action, broad effect.

**Don't delete the account yet.** You want the account disabled, not gone. Deletion removes the mailbox and OneDrive data after a retention period. Give yourself time to recover what you need first.

**Also do immediately:**
- Revoke all active sessions: in the Entra ID portal, go to the user → Sessions → Revoke sign-in sessions. This kills any active browser sessions, app tokens, and refresh tokens.
- Remove from all admin roles if applicable

---

### Step 2 — Revoke All Active Sessions and Tokens

Disabling the account stops new logins. Revoking sessions kills the existing ones.

This matters because OAuth tokens and refresh tokens can remain valid even after an account is disabled — depending on the app and token lifetime configuration. Some tokens last 90 days by default.

```powershell
# Revoke all refresh tokens for the user
Revoke-MgUserSignInSession -UserId $user.Id
```

In the Entra ID portal: Users → select user → Revoke sessions.

For high-risk departures, also check **Conditional Access** sign-in logs to see what apps the user authenticated to recently. That list tells you what else needs attention.

---

### Step 3 — Exchange Online: Mailbox Access and Forwarding

The email account is where most data leakage happens — before and after termination.

**Check for forwarding rules first.** A common tactic is to set up an inbox forwarding rule days or weeks before leaving.

```powershell
# Connect to Exchange Online
# Connect-ExchangeOnline

# Check for forwarding rules
Get-InboxRule -Mailbox "jane.doe@yourcompany.com" | 
  Select Name, ForwardTo, ForwardAsAttachmentTo, RedirectTo

# Check mailbox-level forwarding
Get-Mailbox "jane.doe@yourcompany.com" | 
  Select ForwardingAddress, ForwardingSMTPAddress, DeliverToMailboxAndForward
```

If you find forwarding rules to external addresses, document them, then remove them. This is also worth flagging to your legal or HR team if the departure is contentious.

**Grant manager access to the mailbox** before you convert it:

```powershell
# Grant the manager full access
Add-MailboxPermission -Identity "jane.doe@yourcompany.com" `
  -User "manager@yourcompany.com" `
  -AccessRights FullAccess `
  -AutoMapping $true
```

**Convert to a shared mailbox** once you've confirmed the manager has what they need. Shared mailboxes don't require a license:

```powershell
Set-Mailbox -Identity "jane.doe@yourcompany.com" -Type Shared
```

Set an auto-reply if needed, then remove the license. The mailbox and its history stays accessible to the manager indefinitely.

---

### Step 4 — OneDrive: Recover the Files

OneDrive is where work-in-progress lives — often undocumented, often critical, often gone if you delete the account too quickly.

By default, M365 retains a deleted user's OneDrive for 30 days (configurable up to 180 days in SharePoint admin settings). Don't wait.

In the SharePoint Admin Center: Active Sites → find the user's OneDrive URL → give a manager or IT admin access.

Or via PowerShell:

```powershell
# Grant secondary owner access to OneDrive
Set-SPOUser -Site "https://yourcompany-my.sharepoint.com/personal/jane_doe_yourcompany_com" `
  -LoginName "manager@yourcompany.com" `
  -IsSiteCollectionAdmin $true
```

Review the files, download anything critical to a shared location, and then let the account deletion process handle the rest on schedule.

---

### Step 5 — Intune: Wipe or Retire the Device

If the employee used a company-managed device, it needs to be wiped before they return it — or remotely wiped if they're keeping it.

**Retire** removes company data and policies but leaves personal data intact. Use this for BYOD.

**Wipe** factory resets the device completely. Use this for corporate-owned devices.

In the Intune portal: Devices → find the device → Retire or Wipe.

Or via PowerShell/Graph API for bulk actions.

**Don't skip this.** A company laptop returned without a wipe still has local copies of files, cached credentials, browser sessions, and potentially VPN certificates. Even a "nice" departure doesn't make this safe to skip.

Also: **remove the device from Entra ID** (formerly Azure AD Join) so it no longer appears as a trusted device in Conditional Access evaluations.

---

### Step 6 — Multi-Factor Authentication: Remove Their Methods

When you disable the account, MFA becomes moot for that account. But if the employee was registered as an MFA method on a shared account or admin account — that's a different problem.

Check:
- Were they an emergency access account holder?
- Were they a backup MFA contact on any shared admin mailbox?
- Did they have any application passwords configured (legacy auth)?

```powershell
# List authentication methods for the user
Get-MgUserAuthenticationMethod -UserId $user.Id
```

Remove any methods tied to their personal phone number or authenticator app before account deletion.

---

### Step 7 — SaaS Applications: The Long Tail

This is where most small businesses have the biggest gaps. Your M365 tenant is locked down — but what about everything else?

Work through this list and check each one:

- **GitHub / GitLab** — remove from org, revoke personal access tokens
- **Slack** — deactivate account, transfer any private channel ownership
- **Salesforce / HubSpot** — deactivate user, reassign records and open tasks
- **Zoom** — deactivate account
- **AWS / Azure / GCP** — remove IAM user, revoke API keys, rotate any shared secrets they knew
- **1Password / Bitwarden** — remove from vault, rotate any shared credentials they had access to
- **Notion / Confluence / Jira** — deactivate, transfer page/project ownership
- **Google Workspace** (if used alongside M365) — disable account, recover Drive files
- **Figma, Miro, Loom, Linear** — any tool with company data
- **Domain registrars and DNS providers** — if they had access, rotate credentials
- **Billing accounts** — check if their card or personal account was attached to any service

The honest answer for most startups: you don't have a complete list. That's the problem. This is exactly why a SaaS inventory and access register matters — built before you need it, not during an offboarding at 5pm on a Friday.

---

### Step 8 — Shared Credentials and Secrets Rotation

Any shared password they knew needs to be rotated. Full stop.

This includes:
- Wi-Fi passwords (if they worked onsite)
- Shared service accounts
- API keys they may have used
- VPN pre-shared keys
- Any "team" accounts with shared login

If you're using a password manager like 1Password Teams or Bitwarden for Business, check the vault access logs to see what they accessed in the last 30–90 days. That list drives your rotation priority.

---

### Step 9 — Audit Log Review

Before you close the ticket, pull the audit logs.

In the Microsoft Purview compliance portal (formerly Security & Compliance Center), you can search audit logs for the user's activity over the past 90 days — file downloads, email forwarding changes, SharePoint access, external sharing events.

This isn't about paranoia. It's about knowing what happened so you can respond appropriately if something surfaces later.

```
Purview → Audit → New Search → filter by user, date range 30-90 days back
```

Look specifically for:
- Large file download volumes in the last 2–4 weeks
- External sharing events
- Email forwarding rule creation or modification
- Admin role changes they may have made

Document what you find. If the departure is contentious, this log is discoverable.

---

### Step 10 — Schedule Account Deletion

Don't delete the account immediately. Standard practice is 30 days minimum — enough time to recover files, handle any stragglers, and field the inevitable "can you find that email she sent last March" request.

Set a calendar reminder. When the 30 days are up, delete the account in Entra ID, which will trigger OneDrive deletion (with the retention period you've set), remove the mailbox if it's not already shared, and clean up the licensing.

---

## The Offboarding Runbook Template

Paste this into Notion, Confluence, or wherever your team documents processes:

```
Employee Offboarding — IT Checklist

Triggered by: [HR notification / manager request]
Employee: 
Last day:
IT owner:

[ ] Entra ID account disabled
[ ] Active sessions revoked (Revoke-MgUserSignInSession)
[ ] Forwarding rules checked and removed
[ ] Manager granted mailbox access
[ ] Mailbox converted to shared / license removed
[ ] OneDrive access granted to manager
[ ] Intune device retired or wiped
[ ] Device removed from Entra ID
[ ] MFA methods reviewed
[ ] SaaS application access removed (see list)
[ ] Shared credentials rotated
[ ] Audit log pulled and reviewed
[ ] Account deletion scheduled: [date]

Notes:
```

---

## What Happens When You Skip This

The most common outcomes we see when offboarding is done poorly:

**The lingering account.** A former employee's account stays active for weeks because IT wasn't notified until after they left. By then they've downloaded the customer list, the pricing model, and forwarded two years of emails to a Gmail account.

**The forgotten SaaS access.** Six months after someone leaves, a security audit reveals they still have admin access to the production AWS account. Their personal laptop — which left with them — was the only device with those credentials saved.

**The shared password problem.** The departed employee knew the password to the shared billing account. Three months later, someone notices unexpected charges. Nobody can figure out when or how the account was compromised.

None of these require a sophisticated attacker. They just require a gap in the process.

---

## How LineSight Digital Can Help

If your offboarding process is currently "disable the email and hope for the best," you're not alone — and it's fixable.

LineSight Digital helps Bay Area startups and small businesses build and run IT offboarding processes that actually work:

- **Offboarding runbook design** — a documented, repeatable process tailored to your M365 and SaaS environment
- **Entra ID and Exchange Online administration** — we handle the technical execution so HR doesn't have to figure out PowerShell
- **SaaS access inventory** — we build and maintain a register of who has access to what, so offboarding isn't a scavenger hunt
- **Audit log review** — post-departure review for high-risk or contentious departures
- **Managed IT** — ongoing administration that includes offboarding as a standard service

**[Schedule a free IT assessment](https://linesightdigital.com/#contact)** — we'll review your current offboarding process and identity posture, and tell you where the gaps are.

---

### Related Posts

- [Microsoft 365 Security Baselines for Small Businesses — What Actually Matters](/posts/microsoft-365-security-baselines-small-business/)
- [5 Signs Your Bay Area Startup Needs a Fractional IT Director](/posts/5-signs-startup-needs-fractional-it-director/)
- [Why Your Startup's Emails Are Landing in Spam — And How to Fix It with SPF, DKIM, and DMARC](/posts/email-authentication-audit/)

---

*Suresh Chand is the Founder & Managing Consultant of [LineSight Digital](https://linesightdigital.com), an IT managed services and consulting firm serving startups and small businesses in the San Jose / Silicon Valley / Bay Area.*
