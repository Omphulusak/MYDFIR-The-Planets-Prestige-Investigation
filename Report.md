# Formal Investigation Report: The Planet’s Prestige

| | |
|---|---|
| **Platform** | Blue Team Labs |
| **Challenge** | The Planet’s Prestige |
| **Report type** | Email investigation (phishing / extortion) |
| **Evidence** | One `.eml` file and its attachment |
| **Classification** | Malicious: spoofed extortion email with a disguised archive attachment |
| **Full analysis** | [README.md](README.md) |

## 1. Executive Summary

On Tue, 26 Jan 2021 at 06:41:18 UTC (01:41:18 -0500 per the sender’s header; 22:41:18 -0800 on 25 Jan per Google’s receiving server), an Army Major on Earth received an email that appeared to come from `billjobs[@]microapple[.]com`. It demanded 1 Billion CoCanDs for the return of abducted citizens, including the Planetary President’s daughter.

The message **failed SPF**: the sending IP `93.99.104.210`, belonging to the free fake-mailer service **emkei[.]cz**, is not a permitted sender for `microapple[.]com`. The real reply address was `negeja3921[@]pashter[.]com`. The “PDF” attachment was a ZIP archive containing three files. One of them, `Money.xlsx`, states that the ransom was a decoy and that the real intent is **war with the CoCanDians**.

**Conclusion:** the email is malicious. The ransom demand should not be paid.

## 2. Key Findings

1. **Spoofed sender, SPF fail.** The `From` and `Return-Path` are `billjobs[@]microapple[.]com`, but Google recorded `spf=fail` because `93.99.104.210` is not designated to send for that domain. The `Reply-To` is `negeja3921[@]pashter[.]com`.
2. **Fake-mailer origin.** The `Received` header shows delivery from `emkei[.]cz` (`93.99.104.210`, Prague) to `mx.google.com`. The message was created on the mailer’s own server (Postfix, queue id `1993E221F8`).
3. **Encoded body.** The message body was Base64-encoded and contained the ransom demand and a puzzle instruction.
4. **Disguised attachment.** `PuzzleToCoCanDa.pdf` carries a ZIP signature (`50 4B 03 04`) rather than a PDF signature (`25 50 44 46`).
5. **Real intent concealed.** `Money.xlsx` says the money was never the goal and announces war. A Base64 string in the workbook decodes to the location **The Martian Colony, Beside Interplanetary Spaceport.**
6. **Attribution lead.** `GoodJobMajor.pdf` lists the author as `Pestero Negeja`, matching the `negeja3921` Reply-To mailbox.

## 3. Indicators of Compromise

| Type | Value |
|---|---|
| Spoofed sender | `billjobs[@]microapple[.]com` |
| Reply-To / actor | `negeja3921[@]pashter[.]com` |
| Mail service | `emkei[.]cz` / `93.99.104.210` |
| Attachment | `PuzzleToCoCanDa.pdf` (actually a zip) |
| Contents | `GoodJobMajor.pdf`, `Money.xlsx`, `DaughtersCrown.jpeg` |
| Hidden location | Base64 in `Money.xlsx` |
| `GoodJobMajor.pdf` SHA-256 | `ff61467a025529020b55685df1102b44e28310aa79cd3ef2ea6a93af9477480b` |

## 4. Incident Summary (5Ws and How)

| Question | Answer |
|---|---|
| **Who** | Target: the Army Major on Earth. Actor: the sender using `negeja3921[@]pashter[.]com` (author name “Pestero Negeja”). |
| **What** | A spoofed extortion email (SPF fail) with a disguised ZIP attachment. |
| **When** | Tue, 26 Jan 2021 06:41:18 UTC. Google’s receipt stamp: Mon, 25 Jan 2021 22:41:18 -0800 (PST). |
| **Where** | Delivered to `themajoronearth[@]gmail[.]com`. |
| **Why** | The ransom was a decoy. The hidden message declares war on the CoCanDians. |
| **How** | The actor used emkei[.]cz to spoof the sender, then attached a ZIP renamed `.pdf` containing a PDF, an image and a spreadsheet with Base64-encoded content. |

## 5. Assessment

The email combines social engineering (urgency, a family emergency, a ransom) with technical deception (spoofed sender, wrong file extension, encoded content). The detail that the attacker will be at the disclosed location suggests the “rescue” lead may be bait.

## 6. Recommendations

1. **Do not pay the ransom.** It is a decoy.
2. **Verify the location lead independently** before dispatching a retrieval team to The Martian Colony.
3. **Preserve the original `.eml`** as evidence and do not reply to the Reply-To address.
4. **Block and alert** on `pashter[.]com`, `microapple[.]com` and `emkei[.]cz` at the mail gateway, and write network detections for traffic involving `pashter[.]com`.
5. **Enforce SPF/DMARC checks** and quarantine or reject mail that fails them.
6. **Quarantine attachments whose file extension does not match their file signature.**
7. **Flag Office documents containing Base64-encoded strings** for review.

## 7. Limitations

- The sender-supplied `Date` header can be chosen freely with the mailer. The send time is corroborated by the `Received` timestamp added by Google’s server, which gives the same instant (06:41:18 UTC).
- `93.99.104.210` belongs to the mail service and shows a 0% AbuseIPDB confidence score. It should not be treated as the attacker’s own address.
- The Reply-To mailbox and the PDF author name are attacker-controlled and are attribution leads, not confirmed identity.
