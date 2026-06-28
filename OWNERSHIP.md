# Ownership

This project's ownership is asserted with a **privacy-preserving cryptographic
commitment**. The owner's legal identity is *not* published here, but can be
proven at any later time, at the owner's discretion.

## Commitment

Each owner is represented by a SHA-256 commitment:

```
commitment = SHA-256( full_legal_name + "|" + secret_salt )
```

The `full_legal_name` and a high-entropy `secret_salt` are held privately by the
owner. Only the resulting hash is published below. The salt makes the commitment
**hiding** (the name cannot be recovered or guessed-and-confirmed from the hash)
while keeping it **binding** (no other name can reproduce the same hash).

| # | Owner commitment (SHA-256) |
|---|----------------------------|
| 1 | `80ebc4eb59228e6babab733393fcc8d9c160e589f0f705ee59206ac57ee3b9c8` |
| 2 | `5bd1d050f47d527277c28616fa5abb7fd384bd9a5279c0bbdf114b4530eff554` |
| 3 | `afd51f8d2960217413fe3f37efdc076e82753849ea11383b3088060d047fa185` |

The publication date is fixed by this file's first commit in the Git history.

## How to verify a claim of ownership

When an owner chooses to prove ownership, they disclose their `full_legal_name`
and `secret_salt`. Anyone can then verify the disclosed values reproduce the
published commitment:

```bash
printf '%s|%s' "<FULL_LEGAL_NAME>" "<SECRET_SALT>" | shasum -a 256
```

If the output matches a commitment in the table above, the claim is valid. Since
the commitment has been public (and timestamped by Git) since the date of first
commit, a matching disclosure proves the claimant committed to that identity at
that time.
