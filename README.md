# The Planet's Prestige: Email Investigation

Digital forensics / phishing analysis write-up for the **Blue Team Labs Online** challenge *The Planet's Prestige*.

> **Scenario:** CoCanDa, "The Heaven of the Universe", is suffering a wave of unexplained abductions of its citizens (CoCanDians). The Planetary President's daughter has also disappeared. On the third day, the CoCanDa representative, an Army Major on Earth, receives an email.

---

## Tools used

- Notepad++ (raw `.eml` inspection)
- CyberChef (Base64 decoding, hex conversion)
- Gary Kessler's file signature table (magic bytes)
- AbuseIPDB and VirusTotal (reputation checks)
- Metadata viewer (PDF author, hash)

---

## Investigation summary

The Major received an email that appeared to come from `billjobs@microapple[.]com`, but the `Reply-To` address was `negeja3921@pashter[.]com`. The email was sent through **emkei[.]cz**, a free fake-mailer service (server IP `93.99.104.210`, Prague, Czechia), which explains the spoofed sender.

The Base64-encoded body is a ransom demand for **1 Billion CoCanDs**, claiming the abducted CoCanDians and the President's daughter are safe at a secret location. It tells the Major to solve a puzzle in the attachment and hints "Don't Trust Your Eyes".

The attachment `PuzzleToCoCanDa.pdf` is **not a PDF**. Its Base64 begins with `UEsDBBQA`, which decodes to `50 4B 03 04` (ZIP magic bytes). Extracting it reveals three files:

1. `GoodJobMajor.pdf` says the CoCanDians are "safe", points to `DaughtersCrown` as proof, and says the payment location is in `Money.xlsx`.
2. `DaughtersCrown.jpeg` is a crown emoji image presented as "proof".
3. `Money.xlsx` contains the real message: the ransom was a decoy and the actual intent is **war with the CoCanDians**. A hidden Base64 string decodes to the location **"The Martian Colony, Beside Interplanetary Spaceport."**

The PDF author metadata (**Pestero Negeja**) matches the `negeja3921@pashter[.]com` Reply-To, which ties the document to the sender.

---

## Email header evidence

| Field | Value |
|---|---|
| To | `themajoronearth@gmail[.]com` |
| Subject | A Hope to CoCanDa |
| From | "Bill" `billjobs@microapple[.]com` |
| Errors-To | `billjobs@microapple[.]com` |
| Reply-To | `negeja3921@pashter[.]com` |
| X-Priority / Importance | 3 (Normal) / Normal |
| MIME-version | 1.0 |
| Content-Type | `multipart/mixed; boundary=BOUND_600FB98E0DCEE8.49207210` |
| Message-Id | `<20210126064118.1993E221F8@localhost>` |
| Date (sent) | Tue, 26 Jan 2021 01:41:18 -0500 (EST) = 06:41:18 UTC |

The `Date` header is set by the sender. emkei lets the sender choose it, so treat it as the claimed send time, not a verified one.

---

## Indicators of Compromise (IOCs)

| Type | Value |
|---|---|
| Spoofed sender | `billjobs@microapple[.]com` |
| Reply-To / actor | `negeja3921@pashter[.]com` |
| Mail service | `emkei[.]cz` / `93.99.104.210` |
| Attachment | `PuzzleToCoCanDa.pdf` (actually a zip) |
| Contents | `GoodJobMajor.pdf`, `Money.xlsx`, `DaughtersCrown.jpeg` |
| Hidden location | Base64 in `Money.xlsx` |

**Additional notes**

- `GoodJobMajor.pdf` SHA-256: `ff61467a025529020b55685df1102b44e28310aa79cd3ef2ea6a93af9477480b`
- `93.99.104.210` is infrastructure belonging to the mail service (ISP: FinalTek, Prague), not the attacker's own host. AbuseIPDB shows 28 reports and a 0% confidence score.
- `emkei[.]cz` was flagged as malicious/phishing by 6/92 vendors on VirusTotal.

---

## 5Ws and How

- **Who:** the Army Major on Earth (target); the sender operating as "Pestero Negeja" (actor).
- **What:** received a spoofed email containing a ransom demand and a disguised zip attachment.
- **When:** header date Tue, 26 Jan 2021 01:41:18 -0500 (EST).
- **Where:** delivered to `themajoronearth@gmail[.]com`.
- **Why:** the ransom was a decoy. `Money.xlsx` reveals the true intent, a declaration of war on CoCanDians.
- **How:** the actor used emkei[.]cz to spoof `billjobs@microapple[.]com`, with a zip disguised as a `.pdf` containing a PDF, an image and a spreadsheet with hidden Base64 content.

---

## Recommendations

1. **Do not pay the ransom.** It is a decoy.
2. Treat the "rescue" location as possible bait (the actor says they will be there too). Verify before sending a retrieval team.
3. Preserve the original `.eml` as evidence and do not reply to the Reply-To address.
4. Block and alert on `pashter[.]com`, `microapple[.]com` and `emkei[.]cz` at the mail gateway and write network detections for traffic involving them.
5. Quarantine mail whose file extension does not match the real file type (magic bytes check).
6. Flag attachments that contain Base64-encoded strings or hidden content in Office documents.

---

## Lessons learned

- Always read the raw headers: compare `From`, `Reply-To`, `Errors-To` and `Message-Id`.
- Never trust file extensions. Check magic bytes (`UEsDBBQA` in Base64 = `PK\x03\x04`).
- Check the XML inside Office files (`xl/sharedStrings.xml`) for hidden text.
- Defang IOCs before sharing (`example[.]com`).
- A sender's IP in a header from a fake-mailer service identifies the service, not the attacker.
