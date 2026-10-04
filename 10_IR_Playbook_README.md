# 📘 IR Playbook — Phishing Credential Theft (NIST Framework)
### Project 10 — SOC Analyst Portfolio | Incident Response | NIST SP 800-61 | Tabletop Exercise

`Role: SOC Analyst L1` `Framework: NIST SP 800-61` `Scenario: Phishing Credential Theft` `Severity: High` `Status: Completed`

---

## 📌 What Is This Project? (Simple Explanation — No Tech Background Needed)

Imagine a fire drill at a school. There's a written plan for exactly what to do if there's a fire — who calls the fire brigade, who leads people out, which door to use. Nobody waits until a real fire starts to figure this out.

This project is the SAME IDEA, but for a **fake email that steals someone's password** instead of a fire. I wrote the step-by-step plan (called a "playbook"), then **tested the plan** by walking through a pretend attack scenario out loud — like a practice fire drill — to find out where the plan had holes, and then fixed those holes.

---

## 🎯 Why This Matters

```
Without a playbook:
Attack happens → team panics → argues about 
"who does what" → important steps missed → 
more damage, slower recovery

With a playbook:
Attack happens → everyone already knows their 
role → steps followed fast and in order → 
damage contained quickly
```

Every real company — and every security audit — requires written playbooks like this one. This project proves I can build one properly, using **NIST**, the official framework followed by companies and governments worldwide.

---

## 🧠 Key Words Explained Simply

```
NIST          → A US government body that publishes 
                free official security rulebooks. 
                The one I used here is called 
                NIST SP 800-61.

Phishing      → A fake email pretending to be from 
                someone trustworthy (like Microsoft), 
                trying to trick you into giving up 
                your password.

IOC           → "Indicator of Compromise" = evidence 
                that an attack happened (a bad link, 
                a bad sender address, etc.)

Tabletop      → A group sits down and talks through 
Exercise        a fake attack step-by-step, out loud, 
                using the playbook as a guide — no 
                real computers touched. It's a 
                practice drill.

SOC           → Security Operations Center — the 
                team that watches for and responds 
                to attacks, 24/7.
```

---

# PHASE 1 — ATTACK (The Scenario)

## 🎭 Incident Scenario

```
09:00 AM — An employee in the Finance team 
receives an email that looks exactly like a 
real Microsoft 365 security alert:

  From: security-noreply@micros0ft-verify[.]com
  Subject: Unusual sign-in activity detected — 
           verify your account now

  ⚠️ Note: All domains/URLs in this document are 
  defanged (brackets around dots) per standard 
  SOC reporting practice, to prevent accidental 
  clicks or security scanner triggers.

  "We noticed a sign-in from a new device. 
   Click below to confirm it was you."
   [Verify My Account]

09:03 AM — The employee clicks the link. It 
opens a fake Microsoft 365 login page that 
looks identical to the real one.

09:04 AM — The employee types in their real 
username and password. The fake page captures 
it and silently redirects them to the real 
Microsoft login page (so they don't suspect 
anything).

09:12 AM — Microsoft Defender fires an alert: 
"Sign-in from unfamiliar location" on the 
employee's account, from an IP address in a 
different country, 6 minutes after the 
phishing click.
```

**📸 Reference image placeholder:** `images/01-phishing-email-sample.png` — *screenshot of a sample phishing email template for reference*

---

# PHASE 2 — DOCUMENT (Investigation Timeline)

## 🕐 Full Timeline

| Time | Event | Source |
|------|-------|--------|
| 09:00 | Phishing email delivered to employee inbox | Email Security Gateway |
| 09:03 | Employee clicks malicious link | Proxy/Web logs |
| 09:04 | Employee enters credentials on fake page | (Inferred from timeline + user interview) |
| 09:12 | Alert: Sign-in from unfamiliar location | Microsoft Defender |
| 09:15 | SOC Analyst (me) picks up the alert | Splunk / Ticketing System |
| 09:20 | Investigation begins | — |
| 09:35 | Containment actions started | — |
| 10:10 | Incident contained | — |

---

## 🔎 Investigation Steps — What I Checked

### Step 1 — Analyze the Email Header
```
What I looked for:
→ Real sender domain vs. displayed sender name
→ SPF / DKIM / DMARC authentication results
→ Return-Path mismatch

Finding:
→ Sender domain: micros0ft-verify[.]com (note the 
  ZERO instead of the letter O — a classic 
  lookalike domain trick)
→ SPF: FAIL
→ DKIM: FAIL
→ DMARC: FAIL
```

### Step 2 — Investigate the Malicious URL
```
Tool used: URLScan.io

What I did:
→ Submitted the URL from the phishing email 
  in a safe, isolated sandbox environment
→ Reviewed the screenshot URLScan captured 
  of the fake login page
→ Checked the redirect chain and hosting IP

Finding:
→ Page was an exact visual clone of the real 
  Microsoft 365 login page
→ Hosted on a newly registered domain (less 
  than 7 days old — a major red flag)
```

### Step 3 — Check Reputation
```
Tool used: VirusTotal

What I did:
→ Submitted the malicious URL and sender domain
→ Checked how many security vendors flagged it

Finding:
→ 34 out of 70 security vendors flagged the 
  URL as phishing/malicious
```

### Step 4 — Confirm the Account Compromise
```
Tool used: Microsoft Defender / Sign-in logs

What I looked for:
→ Sign-in location, device, and IP address 
  compared to the employee's normal pattern

Finding:
→ Sign-in came from an IP address in a 
  different country, on a device never seen 
  before for this user — confirms the 
  credentials were actually used by the 
  attacker, not just phished
```

**📸 Screenshot placeholder:** `images/02-urlscan-result.png` — *URLScan.io report showing the fake login page*
**📸 Screenshot placeholder:** `images/03-virustotal-result.png` — *VirusTotal detection results for the malicious URL*

---

# PHASE 3 — ASSESS (Ticket Notes & Analyst Actions)

## 🎫 SOC Ticket — Sample Artifact

```
================================================================================
SOC INCIDENT TICKET
================================================================================
TICKET ID:        INC-2026-0417
PRIORITY:         P1 - HIGH
CATEGORY:         Phishing / Credential Compromise
ASSIGNED TO:       Riyaz Shaik (SOC Analyst L1)
STATUS:            RESOLVED

SUMMARY:
Employee in Finance department clicked a phishing 
link impersonating Microsoft 365 and entered valid 
credentials. Account showed sign-in from an 
unfamiliar location 8 minutes after the click, 
confirming credential compromise.

IOCs IDENTIFIED:
- Sender domain: micros0ft-verify.com
- Malicious URL: hxxps://micros0ft-verify[.]com/login
- Attacker sign-in IP: 185.220.XX.XX (anonymized)

ANALYST ACTIONS TAKEN:
1. Verified phishing email via header analysis 
   (SPF/DKIM/DMARC all FAIL)
2. Investigated URL safely using URLScan.io sandbox
3. Checked IOC reputation via VirusTotal (34/70 
   vendors flagged malicious)
4. Confirmed account compromise via Defender sign-in logs
5. Escalated to containment actions (below)

STATUS: Contained and resolved within 1 hour of detection.
================================================================================
```

---

## 🧭 Why I Classified This as High Severity

```
Severity Decision Logic:

Was a real password entered?           YES
Was the account actually accessed      YES
  by the attacker (not just phished)?
Could this account access sensitive    YES (Finance)
  company/financial data?
Could this spread further (shared      Possible — 
  mailbox, forwarded emails)?          needs checking

→ All factors point to HIGH severity. 
  If credentials were phished but NEVER used, 
  this could have been Medium. The confirmed 
  sign-in is what pushes it to High.
```

---

# PHASE 4 — MITIGATE (Containment, Eradication, Recovery)

## 🧯 NIST 4-Phase Response Actions

### 1. Containment (Stop the bleeding — first priority)
```
✅ Disable the compromised account immediately 
   in Active Directory / Microsoft 365 Admin Center
✅ Revoke all active sessions and tokens for the 
   account (forces re-login everywhere)
✅ Block the malicious URL and sender domain at 
   the email gateway and web proxy
✅ Notify the employee directly — do not let them 
   find out through rumor
```

### 2. Eradication (Remove the threat)
```
✅ Search mailbox for any auto-forwarding rules 
   the attacker may have set up (common attacker 
   trick to keep reading email after being locked out)
✅ Check for any files shared or downloaded during 
   the compromised session window (09:12–09:20)
✅ Search organization-wide for the same phishing 
   email sent to other employees — block and 
   remove from all inboxes
```

### 3. Recovery (Back to normal)
```
✅ Reset the employee's password
✅ Re-enable Multi-Factor Authentication (MFA) and 
   confirm it's enforced going forward
✅ Re-enable the account only after confirming no 
   persistence (forwarding rules, app permissions) remains
✅ Monitor the account closely for 72 hours
```

### 4. Lessons Learned (Post-Incident Review)
```
✅ Held a review: how did this email bypass the 
   email security gateway filter?
✅ Updated email filter rules to catch lookalike 
   domains (e.g., "0" instead of "o")
✅ Scheduled a company-wide phishing awareness 
   reminder
✅ Updated this playbook based on findings (see 
   Tabletop Exercise section below)
```

---

## 🎭 Tabletop Exercise — Testing the Playbook

```
Scenario read aloud to the "team":

"It's 9:12 AM. Microsoft Defender fires an alert 
for unfamiliar sign-in location on a Finance 
employee's account. You are the on-shift SOC L1 
Analyst. Walk through exactly what you do, step 
by step, using the playbook."
```

### What I Walked Through
```
09:15 → Picked up the alert — confirmed severity 
        using the decision logic above

09:18 → Realized the playbook did NOT clearly say 
        WHO has authority to disable a user account 
        — does L1 need manager approval first? 
        This was AMBIGUOUS.

09:25 → Simulated containment: disable account, 
        revoke sessions, block URL at gateway

09:32 → Discovered the playbook had no clear 
        answer for "what if the attacker set up 
        an auto-forwarding rule and is still 
        reading emails even after the password 
        is reset?"
```

## 🔧 What Broke — And How I Fixed It (Required by Task)

| Gap Found During Tabletop | Fix Added to Playbook |
|---|---|
| No clear owner for "disable account" decision | Added: "SOC L1 Analyst has standing authority to disable any account showing confirmed sign-in compromise within the first 15 minutes, without waiting for manager approval" |
| No check for mailbox auto-forwarding persistence | Added mandatory step: "Always check for forwarding rules and inbox delegation BEFORE marking an incident as closed" |
| No employee communication plan | Added: "Notify affected employee directly by phone within 30 minutes — never let them learn through rumor or a company-wide alert" |
| No guidance on checking for spread | Added: "Search organization-wide for identical phishing email subject/sender before closing — assume more than one employee was targeted" |

```
This is exactly why tabletop exercises matter — 
a playbook that looks complete on paper often has 
real gaps that only appear when you walk through 
a realistic scenario.
```

---

## 🎯 MITRE ATT&CK Mapping

| Stage | Technique | ID | Tactic |
|-------|-----------|-----|--------|
| Phishing email delivery | Phishing: Spearphishing Link | T1566.002 | Initial Access |
| Credential entry on fake page | Valid Accounts | T1078 | Initial Access / Persistence |
| Lookalike domain registration | Acquire Infrastructure: Domains | T1583.001 | Resource Development |
| Victim clicking the link | User Execution: Malicious Link | T1204.001 | Execution |

---

## ✅ Recommendations

```
1. Enforce MFA on all accounts — this alone would 
   have stopped the attacker from using the 
   stolen password
2. Update email gateway rules to catch lookalike 
   domains (character substitution like 0 for o)
3. Run phishing awareness training + simulated 
   phishing tests quarterly
4. Build automatic mailbox-forwarding-rule 
   detection as a standing SIEM alert
5. Run this tabletop exercise again every 6 months 
   with a different scenario
```

---

## 💼 Skills Demonstrated

```
✅ NIST SP 800-61 incident response framework
✅ Phishing email header analysis (SPF/DKIM/DMARC)
✅ URL investigation using URLScan.io sandbox
✅ Threat intelligence lookup using VirusTotal
✅ Account compromise confirmation via sign-in logs
✅ Incident ticket documentation
✅ Severity classification and risk reasoning
✅ Full 4-phase incident response execution
✅ Tabletop exercise facilitation and gap analysis
✅ Playbook writing and continuous improvement
✅ MITRE ATT&CK technique mapping
```

---

## 🗣️ 30-Second Interview Answer

> "I wrote a NIST SP 800-61 based incident response playbook for a phishing credential-theft scenario — a fake Microsoft 365 email leading to account compromise. I documented the full investigation using email header analysis, URLScan, and VirusTotal, built a sample SOC ticket, and executed all four NIST phases: containment, eradication, recovery, and lessons learned. I then ran a tabletop exercise to test the playbook, which exposed real gaps — like unclear authority to disable an account and no check for mailbox forwarding rules — and I fixed both in a revised version. I mapped the scenario to MITRE ATT&CK techniques including T1566.002 and T1078."

---

## 📝 Resume Project Description

```
IR Playbook — Phishing Credential Theft (NIST Framework)
Authored a NIST SP 800-61-based incident response 
playbook for a phishing credential-theft scenario 
involving a spoofed Microsoft 365 login page. 
Investigated using email header analysis (SPF/DKIM/
DMARC), URLScan.io, and VirusTotal, and documented a 
full incident timeline, SOC ticket, and severity 
justification. Executed all four NIST response phases — 
containment, eradication, recovery, and lessons learned — 
and validated the playbook through a tabletop exercise 
that identified and resolved 4 real process gaps. Mapped 
to MITRE ATT&CK (T1566.002, T1078, T1583.001, T1204.001).
```

---

## ✅ ATS-Friendly Resume Bullets

```
• Authored a NIST SP 800-61 incident response playbook 
  for a phishing credential-theft scenario
• Investigated phishing emails using header analysis 
  (SPF, DKIM, DMARC), URLScan.io, and VirusTotal
• Documented full incident timeline, SOC ticket, and 
  severity classification for a credential-compromise event
• Executed containment, eradication, recovery, and 
  lessons-learned phases per NIST framework
• Facilitated a tabletop exercise that identified and 
  resolved 4 process gaps in the incident response plan
• Mapped phishing attack techniques to MITRE ATT&CK 
  (T1566.002, T1078, T1583.001, T1204.001)
```

---

## ❓ Interview Q&A

**Q: How do you know the account was actually compromised, not just phished?**
> Phishing alone means the password was stolen. I confirmed compromise by checking Defender's sign-in logs — there was an actual successful sign-in from an unfamiliar country and device 8 minutes after the click. That confirmed the attacker used the credentials.

**Q: Why High severity and not Medium?**
> Because the credentials were confirmed to be actively used by the attacker, and the account belonged to Finance — sensitive financial data could be accessed. If it had only been a click with no confirmed sign-in, I'd rate it Medium.

**Q: What was the most important gap your tabletop exercise found?**
> The lack of a mailbox forwarding-rule check. Even after resetting a password, an attacker who set up an auto-forward rule can keep reading the victim's email indefinitely. That's an easy thing to miss if it's not explicitly in the playbook.

**Q: Why do tabletop exercises matter if you already have a written playbook?**
> A playbook can look complete on paper but fall apart under a real timeline. Walking through it out loud, step by step, exposes exactly where people would hesitate or disagree — which is much cheaper to find in a drill than during a real attack.

---

## 📁 Repository Structure

```
10.IR-Playbook-Phishing-Incident-Response/
├── README.md
├── playbook/
│   └── NIST_Phishing_IR_Playbook.docx
├── incident-report/
│   ├── INC-2026-0417_ticket.txt
│   └── timeline.md
├── tabletop-exercise/
│   └── gaps_and_fixes.md
└── images/
    ├── 01-phishing-email-sample.png
    ├── 02-urlscan-result.png
    └── 03-virustotal-result.png
```

---

## ⚠️ Disclaimer

> This playbook was built as a portfolio and learning exercise based on the publicly available NIST SP 800-61 framework. The scenario, IOCs, and ticket are simulated for educational purposes and do not represent any real company's actual incident.

---

*IR Playbook | SOC Analyst L1 Portfolio | Riyaz Shaik*
