# Trove

**Encrypted, decentralised accounting for organisations where privacy is protection.**

Trove is real double-entry bookkeeping — journals, accounts, invoices, reports —
built so your data is encrypted on your device before it ever leaves. The
encryption key never reaches anyone else: not even the people who operate Trove
can read your books. Privacy by architecture, not by policy.

---

## How it works

- **Client-side encryption.** All data is encrypted in the browser with
  AES-256-GCM. Only the encrypted blob ever leaves the device.
- **You hold the key.** Identity is a MetaMask wallet or a passkey, which
  derives (and wraps) the vault key. No password database, no server-side key.
- **Decentralised storage.** Encrypted blobs are content-addressed and pinned to
  IPFS, so the data isn't bound to any single server.
- **No build step.** The app is plain static HTML/JS — no framework, no bundler.

---

## Independence & self-hosting

The app and your identity/keys are **developer-independent today**; storage is
independent in principle (via your own IPFS node or pinning account). The full,
honest map — including current gaps and the roadmap to turnkey self-hosting — is
in **[`docs/SELF_HOSTING.md`](docs/SELF_HOSTING.md)**.

In short: the operator can disappear and you keep (1) the code — this repo, a
single offline HTML file; (2) the encrypted vault — wherever *you* pinned it, plus
your local backups; and (3) the key — your wallet/passkey and recovery shares.

---

## Zero-knowledge proofs — status & honest caveat

Trove uses **Groth16** (via snarkjs) to prove a batch of transactions balances
without revealing the amounts. Groth16 requires a **per-circuit trusted setup**;
whoever ran it and kept the secret randomness ("toxic waste") could, in
principle, forge proofs that verify as valid. This is an **integrity** concern
(forgeability) — **not** a confidentiality one. It cannot expose anyone's data.

**Current state (being transparent):** the proving/verifying keys are shipped,
but the circuit source and a multi-party ceremony transcript are **not yet
published**. We are (1) publishing the circuit source + the exact setup commands,
and (2) evaluating a small multi-party setup ceremony and/or a transparent proof
system (PLONK / STARK) so the auditability guarantee does not rest on a single
party. Tracked as a roadmap item.

---

## Repository layout

```
trove/
├── public/              ← the app (served as static files)
│   ├── index.html       ← main app (owner / accountant)
│   ├── staff.html       ← staff expenses & invoices app (passkey-only)
│   └── _headers         ← security headers
├── docs/
│   └── SELF_HOSTING.md  ← run Trove independently of the developer
└── wrangler.jsonc       ← Cloudflare deploy config (serves ./public)
```

---

## Running locally

It's a static site — serve the `public/` directory with any static file server:

```bash
npx serve public
# or
python3 -m http.server --directory public 8000
```

Then open the printed URL in your browser.

---

## Deploying

The app serves `public/` as static assets on Cloudflare. With the
[Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/):

```bash
npx wrangler deploy
```

---

## Security

No secrets live in this repository. Pinata storage credentials and wallet keys
are entered at runtime and stay on the user's device — they are never committed.
If you find a security issue, please open an issue or contact the maintainers
before disclosing publicly.

---

## License

Licensed under the **GNU Affero General Public License v3.0** — see
[`LICENSE`](LICENSE). If you run a modified version as a network service, the
AGPL requires you to make your modified source available to its users.
