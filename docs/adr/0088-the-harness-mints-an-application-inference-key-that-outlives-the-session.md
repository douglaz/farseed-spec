# The harness mints an application inference key that outlives the session: an owner-directed amendment

Decided 2026-11-16 on the owner's instruction, while the glossary's terms were being settled. In a
live test an operator's application needed a model of its own, and the only way the specification
gave it a key was for the operator to obtain one at a service and enter it on `place_secret`'s
card. The owner said "The model should ask the key", "The key may be crafted on demand by the
harness", and then "the user shouldn't give this key. this key should be controlled by the
harness (our app)". Asked how long the key lives, the owner answered "agree, outlive the
session", with a cap of 0.25 USD and an expiry of 30 days. Asked how the operator approves it, the
owner answered "yes, one press to create key": a card with "Create key" and "Don't".

This is an owner-directed amendment that loosens a security rule, not a clarification. It reverses
[ADR-0039](./0039-installed-agents-are-the-operators-applications.md)'s rejection of "Give the
application a capped key minted from the harness's inference account", and the sentence of
`ARC-43` that carried it, "The harness mints nothing for an application". That record's other
decisions stand.

## Why

ADR-0039 rejected the minted key because it would be "a harness credential on a machine, which
this record exists to keep impossible". Its reason for keeping harness credentials off machines
is `ARC-1`'s: a machine that can call the harness's model unprompted is the harness acting
unprompted. A key the harness mints for an application at the operator's press does not do that.
The application's calls with it are its own; they reach no session, nothing they return starts
work in the harness, harness code never uses the key for inference, and the model has no tool
that does, although a model with root on that machine can read it (below).
What remains is a cost to the operator's balance and a credential on a machine, both bounded and
both stated below. Against that, the operator no longer has to go to a service, buy a key and
paste it, when the balance the harness already uses could fund the application.

## Decision and owner

`ARC-1b` owns the rule, and `SEC-5` row 24 is the key's row. `ARC-43` lists the job,
`place_inference_key`, and points at `ARC-1b`; `ARC-31a`'s account credential mints and revokes
the key; `CNF-118` tests it.

- **The model asks; the operator presses once.** The card offers "Create key" and "Don't", and
  "Don't" mints nothing, sends nothing to the aggregator and spends nothing.
- **The cap is 0.25 US dollars, the expiry 30 days.** `ARC-31a` says "The spending limit
  MUST be expressed in the issuer account's own spend-accounting unit", and it infers no currency
  conversion, so a key is minted only where the pinned profile names the US dollar as that unit,
  at 0.25 in it; under any other unit the card offers only "Don't". The operator sets neither; the
  session key's 1 to 1440 minute range does not apply.
- **It is placed as an application secret is.** The key travels only as `place_secret`'s value,
  under its standard-input, custody, scan-reference and listing rules, never in model context.
  The issuer's identifier for it, with its cap and expiry, is journaled with the placement in the
  machine's state and carried by the recovery sheet (`STA-16`, credential format payload version
  9), so a Restore keeps the operator's Revoke and the revocation at destruction.
- **It outlives the session.** `ARC-1b` says "It is not a key the session minted". End, expiry,
  cap exhaustion and an explicit app Lock revoke nothing of it.
- **Three things end it:** its machine's destruction, its expiry, and the operator's revocation
  from the trust display. The display lists every key that is neither confirmed revoked nor
  expired, whatever its placement's state, so a key that outlives its placement can still be
  revoked.

## What is loosened

- `ARC-1`: a machine may hold an inference key minted from the operator's inference account.
  `ARC-1` now says why that key is not a credential belonging to the harness.
- `ARC-43`: "The harness mints nothing for an application" is reversed.
- `SEC-5` row 14 mints and revokes row 24 as well as row 2.
- The recovery sheet gains payload version 9, carrying each live key's identifier, cap, expiry
  and placement (never the key).
- For this key, the rotation card and the trust display no longer say that revoking the old
  value is "the operator's act, which the harness does not verify": the harness revokes it on
  the operator's Revoke, and the rotation itself still revokes nothing.
- A key minted at a session's request lives up to 30 days after that session, on a machine where
  anything with root can read it, a later session's model included. Through it, that model can
  spend up to the key's cap on calls the harness neither makes nor counts.
- The application, and anything with root on its machine, can read the account's usage history,
  as a session key can (`ARC-31a`): when the harness's sessions ran, which models they used and
  what they cost.
- Each key adds to what a leak can spend: the lesser of the remaining balance and its cap, per
  key, under the conditions `ARC-31a` sets on a session key's bound.

## Choices made in drafting

The owner's words left these open; each follows from an existing rule or is the stricter
reading, and each is open to review.

- **Not on a multi-tenant machine, for now.** `ARC-37` says "A multi-tenant machine MUST NOT hold
  spendable key material", and the key spends inference credit. Whether a capped inference key
  with no funding authority is the spendable key material that rule bars is not decided here:
  narrowing a blocking rule is the owner's call. Until then the strict reading stands, and a
  request on such a machine is refused as `ARC-36a` refuses a request needing a less restrictive
  class. Because an ad-hoc machine is multi-tenant by default (`ARC-36a`), the key is available
  only where the operator answered single-purpose before binding. This is the open question of
  this record.
- **The procured path only.** Without an inference account credential there is nothing to mint
  from.
- **Revocation at destruction is requested at the confirmed destroy**, at LNVPS the confirmed stop
  that stands in for one (`ARC-22`), or at the deletion evidence `SEC-5` admits if that comes
  first. Waiting for the vendor's deletion confirmation, which LNVPS may never give, would leave
  the key live; revoking earlier can only shorten its life.
- **A rotation revokes nothing.** The owner named three ends, and a rotation is not one of them;
  the old key ends by one of the three.
- **Revocation ends the key, not its placement.** The placement's scan reference and listing end
  as `ARC-43` and `SEC-5` say, so a revoked key's bytes stay protected in later output.
- **The terminal does not seal the key.** A session key's mint terminal keeps the key in the
  local store; this key's terminal keeps only its identifier and the issuer's accepted facts, so
  the key lives nowhere but in transit and on its machine.

## Formal companion

None. `ARC-1b` is Markdown only, like `place_secret`'s rules, and no declaration in
`tools/formal/` changes.

## Alternatives considered

| Option | Disposition |
|---|---|
| The operator obtains a key at a service and enters it on `place_secret`'s card (the previous rule, ADR-0039) | Kept for every key the harness does not mint, but no longer the only way an application gets an inference key. The owner: "the user shouldn't give this key". |
| Hand the application the session's own inference key | Still rejected, as ADR-0039 rejected it: that key is the session's and is revoked at its End. |
| Revoke the application's key when the session ends | Not chosen by the owner, who chose "agree, outlive the session". The application keeps running after the session; a key revoked at End would stop it. |
| Mint with no approval press | Not chosen by the owner, who chose one press, "Create key" or "Don't". |
