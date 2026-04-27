---
name: kamy
description: Generate production-ready PDFs (invoices, receipts, contracts, reports, brochures) from Handlebars templates or raw HTML using the Kamy API. Use whenever the user asks for a PDF, document, invoice, receipt, statement, certificate, contract, report, brochure, or anything that needs to be printed or downloaded as a PDF.
version: 1.0.0
license: MIT
---

# Kamy — PDF generation

Kamy renders pixel-perfect PDFs from HTML/Handlebars templates via a
hosted Chromium pipeline. This skill teaches you when and how to call
the Kamy API on the user's behalf.

## When to use this skill

Activate whenever the user asks to:

- "Generate / create / make a PDF…"
- Produce an invoice, receipt, quote, estimate, statement, contract,
  agreement, certificate, report, payslip, brochure, flyer, or any
  printable document.
- Convert a web page or HTML snippet to a PDF.
- Build a document pipeline (cron-driven monthly statements, webhook-
  triggered receipts, etc.).

Do **not** activate for:

- Editing existing PDFs (Kamy only generates, it doesn't manipulate).
- Extracting text or data from PDFs (use a separate OCR / parsing
  tool).
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

## Error handling

`render()` rejects with a structured error containing `code`,
`message`, and optional `details`. Surface the message to the user
verbatim — Kamy errors are written for end-user consumption. The
common ones are documented in the `kamy-sdk` rule.

## More information

- Docs: https://docs.kamy.dev
- Dashboard: https://kamy.dev/dashboard
- MCP server (auto-loaded by this plugin): https://mcp.kamy.dev
