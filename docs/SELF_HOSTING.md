# Trove — Self-Hosting & Independence

*How to run Trove without depending on its original developer, and an honest map of what
is and isn't independent today.*

The headline claim is **"runs independently of the developer."** This doc states exactly
how far that's true, what you do to self-host, and where the gaps are.

---

## The three layers (and who you depend on for each)

Trove is three separable things. Independence has to be evaluated per layer:

| Layer | What it is | Dependency today | Self-host path |
|---|---|---|---|
| **App** | the UI + all crypto | a static folder | host it anywhere, or run from disk |
| **Identity** | who you are / your key | none — client-side | MetaMask or passkey; nothing server-side |
| **Storage** | the encrypted vault | a pinning service (Pinata) | your own IPFS node or another pinning service |

The app and identity layers are **genuinely developer-independent today**. The storage
layer is the one that still leans on a third party out of the box — that's the honest gap,
addressed below.

---

## 1. The app — fully independent

`public/index.html` is a self-contained HTML app: all logic and cryptography run in the
browser, with **no backend and no build step**. The host (e.g. Cloudflare) is just a static
file host — **not** a dependency, only a convenience. You can serve `public/` from:

- any static host (Netlify, GitHub Pages, S3, nginx, …),
- IPFS itself (pin it; open by CID through any gateway),
- or **locally** — serve the `public/` folder (Passkeys/WebAuthn need `https://` or
  `localhost`; MetaMask also works from `file://`).

**Third-party libraries (and offline).** The app uses four JS libraries:

- **elliptic** (secp256k1 ECDH — crypto) is **self-hosted** from `public/vendor/` — no
  third party sits in the crypto path.
- **snarkjs**, **xlsx**, **papaparse** load from a CDN (jsDelivr) with **Subresource
  Integrity** (the browser rejects a tampered file) plus a **local fallback** in
  `public/vendor/` that loads automatically if the CDN is unreachable, removed, or fails SRI.
- **Google Fonts** is cosmetic and falls back to the system font if unreachable.

So Trove runs **fully offline / with no external requests** when you serve the whole
`public/` folder — the CDN fetches fail and the vendored copies take over — while a normal
hosted deploy still uses the CDN for those three libs to offload bandwidth. Provenance and
integrity hashes for every vendored file are in
[`public/vendor/README.md`](../public/vendor/README.md).

**Verify what you run matches the source.** Hash `index.html` against this repo, hash the
`vendor/` files against `public/vendor/README.md`, or serve from IPFS so the CID *is* the
integrity check — no trust in anyone's servers required.

### Deploy your own instance
```bash
git clone https://github.com/habitable-replica/trove-public
cd trove-public
npx serve public          # or push public/ to any static host
```

---

## 2. Identity — fully independent

Your identity and encryption key are derived **in your browser** from a MetaMask signature
or a passkey. The key never leaves your device, and the operator holds nothing
cryptographic — no keys, wallets, or recovery shares. There is no login server to depend on
or replace.

---

## 3. Storage — the layer to self-host

The vault is an **encrypted blob** stored on IPFS, addressed by a content hash (CID). Two
operations matter:

- **Pinning** (write): keeping the blob alive on IPFS. Today via a pinning service (Pinata).
- **Fetching** (read): retrieving a CID. Already **configurable** in-app (dedicated gateway
  + optional gateway token, with a public-gateway fallback race).

### Reality check on IPFS
IPFS does **not** auto-replicate. A private encrypted blob lives **only** where it's
explicitly pinned. If the only pin is on a pinning account you lose access to, the blob is
effectively gone. So self-hosting storage = **you control the pinning**, plus the
non-negotiable spine: **keep full local encrypted backups** (the app's export). A backup is
the only thing that guarantees the data still *exists* if a pinning provider is lost.

### Option A — Run your own IPFS node (most independent)
1. Run [Kubo](https://docs.ipfs.tech/install/command-line/) (the reference IPFS node).
2. Point Trove's **fetch** path at your node's gateway (Settings → *Private gateway URL*,
   e.g. `http://localhost:8080`), so reads come from your own infrastructure.
3. Pin the vault CIDs to your node (`ipfs pin add <cid>`).
4. **Current limitation:** the in-app **write/pin** call is still wired to Pinata's API
   (`PINATA_PIN` in `public/index.html`). Until the pin endpoint is configurable (see
   roadmap), self-pinning means re-pinning the CIDs the app produces to your node, or
   pointing the constant at a node that speaks Pinata's API. Reads are fully yours today;
   writes are the remaining piece.

### Option B — A different pinning service
Use another provider (e.g. Filebase, web3.storage, your own Pinata Cluster). If it exposes
a **Pinata-compatible** `pinJSONToIPFS` endpoint, set the JWT in Settings and you're done.
If it uses the standard **IPFS Pinning Service API** instead, it needs the small adapter
noted in the roadmap. Add the provider's gateway in Settings for reads.

### Option C — Own your pinning account (simplest)
Keep using Pinata, but on **your own** account that you control. The app is unchanged; the
original developer is no longer in the loop for storage. Pair with regular local backups so
the account is never a single point of failure.

---

## What survives if the developer disappears

Recovery needs three legs, and all three can be held by you:

1. **The code** — open source (this repo); a static folder that runs fully offline.
2. **The encrypted vault** — wherever *you* pinned it + your local backups.
3. **The key** — your wallet/passkey, plus recovery shares held by your org.

None of these require the developer to be online, reachable, or even in existence.

---

## Honest gaps & roadmap

We'd rather state these plainly than overclaim independence:

- **Pin endpoint is Pinata-shaped.** `PINATA_PIN` is currently a hard-coded constant.
  *Roadmap:* make the pin endpoint configurable (Pinata-compatible URL or IPFS Pinning
  Service API) so Options A/B are turnkey. Reads are already configurable.
- **No one-command self-host yet** for the full stack (app + your pinning). The app and
  identity layers are turnkey; storage requires the manual steps above.
- **Mirroring** (a second pinning service + multi-gateway fetch, with a vendor-neutral
  daily CID pointer) is planned — the durable answer to "don't depend on any single
  provider."

**Bottom line:** the app and your identity/keys are developer-independent **today**;
storage is independent in principle and via Options A–C, with the pin-endpoint config and
mirroring being the work that makes full self-hosting turnkey.
