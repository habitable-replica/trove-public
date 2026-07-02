# Vendored third-party libraries

Local copies of the JavaScript libraries the app depends on, so Trove keeps working
if a CDN removes a file or is unreachable, and so it can run fully offline / self-hosted.
Every file here is a **verbatim** copy of a specific published version — nothing is
modified. This file documents where each came from so you can independently verify it.

## How the app loads them (in `index.html` / `staff.html`)

| Library | Version | Loading strategy | Global |
|---|---|---|---|
| **elliptic** | 6.5.4 | **self-hosted** (this folder only — no CDN, so no third party in the crypto path) | `window.elliptic` |
| **snarkjs** | 0.7.3 | CDN-primary (jsDelivr) + **SRI** + local fallback (this folder) | `window.snarkjs` |
| **xlsx** (SheetJS) | 0.18.5 | CDN-primary (jsDelivr) + **SRI** + local fallback | `window.XLSX` |
| **papaparse** | 5.4.1 | CDN-primary (jsDelivr) + **SRI** + local fallback | `window.Papa` |

- **SRI** = Subresource Integrity: the browser verifies the CDN file against the hash below
  and refuses to run it if it doesn't match. If the CDN file is missing or fails SRI, the
  small inline `document.write` fallback loads the copy from this folder instead.
- **Google Fonts** (cosmetic) is still loaded from `fonts.googleapis.com`; the CSS falls
  back to the system `sans-serif` if it's unreachable. Not vendored.

## Integrity hashes (SRI, SHA-384)

```
snarkjs-0.7.3.min.js      sha384-HLrBpnE0aMqTKCyu06aW2jOHQ9l7ie7Yr4H8RguQHHFDThjEGIlfcVLN+Vd9D4ag
xlsx-0.18.5.full.min.js   sha384-vtjasyidUo0kW94K5MXDXntzOJpQgBKXmE7e2Ga4LG0skTTLeBi97eFAXsqewJjw
papaparse-5.4.1.min.js    sha384-D/t0ZMqQW31H3az8ktEiNb39wyKnS82iFY52QPACM+IjKW3jDUhyIgh2PApRqJZs
```

Verify any file:
```bash
printf 'sha384-%s\n' "$(openssl dgst -sha384 -binary <file> | openssl base64 -A)"
```

## Provenance / how to regenerate

`snarkjs`, `xlsx`, `papaparse` are the exact files from their npm packages (which is also
what jsDelivr serves at `/npm/<pkg>@<version>/<path>`):

```bash
npm install --no-save snarkjs@0.7.3 xlsx@0.18.5 papaparse@5.4.1
cp node_modules/snarkjs/build/snarkjs.min.js   public/vendor/snarkjs-0.7.3.min.js
cp node_modules/xlsx/dist/xlsx.full.min.js     public/vendor/xlsx-0.18.5.full.min.js
cp node_modules/papaparse/papaparse.min.js     public/vendor/papaparse-5.4.1.min.js
```

`elliptic`'s npm package ships only CommonJS source (no browser bundle), so we build a
UMD/IIFE bundle from that exact source with esbuild:

```bash
npm install --no-save elliptic@6.5.4 esbuild
printf 'module.exports={};\n' > tools/empty.js   # stub Node's crypto (dead branch in-browser)
npx esbuild ./node_modules/elliptic/lib/elliptic.js \
  --bundle --minify --format=iife --global-name=elliptic --platform=browser \
  --alias:crypto=./tools/empty.js --outfile=public/vendor/elliptic-6.5.4.min.js
```

The elliptic bundle is verified against the reference package: for the same private keys it
produces identical public keys and identical ECDH shared secrets (see the vendoring commit).
