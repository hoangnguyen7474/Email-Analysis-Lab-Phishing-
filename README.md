# Suspicious Email and Attachment Analysis Lab

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

![Authentication results](assets/featured/02-authentication-results.png)

Key observations:

- Sender IP: `93.99.104.210`
- Sending service: `emkei.cz`
- Claimed sender domain: `microapple.com`
- Reply-To domain: `pashter.com`
- SPF result: `fail`

### 2. Decode the Base64 content

The message body was marked as Base64 encoded. It was decoded in CyberChef to reveal the readable email text.

![Base64 decoding](assets/featured/03-base64-decoding.png)

### 3. Validate the attachment type

Although the attachment was named `PuzzleToCoCanDa.pdf`, its leading bytes were:

```text
50 4B 03 04
```

This signature identifies a ZIP-compatible archive, not a PDF. A normal PDF begins with:

```text
25 50 44 46
```

![ZIP magic bytes](assets/featured/04-zip-magic-bytes.png)

### 4. Extract and inspect the files

After decoding and saving the attachment as a ZIP archive, the following files were found:

| File | Identified type | Evidence |
|---|---|---|
| `DaughtersCrown` | JPEG image | `FF D8 FF E0` |
| `GoodJobMajor` | PDF document | `25 50 44 46` |
| `Money.xlsx` | Office Open XML spreadsheet | ZIP/OOXML structure |

![Extracted files](assets/featured/05-extracted-files.png)

### 5. Follow the puzzle chain

The PDF directed the analyst to the image and spreadsheet. The spreadsheet contained multiple worksheets; a blank-looking sheet held text hidden by formatting.

![Puzzle PDF](assets/featured/08-puzzle-pdf.png)

After clearing the formatting, a Base64 string became visible.

![Hidden Base64 string](assets/featured/10-hidden-base64.png)

Decoding that string produced the final lab answer.

![Decoded location](assets/featured/11-decoded-location.png)

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

## Repository structure

```text
.
├── README.md
├── REPORT.md
├── GITHUB_UPLOAD_GUIDE.md
├── docs/
│   ├── analysis-methodology.md
│   └── evidence-handling.md
├── iocs/
│   └── indicators.csv
├── scripts/
│   ├── hash_evidence.py
│   └── identify_file_type.py
├── evidence/
│   └── README.md
├── assets/
│   ├── featured/
│   └── screenshots/
└── .github/workflows/
    └── python-check.yml
```

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

The complete report is available in [REPORT.md](REPORT.md).

## Portfolio summary

This project demonstrates email-header analysis, authentication review, MIME and Base64 decoding, file-signature validation, archive inspection, hidden-content discovery, IOC documentation, and security-report writing.
