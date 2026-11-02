# AD Helpdesk Troubleshooting Lab

Eight IT helpdesk tickets worked end to end in a home Active Directory lab and documented as troubleshooting logs. Each ticket is a PDF with the problem, the steps taken, the root cause, the fix, how it was verified, what I got wrong, and screenshots as evidence.

**Author:** Ahmed Elfatih, Computer Science (Cybersecurity) graduate based in the UAE, targeting IT helpdesk and entry-level networking roles. CCNA studies completed, exam pending.
**Contact:** `ahmedelfatih01` · `ahmedelfatih345@outlook.com`

> **What this is, and isn't.** This is a home lab, not a production environment. The users are fictional, there is one domain controller and one client, and Tickets after ticket 6 are scripted break/fix exercises. Everything shown was actually done in the lab and verified on screen. I wrote it to practise the helpdesk workflow and to show how I document my work.

---

## Tickets

| # | Ticket | Type | What it demonstrates | PDF |
|---|---|---|---|---|
| 1 | Password reset | Request | Identity verification, temporary password with forced change, confirming the change server-side, handling a rejected new password | [Ticket01](tickets/Ticket01-Password-Reset.pdf) |
| 2 | Account lockout | Incident | Triage by error message, confirming the lockout in AD, tracing the source with Security event 4740, unlocking | [Ticket02](tickets/Ticket02-Account-Lockout.pdf) |
| 3 | New hire onboarding | Request | Account creation in the correct OU, department group, directory attributes, positive and negative access tests | [Ticket03](tickets/Ticket03-New-Hire-Onboarding.pdf) |
| 4 | Access request | Request | Validating a data-access request, group-based access, why a sign-out is needed for the change to apply | [Ticket04](tickets/Ticket04-Access-Request.pdf) |
| 5 | Mapped drive (GPO) | Request / Incident | Group Policy Preference drive mapping, item-level targeting, a seven-step "my drive isn't showing" checklist | [Ticket05](tickets/Ticket05-Mapped-Drive.pdf) |
| 6 | Leaver | Request | Disable rather than delete, removing groups, recording the change, moving to a Disabled OU | [Ticket06](tickets/Ticket06-Leaver.pdf) |
| 7 | DNS fallback | Break/fix | A wrong DNS server hidden by NetBIOS/LLMNR fallback, testing with `-DnsOnly`, DNS suffixes, cache TTL | [Ticket07](tickets/Ticket07-DNS-Fallback.pdf) |
| 8 | Hosts file | Break/fix | A poisoned hosts file entry that only affects the FQDN, name resolution order, elevated edit, clearing the DNS cache | [Ticket08](tickets/Ticket08-Hosts-File.pdf) |

---

## Troubleshooting methodology

Every ticket follows the same sequence.

1. **Validate the request.** Confirm who is asking and that they are entitled to ask. A password reset or a data-access grant is a security decision, not just a click. Access requests also need the data owner's approval and an end date for temporary access.
2. **Identify the symptom precisely.** The exact error text matters. "My password isn't working" can mean a lockout, a wrong password, a disabled account or an expired password, and each has a different fix (Ticket 2 has the full triage table).
3. **Reproduce and check both sides.** Confirm what the user sees on the client, then confirm the state on the server. The error message alone is not proof (`Get-ADUser`, `whoami /groups`, `Get-DnsClientServerAddress`).
4. **Find the cause, not just the fix.** Unlocking an account without checking event 4740 can mean it locks again ten minutes later. Ticket 2 traces the failed attempts back to the originating machine before unlocking.
5. **Fix with least privilege.** Give exactly what the role needs. Use groups instead of per-user permissions, and never touch privileged groups.
6. **Verify, including negative tests.** Prove what the user can reach and what they cannot (Ticket 3 tests both the Sales and Finance shares). Confirm the fix did not open anything else.
7. **Document.** Each ticket uses the same template, so a colleague can pick it up cold.

**Break/fix exercises (Tickets 7 and 8).** A PowerShell script introduces a fault on the client VM. I work from a symptom-only ticket description, diagnose, fix and then write it up.

**Mistakes are recorded on purpose.** In both break/fix tickets I first assumed the fault had not landed because the share was still reachable. The "What I got wrong or assumed" section of each log explains why that was wrong. Ticket 7 shows NetBIOS/LLMNR fallback masking a broken DNS setting. Ticket 8 shows a hosts file entry matching only the exact name typed.

---

## Ticket template

Every PDF uses the same eight sections:

1. **Summary:** title, date, ticket ID, status
2. **Problem:** symptom, impact, when it started, recent changes
3. **Environment:** device and OS, network and server, software
4. **Troubleshooting Steps:** a table of what was tried and the result
5. **Root Cause:** what actually broke and why
6. **Fix:** what solved it, with the exact commands or settings
7. **Verification:** how the fix was confirmed
8. **Lessons Learned:** what to check first, what was wrong or assumed, how to prevent it
9. **Screenshots:** the evidence, as numbered figures at the end

Tickets 1 to 6 are written as walkthroughs of requests I carried out and also include background notes and a "if it still doesn't work" table. Tickets 7 and 8 are straight troubleshooting logs. Ticket IDs follow the pattern `TCK0N-ShortName`.

---

## Lab environment

| Component | Detail |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Domain controller | Windows Server 2022 with AD DS, DNS and File Services roles |
| Domain | `acme.local` |
| Client | Windows 10 Pro VM, domain-joined |
| Network | Domain-facing network `192.168.50.0/24` on one adapter, plus a separate adapter for internet access |
| Directory structure | `Acme` OU containing `Users` (Finance, HR, IT, Sales), `Groups`, `Computers` and `Disabled` |
| Security groups | `GRP-Sales`, `GRP-Finance`, `GRP-HR`, `GRP-IT`, `GRP-Helpdesk` |
| Users | Fictional staff across Sales, Finance and HR |
| Lockout policy | 5 failed attempts, 30-minute lockout and observation window |

## Tools and technologies

- **Active Directory Domain Services:** users, groups, OUs, account state, password policy, lockout policy
- **Active Directory Users and Computers (ADUC):** day-to-day account administration
- **Group Policy Management (GPMC):** Group Policy Preferences, drive maps, item-level targeting, linking and scope
- **PowerShell:** `ActiveDirectory` module (`Get-ADUser`, `Set-ADAccountPassword`, `Unlock-ADAccount`, `Disable-ADAccount`, `Add-ADGroupMember`, `New-ADUser`, `Move-ADObject`) and DNS client cmdlets (`Resolve-DnsName`, `Get-DnsClientServerAddress`, `Set-DnsClientServerAddress`, `Clear-DnsClientCache`)
- **Windows diagnostics:** `whoami /groups`, `gpupdate`, `gpresult`, Event Viewer and `Get-WinEvent` (Security events 4740 and 4728)
- **File services:** SMB shares and NTFS permissions, UNC paths and mapped drives
- **Name resolution:** DNS, hosts file, NetBIOS, LLMNR and mDNS fallback, DNS suffixes, TTL and client cache

---

## Concepts covered

- Authentication versus authorization, and why identity is verified before any account change
- The access token: why a group change only takes effect after the user signs out and back in
- Share permissions versus NTFS permissions, and why an explicit Deny wins
- OU placement (which policies apply) versus group membership (what can be accessed)
- Why accounts are disabled rather than deleted when someone leaves
- Name resolution order on Windows (cache, hosts file, DNS, then link-local fallbacks), and how a fallback can hide a real DNS fault
- The difference between a permission error and a name-resolution error, and what each points to

---

## Security handling in the lab

- Passwords are blacked out in screenshots, and Windows product IDs are redacted.
- Temporary passwords in the lab are simple for demonstration. Ticket 1 notes that in production they should be random, unique per ticket and passed over a secure channel.
- Support never sees the user's final password. The "must change at next logon" flag is verified in AD to show this.

---

## Repository layout

```
ad-helpdesk-troubleshooting-lab/
├── README.md
└── tickets/
    ├── Ticket01-Password-Reset.pdf
    ├── Ticket02-Account-Lockout.pdf
    ├── Ticket03-New-Hire-Onboarding.pdf
    ├── Ticket04-Access-Request.pdf
    ├── Ticket05-Mapped-Drive.pdf
    ├── Ticket06-Leaver.pdf
    ├── Ticket07-DNS-Fallback.pdf
    └── Ticket08-Hosts-File.pdf
```

## Limitations

- Single domain controller, so replication delays and per-DC lockout counters are described in the notes but were not observed.
- Windows 10 client only. No macOS, mobile or Microsoft 365 or Entra ID scenarios.
- No ticketing system is integrated into these logs. They are standalone documents.
- Building Group Policy is normally a sysadmin task. Ticket 5 builds the policy first so that the helpdesk troubleshooting checklist can be tested against something real.
