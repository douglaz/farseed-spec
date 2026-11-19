# The local unlock passphrase is recommended, not required: an owner-directed amendment

Decided 2026-11-19 on the owner's instruction. The owner said the local unlock passphrase "should be optional",
with "perhaps we recommend 16 chars min, but accept anything including zero", and then, of the specification,
"I think it can be the official feature". This is an owner-directed amendment that loosens a security rule, not a
clarification. It reverses the credential format's "at least 16 Unicode code points".

## Why

The owner chose to make the passphrase optional for the operator. The recommendation stays, and the consequence of
a shorter or empty one is stated plainly before it is chosen, instead of being forbidden.

## Decision

- The operator chooses the local unlock passphrase's length. The page recommends 16 or more Unicode code points,
  kept in a password manager, and accepts any length, including none (`STA-23`; the credential format's local
  encryption envelope).
- Before a store is created, or a passphrase changed, with fewer than 16 Unicode code points, the page states in
  plain words what that length means. Such a store is created, or the passphrase so changed, only by the operator's
  explicit action while that statement is shown; no implicit submission does it (`STA-23`).
- The envelope, KDF and parameters are unchanged. An empty passphrase is the empty byte string fed to the same
  PBKDF2. WebCrypto's PBKDF2 accepted zero-length key material in Chrome 133 on an Android 16 emulator, checked on
  2026-11-19. It has not been checked on a physical phone; the first-stage phone's measurement ran in an Android
  WebView, Chrome 152 ([findings, 2026-09-11](../findings/2026-09-11-unlock-latency.md)). Until an empty-passphrase
  create passes on a physical Android Chrome (`STG-14`, `CNF-82`), that support is unverified there.
- The random data key still encrypts the store, and only its copy wrapped under the PBKDF2-derived key persists.
  Credentials allowed to persist stay encrypted in the local store, and memory-only credentials stay in memory. So
  `STG-10` and `CNF-15` still hold, and the weakened protection is declared, not hidden (`SEC-5` row 18, `STA-17`).

## What is loosened

- **No passphrase.** Anyone who can open the browser profile, or who holds a copy of its data, opens the store with
  one KDF run, using the salt stored beside it. That reaches:
  - the seed, and so every maintained machine, every relay pass, every future introduction and every declared
    handoff;
  - the inference account credential, a procured paid session key, every application inference key, and every
    other credential the inventory keeps in the local store.

  The vendor credential is not among them, because it stays in browser memory only (`SEC-5` row 1). Such a holder
  can also forge records that authenticate.
- **A short passphrase.** Such a copy can be guessed offline, and a successful guess gives the same access.
- **Lock, with no passphrase,** keeps out no one who can use the unlocked phone: Unlock is one press.
- **A lost or stolen phone** whose passphrase was empty is open at once to whoever reaches its browser data
  (`STA-17`).
- **A copied envelope** taken while the passphrase was empty or short stays open to that passphrase, even after a
  later change. A change rewraps the same data key, so whoever opens that copy can still read and forge the store's
  records after the change, until the store is re-encrypted under a fresh data key (credential format; `SEC-5`
  row 20).

## What stays

- Encoding without normalization or trimming; PBKDF2-HMAC-SHA-256 at 600,000 iterations with a fresh 16-byte
  salt; the envelope and its authentication.
- Wrong-passphrase, authentication and version failures still fail closed.
- The seed ceremony, and re-display only after the local passphrase is entered again. An empty passphrase is
  entered by the explicit unlock action with nothing typed.
- Lock and reload still require an unlock. Every other lifecycle rule of `STA-23` is unchanged.
- No plaintext or device-synced fallback.
- The recovery-sheet passphrase and its rules; this record does not touch them.

## Formal companion

No formal statement changes. `TauWeb.Authority` represents an unlock as an explicit event, and an unlock with an
empty passphrase is still one.

## Alternatives considered

| Option | Disposition |
|---|---|
| Keep the 16-code-point minimum | Not chosen by the owner. |
| A checkbox or a second press for an empty passphrase | Not chosen: one explicit action after the statement is the operator's choice (four readers of four). |
