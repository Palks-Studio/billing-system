<p align="center">
  <img src="docs/images/paid_en.png"
       alt="Invoice payment interface — view pending invoices and mark them as paid"
       width="600">
</p>

> 🇬🇧 English | [🇫🇷 Français](./README_FR.md)

![License](https://img.shields.io/badge/License-LICENSE.md-lightgreen.svg)
![Offline First](https://img.shields.io/badge/Mode-Offline%20First-0095b1?style=flat)
![Self Hosted](https://img.shields.io/badge/Hosting-Self%20Hosted-6b7280?style=flat)
![PHP](https://img.shields.io/badge/PHP-8%2B-777BB4?style=flat&logo=php&logoColor=white)
[![Invoicing without SaaS](https://img.shields.io/badge/Invoicing%20without%20SaaS-0095b1?style=flat)](https://palks-studio.com/en/invoicing-without-saas)

<p align="center">
  <a href="https://palks-studio.com">
    <img src="https://img.shields.io/badge/Palks%20Studio-Website-0095b1?style=for-the-badge" />
  </a>
</p>


# Billing System

> ⚠️ This repository presents the project and its technical documentation.  
> The production version is not publicly distributed.

> Installation is performed directly on the client's hosting infrastructure.  
> If you are interested in using this system, please contact **Palks Studio**.

Complete, autonomous and bilingual (FR/EN) billing system deployable on any PHP/Apache hosting. No database. No SaaS dependency. Self-hosted with full ownership of your data.

[![Invoicing without SaaS](https://img.shields.io/badge/Invoicing%20without%20SaaS-0095b1?style=flat)](https://palks-studio.com/en/invoicing-without-saas)

---

## Overview

Billing System is a suite of three interconnected billing tools accessible from a unified interface. It covers the full lifecycle of a service engagement: from quote generation to invoice settlement, including electronic signature and structured archiving.

The system is designed to be deployed directly on the client's server, on a standard Apache hosting environment with PHP 8.x and Composer. It requires no database, no third-party service, and no subscription.

---

## Features

- Interface de création de devis et factures avec génération PDF côté serveur via mPDF  
- Automatic pre-generation of the paid invoice at billing time  
- Electronic quote signing by the client (touch/mouse canvas)  
- Client auto-fill from archives (SIREN, SIRET, VAT, email, name)  
- Structured archiving by client and period  
- Secure sequential numbering (file lock)  
- Monthly export of invoices (ZIP)  
- Monthly export of revenue records (CSV)  
- Yearly export of the revenue journal (CSV)  
- Automatic email notifications at each stage (quote, invoice, settlement)  
- Cross-module navigation bar  
- Bilingual FR/EN interface with real-time language switch  
- Dark mode / light mode with persistence  
- No database  
- No SaaS dependency  
- Basic security: secure sessions, tokens, brute-force protection  
- International support: automatic VAT legal notice based on client zone (FR / EU / non-EU)  
- VAT forced to 0% for EU and non-EU clients, with appropriate legal notice  
- 249 countries centralised in `countries.php` — ISO code in backend, full name in UI  
- Country autocomplete without accents, displayed as full name, transmitted as ISO  
- VAT column hidden in documents when no VAT applies  
- `vat_zone` column in CSV revenue journal (values: `fr`, `eu`, `world`)

---

## Project Structure

```
billing-system/
│
├── public/
│   │  └── assets/
│   │      ├── logo*              → Company logo if provided
│   │      ├── signature.png      → User signature used on quotes and invoices (PNG format)
│   │      └── favicon*           → Optional favicon displayed in the browser tab
│   │
│   ├── Endpoint a                → Quote generation endpoint
│   ├── Endpoint b                → Invoice generation endpoint
│   ├── Endpoint c                → Direct invoice generation endpoint
│   ├── Endpoint d                → Paid invoice generation endpoint
│   │
│   ├── Interface a               → Quote generation interface
│   ├── Interface b               → Direct invoice generation interface
│   ├── Interface c               → Payment tracking interface
│   ├── sign.php                  → Signature interface
│   ├── export a                  → Archived invoices ZIP export
│   ├── export b                  → Revenue journal CSV export
│   │
│   ├── countries.php             → List of 249 countries in ISO format
│   ├── search.php                → Client search and auto-fill
│   ├── serve.php                 → Secure PDF access via token
│   └── save.php                  → Quote save and archiving
│
├── vendor/                       → Libraries used by the document generation engine
├── templates/                    → HTML templates used for document rendering
│
├── app.php                       → Central configuration for issuer and bank details
├── mailer.php                    → Internal email sending script with attachments
├── core.php                      → Main engine: generation logic, calculations and archiving
├── LICENSE.md                    → Project license
│
├── contracts/                    → Signed and unsigned quote archiving
├── counters/                     → Numbering counters (quotes and invoices)
├── logs/                         → System logs (optional)
├── data/                         → Operational data
│
└── docs/
    ├── USER_GUIDE.md             → User guide
    ├── OVERVIEW.md               → Project overview and general system description
    └── README.md                 → Installation and usage documentation (client version)
```


---

## The Three Modules

### 1. Quote generator

**How it works:**  

1. The user fills in the form: issuer details, client details, service lines, bank details, and settings (currency, PDF language).  
2. A live preview of totals (excl. VAT / VAT / incl. VAT) is calculated in real time.  
3. On submission, a confirmation dialog appears before generation.  
4. The PDF is generated locally and downloaded. At the same time, the quote is archived server-side with a signature token valid for 30 days.  
5. An email is sent to the client with a link to review and sign the quote online.

**Technical details:**  

- Automatic numbering based on the client’s existing records  
- Client auto-fill using SIREN, SIRET, VAT number, email, or name  
- Client-side validation of SIREN/SIRET (Luhn algorithm)  
- Automatic SIREN extraction from SIRET  
- Real-time language switch (FR/EN) without page reload  
- Currency selector: EUR, USD, GBP, CHF, CAD  
- Conditional display of bank details (IBAN/BIC) in the PDF  
- “Approved and agreed” section with configurable signature image  
- Conditional VAT footer (e.g. VAT exemption under art. 293B CGI when VAT = 0)  
- Automatic PDF pagination

---

### 2. Direct invoicing

Server-side invoice generation interface. For each operation, two PDFs are produced: the standard invoice and a pre-generated “paid” version (pending payment validation).

**How it works:**  

1. The user fills in the form: issuer details, client details, service lines, bank details, and settings (currency, PDF language).  
2. A live preview of totals (excl. VAT / VAT / incl. VAT) is calculated in real time.  
3. On submission, a confirmation dialog appears before generation.  
4. The invoice PDF is generated and downloaded. At the same time, the documents are archived server-side for tracking and payment validation.  
5. An email is sent to the client with the invoice attached.

**Technical details:**

- Annual invoice numbering with collision protection  
- Protection against duplicate invoicing from the same quote reference  
- Full input validation with bilingual error handling (FR/EN)  
- Accurate financial calculations with float precision handling  
- VAT aggregation per rate  
- Automatic logo processing for PDF compatibility  
- Automatic retrieval of issuer information from existing metadata  
- PDF traceability via cryptographic hash (SHA-256)  
- Email delivery with PDF attachment, without external dependencies

---

### 3. Payment validation and tracking

Interface for tracking outstanding invoices and validating payments. Allows users to identify unpaid invoices, mark them as paid, and monitor revenue.

**How it works:**  

1. The module automatically detects invoices awaiting payment.  
2. The user selects a payment date and clicks “Mark as paid”.  
3. The system archives the invoice as paid, updates the metadata, records the transaction in the revenue log, and sends the invoice to the client by email.  
4. A table displaying the latest revenue entries of the year is shown at the bottom of the page.

**Technical details:**  

- Automatic detection of invoices to process  
- Filtering of invoices already marked as paid  
- Prevention of duplicate entries in the revenue log  
- Safe write operations with file locking  
- Protection against form resubmission  
- Pending invoices counter indicator  
- Annual CSV export of revenue

---

### Shared modules

#### Electronic signature

Public page accessible via a secure tokenized link. Allows the client to view the quote and provide an electronic signature.

- Multi-page PDF preview in the browser  
- Signature capture (touch and mouse compatible)  
- Input validation (format and minimum size)  
- Signature image storage  
- Metadata update with signature timestamp and traceability  
- Confirmation emails sent to both client and issuer (FR/EN)  
- Protection against duplicate signing  
- Link expiration after 30 days

#### Client auto-fill

JSON endpoint used during form input. Searches existing records based on SIREN, SIRET, VAT number, email, name, or quote number.

- Returns full client details, language, currency, and related records  
- Allows automatic pre-fill of invoice data from previous quotes  
- Input normalization for consistent matching

#### Secure PDF access

Token-based access layer for serving PDF files without exposing their physical location.

#### Internal mailer

Email sending module with attachment support, used across all features. No external SMTP service required.

---

## Security

- Secure session management (protected cookies, access isolation)  
- Protection against brute-force attempts  
- Session identifier regeneration after authentication  
- Use of secure tokens for accessing sensitive resources  
- Strict validation of inputs and identifiers  
- Action traceability via cryptographic hashing  
- Endpoint protection through access control mechanisms  
- HTTP response security policies (content type, caching, indexing)  
- Internal interfaces are not indexed and not publicly exposed  
- No external dependencies: all data remains under full control

---

## Automatic Emails

| Event              | Recipient(s)    | Content                            |
|--------------------|-----------------|------------------------------------|
| Quote generation   | Client          | Signature link + PDF download link |
| Quote signed       | Client + Issuer | Signature confirmation + PDF link  |
| Invoice generation | Client          | Invoice as attachment              |
| Settlement         | Client          | Paid invoice as attachment         |

All emails are bilingual FR/EN based on the document language.

---

## Technical requirements

- PHP 8.1 or higher  
- Compatible web server (Apache recommended)  
- URL rewriting support  
- GD extension (image processing)  
- Email sending capability enabled on the server  
- Write permissions on storage and archive directories

---

## Data storage

All data is stored using flat files. No database is required.

| Type                     | Description                                   | Format              |
|--------------------------|-----------------------------------------------|---------------------|
| Quotes                   | Archived with associated metadata             | PDF + metadata      |
| Signatures               | Stored as image files                         | PNG                 |
| Invoices                 | Archived with associated metadata             | PDF + metadata      |
| Pending paid invoices    | Pre-generated and awaiting validation         | PDF                 |
| Validated paid invoices  | Archived after payment confirmation           | PDF                 |
| Revenue records          | Structured export of transactions             | CSV                 |
| Counters                 | Sequential numbering tracking                 | Integer             |
| Logs                     | System activity tracking                      | Timestamped text    |

---

© Palks Studio — see LICENSE.md  
- https://palks-studio.com
