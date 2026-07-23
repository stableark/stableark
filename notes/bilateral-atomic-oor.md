# Why Ark should support bilateral atomic OOR

**Audience:** Ark protocol implementers (clients and operators)  
**Topic:** Bilateral atomic out-of-round spends as a general protocol primitive

---

## What we mean

Today a typical OOR / arkoor payment is one-directional:

```text
Alice VTXO  →  Bob payment + Alice change
```

Only one owner spends an input. The recipient only receives.

**Bilateral atomic OOR** would be:

```text
Alice VTXO + Bob VTXO
        →
new Alice VTXO + new Bob VTXO
```

Two active input owners, one OOR package, both old outputs consumed, both parties receive updated allocations. Atomic: the joint redistribution succeeds as a whole, or not at all.

This is not “sats must flow both ways at once.” It is **two inputs, two outputs, one joint agreement**.

---

## Why this is a protocol gap

Ark already has two strong primitives:

- **OOR** — fast value transfer between rounds
- **Rounds** — refresh, consolidate, renew expiry, restore stronger VTXO security

What is missing is a fast way for **two (or more) existing VTXO owners to rewrite a shared allocation** without waiting for a round and without reducing the update to a one-way payment.

That matters whenever the economic object is not “Alice pays Bob,” but “Alice and Bob jointly update who owns how much of a fixed pile of bitcoin.”

Examples that fall in that bucket:

1. **P2P derivatives / hedges** — after a price mark, both claims change together.
2. **Escrow and dual-funded contracts** — release, claw back, or rebalance without a round.
3. **Two-party redistribution / settlement nets** — collapse obligations into new balances in one step.
4. **LP ↔ user collateral updates** — resize both sides off-round when risk changes.
5. **App-level state channels on Ark** — advance shared state `n → n+1` by spending the prior outputs.

You can approximate some of these with sequential A→B (or B→A) sends. That moves value. It does **not** give you a clean, atomic replacement of a multi-party state.

---

## Comparison

| | Normal OOR (A→B) | Bilateral atomic OOR | Round only |
| --- | --- | --- | --- |
| Speed | Fast | Fast | Bound to operator schedule |
| Joint reallocation | Emulated; often fragments VTXOs | Native and clean | Native and clean |
| Clear latest state | App must track a bag of outpoints | Single successor state | Strong, but infrequent |
| Exit / recovery complexity | Grows with fragments and hops | One new allocation set | Best after refresh |
| Mutual authorization | Mostly app-layer | Encoded in the package | Encoded in the round |
| Time-sensitive rebalance | Depends on payment direction and partial success | Both sides updated together | Weak if rounds are delayed |

Rounds already solve consolidation well. They are the wrong **latency path** for frequent shared-state updates.  
OOR already solves latency. Today it is shaped for payments, not for **multi-owner state rewrites**.

The natural split:

```text
Frequent joint reallocation     →  bilateral atomic OOR
Expiry, compression, refresh    →  rounds
```

That is consistent with Ark’s existing design philosophy; it just completes it.

---

## Why this is good for Ark implementations

Framed for any client/operator stack — not for one product or one application:

1. **More expressive apps on unmodified operators**  
   Wallets and protocols can build contracts that feel like off-chain UTXO renegotiation, not only chat-style payments.

2. **Higher useful OOR volume**  
   Shared positions generate many small updates over their lifetime. That is recurring OOR demand for operators, not only one-shot payments.

3. **Less pressure to abuse rounds as an application clock**  
   If every joint update waits for a round, apps become fragile to round timing, availability, and fees. An off-round joint update keeps rounds focused on lifecycle work.

4. **Cleaner client recovery stories**  
   Atomic successor states are easier to reason about than “Alice paid, Bob didn’t consolidate, sequence is ambiguous, exit needs N VTXOs.”

5. **Stronger protocol positioning**  
   “Ark is programmable off-chain bitcoin” is more convincing if two users can renegotiate ownership of existing VTXOs atomically, not only send value one way.

6. **No need for app-specific operators**  
   The operator does not need to understand oracles, prices, or contract semantics. It only needs to co-sign a multi-owner OOR package. Apps can stay on public Ark servers.

7. **Fits the security model Ark already has**  
   OOR already involves operator co-sign and statechain-like assumptions. Bilateral OOR is the same family of mechanism with richer input ownership, not a new trust island.

---

## What this is not asking for

- Not replacing rounds
- Not putting business logic (prices, collateral rules, contract types) into the operator
- Not requiring a custom coordinator per application
- Not blocking apps that start with directional OOR + later round consolidation

Directional OOR remains the right default for payments. Bilateral atomic OOR is the right primitive for **shared-state updates**.

---

## Concrete implementation questions

Useful against any current stack:

1. Can the OOR transaction builder already accept **multiple checkpoint inputs controlled by different owner keys**?
2. Does operator-side OOR session / co-sign flow assume a single sender, or can it orchestrate multi-owner signing?
3. If not: what is the minimal RPC and state-machine change for an `N-input / M-output` package where two clients both sign?

Even a narrowly scoped 2-in / 2-out path would unlock a large class of applications.

---

## Bottom line

Ark already separates fast transfers (OOR) from lifecycle refresh (rounds).  
Bilateral atomic OOR is the missing third move: **fast joint reallocation between existing owners**.

That makes Ark more useful as a settlement fabric for contracts, not only as a payment rail — while keeping economics in clients and co-signing in the operator, where it already belongs.
