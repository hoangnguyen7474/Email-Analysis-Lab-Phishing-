# The Planet's Prestige: Suspicious Email and Attachment Analysis Lab

https://blueteamlabs.online/home/challenge/the-planets-prestige-e5beb8e545

A defensive-security write-up showing how to investigate a suspicious email from its headers through its nested attachment contents.

> **Training disclaimer:** This repository documents a CTF-style email-analysis lab. The indicators and story are presented as lab evidence, not as a confirmed real-world incident. No live malware is included.



## Project overview

The email appeared to come from `billjobs@microapple.com`, but the header evidence showed several warning signs:

- The message was sent from `93.99.104.210` through `emkei.cz`.
- SPF failed for the claimed `microapple.com` sender domain.
- The `From` and `Reply-To` domains did not match.
- The body and attachment content used Base64 encoding.
- An attachment named `PuzzleToCoCanDa.pdf` was actually a ZIP archive.
- Extracted files used missing or misleading extensions.
- A hidden Excel worksheet contained Base64-encoded text concealed by formatting.

The final decoded message identified the fictional location:

```text
The Martian Colony, Beside Interplanetary Spaceport.
```

## Learning objectives

This lab demonstrates how to:

1. Read an email's `Received` chain from the sender side toward the recipient side.
2. Review SPF, DKIM, and DMARC-related authentication results.
3. Compare `From`, `Reply-To`, `Return-Path`, and sending infrastructure.
4. Decode Base64 email content and attachments.
5. Validate a file using its magic bytes instead of trusting its extension.
6. Extract an archive and reveal hidden files.
7. Inspect JPEG, PDF, ZIP, and Office Open XML signatures.
8. Find hidden spreadsheet content and decode the final message.
9. Convert technical findings into an incident-style report.

## Investigation workflow

### 1. Analyze the email headers

The header review focused on the `Received` chain, sender IP, sending service, sender domain, reply address, message ID, MIME content type, and authentication results.
<img width="1318" height="692" alt="image" src="https://github.com/user-attachments/assets/6569cd41-74c9-4b85-a60a-33cb8d9d81dc" />



Key observations:

- Sender IP: `93.99.104.210`
- Sending service: `emkei.cz`
- Claimed sender domain: `microapple.com`
- Reply-To domain: `pashter.com`
- SPF result: `fail`

### 2. Decode the Base64 content

The message body was marked as Base64 encoded. It was decoded in CyberChef to reveal the readable email text.

<img width="1320" height="693" alt="image" src="https://github.com/user-attachments/assets/8b84eb06-919c-4d8a-ba7d-1c23969b1ad1" />


### 3. Validate the attachment type

Although the attachment was named `PuzzleToCoCanDa.pdf`, its leading bytes were:
<img width="1310" height="680" alt="image" src="https://github.com/user-attachments/assets/d0282335-6355-4e79-adc9-488d800e961a" />

```text
50 4B 03 04
```
<img width="1149" height="988" alt="image" src="https://github.com/user-attachments/assets/32892cb6-5fe1-4ec9-bc4f-50030244eb25" />

This signature identifies a ZIP-compatible archive, not a PDF. A normal PDF begins with:
<img width="1024" height="185" alt="image" src="https://github.com/user-attachments/assets/8207b30c-a016-4e11-96f6-b1908f660311" />

```text
25 50 44 46
```



### 4. Extract and inspect the files

After decoding and saving the attachment as a ZIP archive, the following files were found:

<img width="762" height="461" alt="image" src="https://github.com/user-attachments/assets/b338c1cd-2970-4504-8428-d72ee80643df" />


| File | Identified type | Evidence |
|---|---|---|
| `DaughtersCrown` | JPEG image | `FF D8 FF E0` |
| `GoodJobMajor` | PDF document | `25 50 44 46` |
| `Money.xlsx` | Office Open XML spreadsheet | ZIP/OOXML structure |

### 5. Follow the puzzle chain

The PDF directed the analyst to the image and spreadsheet. The spreadsheet contained multiple worksheets; a blank-looking sheet held text hidden by formatting.

<img width="1054" height="696" alt="image" src="https://github.com/user-attachments/assets/97afaa04-b0ef-4fb5-afb7-addec46f7d53" />


After clearing the formatting, a Base64 string became visible.

<img width="921" height="693" alt="image" src="https://github.com/user-attachments/assets/bceb62de-ffe2-4565-b137-b97535b2b09a" />


Decoding that string produced the final lab answer.

<img width="1323" height="412" alt="image" src="https://github.com/user-attachments/assets/c10df584-5d66-4fae-a302-bef90de4fda5" />


## Key findings

| Category | Finding |
|---|---|
| Email authentication | SPF failed for the claimed sender domain |
| Identity mismatch | `From` and `Reply-To` used different domains |
| Sending infrastructure | Message originated from `93.99.104.210` through `emkei.cz` |
| Encoding | Body and attachment data used Base64 |
| File masquerading | A `.pdf` filename contained ZIP-formatted data |
| Hidden content | A hidden spreadsheet worksheet concealed Base64 text |
| Malware | No malware family or executable payload was identified in the supplied lab evidence |
| Final result | `The Martian Colony, Beside Interplanetary Spaceport.` |


## Tools used in the lab

- Email header viewer or text editor
- CyberChef for Base64 decoding
- HxD or another hex editor
- A trusted file-signature reference
- Archive extraction utility
- Microsoft Excel, LibreOffice, or an isolated spreadsheet viewer

## Safe reproduction

Do not analyze unknown attachments on a production workstation. Use a disposable virtual machine or isolated sandbox, disable macros, and preserve the original email before modifying or extracting anything.

The repository intentionally excludes the original email and extracted evidence files. Add only sanitized lab artifacts to the `evidence/` directory.

## Investigation report

The complete report is available in https://github.com/hoangnguyen7474/Email-Analysis-Lab-Phishing-/blob/main/Email_Analysis_Investigation_Report.pdf.

## Portfolio summary

This project demonstrates email-header analysis, authentication review, MIME and Base64 decoding, file-signature validation, archive inspection, hidden-content discovery, IOC documentation, and security-report writing.
