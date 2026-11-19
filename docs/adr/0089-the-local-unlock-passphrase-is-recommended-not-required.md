# The local unlock passphrase is recommended, not required: an owner-directed amendment

Decided 2026-11-19 on the owner's instruction. The owner said the local unlock passphrase "should be optional",
with "perhaps we recommend 16 chars min, but accept anything including zero", and then, of the specification,
"I think it can be the official feature". This is an owner-directed amendment that loosens a security rule, not a
clarification. It reverses the credential format's "at least 16 Unicode code points".

## Why

The owner chose to make the passphrase optional for the operator. The recommendation stays, and the consequence of
a shorter or empty one is stated plainly before it is chosen, instead of being forbidden.

## Decision and owners

Each rule lives in its owning requirement; this record points at them.

- `STA-23` owns the passphrase's length being the operator's choice, the recommendation, the statement shown before
  a short or empty passphrase is set, the explicit action that sets it, the unlock with nothing typed, and the
  admission and Lock disclosure for an empty passphrase.
- The credential format's local encryption envelope owns the encoding (an empty passphrase is the empty byte
  string), the unchanged KDF and parameters, and the integrity limits when the passphrase is empty or guessed.
- `SEC-5` row 18 owns the at-rest consequence of an empty or short passphrase, and `STA-17` the lost or stolen
  phone's.
- `CNF-15` records why it still holds for such a store, and `CNF-82` exercises the creates, unlocks and disclosure.

## Evidence

WebCrypto's PBKDF2 accepted zero-length key material in Chrome 133 on an Android 16 emulator, checked on
2026-11-19. It has not been checked on a physical phone; the first-stage phone's measurement ran in an Android
WebView, Chrome 152 ([findings, 2026-09-11](../findings/2026-09-11-unlock-latency.md)). Until an empty-passphrase
create passes on a physical Android Chrome (`STG-14`, `CNF-82`), that support is unverified there.

## What is loosened

- The credential format's 16-code-point minimum is removed.
- An empty or short passphrase now leaves the local store as open as `SEC-5` row 18 and `STA-17` state. The vendor
  credential is unaffected, because it stays in browser memory only (`SEC-5` row 1).
- A passphrase change still rewraps the same data key, so a copy taken under an empty or short passphrase stays
  open to it (credential format).

## What stays

- The envelope, KDF, parameters and authentication; wrong-passphrase, authentication and version failures still
  fail closed (credential format, `STA-23`).
- The seed ceremony and its re-display, Lock, reload and every other lifecycle rule of `STA-23`.
- No plaintext or device-synced fallback (`STA-23`).
- The recovery-sheet passphrase and its rules; this record does not touch them.

## Formal companion

No formal statement changes. `TauWeb.Authority` represents an unlock as an explicit event, and an unlock with an
empty passphrase is still one.

## Alternatives considered

| Option | Disposition |
|---|---|
| Keep the 16-code-point minimum | Not chosen by the owner. |
| A checkbox or a second press for an empty passphrase | Not chosen: one explicit action after the statement is the operator's choice (four readers of four). |
