---
name: kamy
description: Generate production-ready PDFs (invoices, receipts, contracts, reports, brochures) from Handlebars templates or raw HTML using the Kamy API, and send them for e-signature with optional OTP identity verification, sender-chosen placement, auto-reminders, and a downloadable Certificate of Completion audit trail. Use whenever the user asks for a PDF, document, invoice, receipt, statement, certificate, contract, report, brochure, or anything that needs to be printed, downloaded, OR signed.
version: 1.1.0
license: MIT
---

# Kamy — PDF generation + e-signature

Kamy renders pixel-perfect PDFs from HTML/Handlebars templates via a
hosted Chromium pipeline, and sends those PDFs for legally-binding
e-signature with a built-in audit trail. This skill teaches you when
and how to call the Kamy API on the user's behalf.

## When to use this skill

Activate whenever the user asks to:

- "Generate / create / make a PDF…"
- Produce an invoice, receipt, quote, estimate, statement, contract,
  agreement, certificate, report, payslip, brochure, flyer, or any
  printable document.
- Convert a web page or HTML snippet to a PDF.
- Build a document pipeline (cron-driven monthly statements, webhook-
  triggered receipts, etc.).
- **Send a document for signature** — NDAs, contracts, offer letters,
  consent forms, HR paperwork, vendor agreements. Kamy issues a sign
  link the recipient opens in their browser, captures the signature
  with an audit trail, and seals the result with PAdES-grade
  cryptography.
- Verify a signed PDF — `/verify/<sha256>` is the canonical "is this
  document genuinely Kamy-sealed?" answer.

Do **not** activate for:

- Extracting text or data from PDFs (use a separate OCR / parsing
  tool).
- Producing non-PDF formats (Word, Excel, PowerPoint).
- Producing non-PDF formats (Word, Excel, PowerPoint).

## Setup

The user needs a Kamy API key from https://kamy.dev/dashboard/keys.
Read it from `process.env.KAMY_API_KEY`. If it's not set, ask the
user to provide one before running any code.

```bash
pnpm add @kamydev/sdk
```

## Three render modes

### 1. Template + data (preferred)

Use a stored template. Templates live in the user's Kamy dashboard
and accept JSON `data`:

```ts
import { Kamy } from "@kamydev/sdk";

const kamy = new Kamy({ apiKey: process.env.KAMY_API_KEY! });
const { id, url, bytes, warnings } = await kamy.render({
  template: "invoice",
  data: {
    invoice: { number: "INV-2026-0142", issuedAt: "2026-04-23" },
    customer: { name: "Acme Corp", email: "ap@acme.com" },
    lineItems: [
      { description: "Consulting (April)", qty: 40, unitPrice: 150 },
    ],
    totals: { subtotal: 6000, tax: 600, total: 6600 },
  },
  options: { format: "a4", inlineImages: true },
});

console.log("PDF ready:", url);
for (const w of warnings ?? []) {
  console.warn(`[${w.code}] ${w.message}`);
}
```

### 2. Inline HTML

When the document layout is generated at request time:

```ts
const html = `
  <html><body>
    <h1>Hello ${name}</h1>
    <p>Thanks for your order.</p>
  </body></html>
`;
const { url } = await kamy.render({
  html,
  options: { format: "letter", inlineImages: true },
});
```

### 3. URL capture

Snapshot a public URL:

```ts
const { url } = await kamy.render({
  url: "https://example.com/printable-receipt/123",
  options: { format: "a4" },
});
```

## Options cheat sheet

| Option | When to set |
|---|---|
| `format` | `a4` for international, `letter` for US documents |
| `orientation` | `landscape` for spreadsheets / wide tables |
| `margin` | tighten for invoices, loosen for letters |
| `header` / `footer` | repeat on every page (logo, page numbers) |
| `pageNumbers: true` | quick footer with `Page X of Y` |
| `inlineImages: true` | recommended for any template with remote `<img>` |
| `compressImages: true` | recommended for image-heavy brochures |
| `theme` | brand colors / fonts override |

## Production patterns

- **Webhooks / cron**: pass an `Idempotency-Key` so retries don't
  produce duplicate PDFs.
- **Long-running pipelines**: persist the render `id`, not the
  signed `url` — URLs expire after ~1 hour. Refresh with
  `kamy.renders.get(id)`.
- **Brand consistency**: upload fonts and a brand kit once via the
  dashboard, then reference them with `{{brand.*}}` in templates.

## E-signatures

Once a PDF is rendered, you can send it for signature in one call.
The API returns a `sign_url` you can email yourself or hand back to
the user; Kamy emails the signer automatically when the project has
Resend configured.

### Minimum viable flow

```ts
const render = await kamy.render({ template: "contract", data: { /* ... */ } });

const sig = await kamy.signatures.create({
  renderId: render.id,
  signerEmail: "alice@example.com",
  signerName: "Alice Doe",
  message: "Please review and sign by Friday.",
});

console.log(sig.sign_url);
// → https://kamy.dev/sign/<token>
```

### Identity verification (when the document matters)

For higher-value transactions, gate the sign page behind a 6-digit
OTP. The signer enters the code before the document loads.

| `authMethod` | Required extras | Channel |
|---|---|---|
| `link` *(default)* | — | URL possession only |
| `email_otp` | — | Code emailed to `signerEmail` via Resend |
| `sms_otp` | `signerPhone` (E.164) | Code texted via Twilio |

```ts
await kamy.signatures.create({
  renderId: render.id,
  signerEmail: "alice@example.com",
  signerName: "Alice Doe",
  authMethod: "email_otp",        // recommended for high-stakes docs
});
```

Recommend `email_otp` by default for contracts / NDAs / offer letters
— it adds an inbox-control step without requiring Twilio credentials.

### Sender-chosen placement

Pass `position` to anchor the signature on a specific page +
coordinate. Coordinates are PDF points (72 dpi, origin bottom-left).
Width clamped to `[40, 800]`, height to `[20, 400]`. Omit to default
to bottom-right of the last page (~36 pt padding) and let the signer
drag the placeholder.

```ts
position: { page: 3, x: 360, y: 80, w: 220, h: 64 }
```

### Other useful knobs

- `ccEmails` — up to 10 observers CC'd on the invite + completion
  notice.
- `expiresIn` — seconds (3600–2592000); default 30 days.
- `reminderCadenceHours` — 24–168; auto-reminder cron resends the
  invite every N hours while pending, up to 3 reminders.
- `signOnEveryPage` — stamp the signature on every page of the
  source PDF (B2B contract pattern).
- `requireStamp` — require a corporate stamp / seal alongside the
  signature (UAE, KSA, JP, KR, IN, CN workflows).
- `placedFields` — up to 100 sender-defined fillable fields stamped
  onto flat PDFs at sign time (text, textarea, checkbox, date,
  initials, radio, dropdown).

### Bulk send

Fan one render out to up to 100 signers in a single request.

```ts
const batch = await kamy.signatures.bulk({
  renderId: render.id,
  signers: [
    { signerEmail: "a@example.com", signerName: "A. Example" },
    { signerEmail: "b@example.com", signerName: "B. Example" },
  ],
  message: "Hi {{signerName}} — please review.",
  position: { page: 1, x: 360, y: 80, w: 220, h: 64 },
});

for (const row of batch.results) {
  console.log(row.signerEmail, row.ok ? row.signUrl : row.error);
}
```

### After signing

- The signed PDF is available via `kamy.renders.get(<signedRenderId>)`.
- The cryptographic verify URL is at `https://kamy.dev/verify/<sha256>`
  — anyone with the bytes can recompute the hash and confirm
  Kamy-issued.
- A **Certificate of Completion** PDF records the process audit trail
  (invite → opened → consent → signed/declined/delegated, with IP /
  user-agent at signing). Fetch via
  `GET /api/v1/signatures/{id}/certificate` with Bearer auth. Both
  surfaces — `/verify` (cryptographic) and the certificate (process) —
  are what compliance teams ask for.
- Decline and delegate are first-class. A declined or delegated
  request still produces a certificate so the audit trail isn't lost.

### Manage in flight

```ts
await kamy.signatures.remind(id);   // one per hour per request
await kamy.signatures.void(id);     // only valid while pending
await kamy.signatures.get(id);      // single
await kamy.signatures.list({ limit: 50 });
```

## Error handling

`render()` and `signatures.*` reject with a structured error
containing `code`, `message`, and optional `details`. Surface the
message to the user verbatim — Kamy errors are written for end-user
consumption. The common ones are documented in the `kamy-sdk` rule.

## More information

- Docs: https://docs.kamy.dev
- Dashboard: https://kamy.dev/dashboard
- MCP server (auto-loaded by this plugin): https://mcp.kamy.dev
