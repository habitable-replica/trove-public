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

## Repository layout

```
trove/
├── public/              ← the app (served as static files)
│   ├── index.html       ← main app (owner / accountant)
│   ├── staff.html       ← staff expenses & invoices app (passkey-only)
│   └── _headers         ← security headers
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
