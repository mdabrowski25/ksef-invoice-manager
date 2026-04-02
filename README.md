# @mdab25/ksef-pdf

Node.js package for generating PDF visualizations from KSeF XML documents.

- Invoices: `FA(1)`, `FA(2)`, `FA(3)`
- UPO: `UPO(4.2)`, `UPO(4.3)`
- Runtime: Node.js `>=20`

This package vendors and adapts renderer logic from `CIRFMF/ksef-pdf-generator` for server-side use.

## Wsparcie / Usługi

Potrzebujesz wsparcia przy integracji z KSeF, generowaniu PDF z faktur XML  
lub budowie własnych narzędzi wokół KSeF?

Skontaktuj się ze mną – chętnie pomogę w implementacji, integracji lub  
rozwiązaniu konkretnych problemów technicznych.

LinkedIn: https://www.linkedin.com/in/mateusz-dabrowski25


## Install

```bash
npm install @mdab25/ksef-pdf
```

## Quick Start

```ts
import { readFile, writeFile } from 'node:fs/promises';
import { renderPdfFromXml } from '@mdab25/ksef-pdf';

const xml = await readFile('./invoice.xml', 'utf8');
const pdf = await renderPdfFromXml(xml);

await writeFile('./invoice.pdf', pdf);
```

## API

### Types

```ts
type KsefInvoiceVersion = 'FA(1)' | 'FA(2)' | 'FA(3)';
type KsefUpoVersion = 'UPO(4.2)' | 'UPO(4.3)';
```

### Functions

```ts
function detectInvoiceVersion(xml: string): KsefInvoiceVersion | null;
function detectUpoVersion(xml: string): KsefUpoVersion | null;

function renderPdfFromXml(
  xml: string | Uint8Array | ArrayBuffer | Blob,
  options?: { nrKSeF?: string; qrCode?: string },
): Promise<Uint8Array>;

function renderPdfBase64FromXml(
  xml: string | Uint8Array | ArrayBuffer | Blob,
  options?: { nrKSeF?: string; qrCode?: string },
): Promise<string>;

function renderUpoPdfFromXml(
  xml: string | Uint8Array | ArrayBuffer | Blob
): Promise<Uint8Array>;
```

#### `options`

| Field    | Type     | Description |
|----------|----------|-------------|
| `nrKSeF` | `string` | KSeF invoice number displayed on the PDF. When omitted the label defaults to `OFFLINE`. |
| `qrCode` | `string` | KSeF QR verification URL. When provided, a QR code is rendered on the PDF visualization. |

Compatibility exports (upstream-like):
- `generateInvoice`
- `generatePDFUPO`
- `generateFA1`
- `generateFA2`
- `generateFA3`

## QR Code Support

Since `v0.2.0` you can embed a KSeF QR verification code on the PDF:

```ts
const pdf = await renderPdfFromXml(xml, {
  nrKSeF: '1234567890-20260101-ABC123-DE',
  qrCode: 'https://qr.ksef.mf.gov.pl/invoice/1111111111/01-02-2026/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE',
});
```

The QR verification URL format follows the official KSeF specification:
```
https://qr.ksef.mf.gov.pl/invoice/{NIP}/{DD-MM-YYYY}/{Base64URL-SHA256}
```

See [KSeF QR code docs](https://github.com/KSeF-CIRFMF/ksef-docs/blob/main/kody-qr.md) for details.

## Notes

- This package renders PDFs from XML schema content only.

## Minimal HTTP Service Example (n8n-friendly)

```ts
import express from 'express';
import { renderPdfFromXml } from '@mdab25/ksef-pdf';

const app = express();
app.use(express.text({ type: ['application/xml', 'text/xml'], limit: '10mb' }));

app.get('/health', (_req, res) => res.send('OK'));

app.post('/render', async (req, res) => {
  const pdf = await renderPdfFromXml(req.body, {
    nrKSeF: req.headers['x-ksef-number'] as string,
    qrCode: req.headers['x-qr-code'] as string,
  });
  res.setHeader('Content-Type', 'application/pdf');
  res.send(Buffer.from(pdf));
});

app.listen(3100);
```

## Development

```bash
npm install
npm run typecheck
npm test
npm run build
```

## Publish

```bash
npm login
npm publish --access public
```

Before publishing, verify package contents:

```bash
npm pack --dry-run
```

## License

This package is licensed under `MIT` (see `LICENSE`).

It also includes adapted third-party source code. See:
- `THIRD_PARTY_NOTICES.md`
- `LICENSES/CIRFMF-ksef-pdf-generator-ISC.txt`

