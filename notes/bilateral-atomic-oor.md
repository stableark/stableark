# Why Ark should support joint multi-input offchain updates

**Audience:** Ark-family protocol implementers (clients and operators)  
**Topic:** Atomic multi-owner offchain spends as a general protocol primitive  
**Also called:** “bilateral atomic OOR” in Second/Wavelength-era language

> **Status (July 2026):** This capability is **not uniformly missing**. See [implementation landscape](implementation-landscape.md). Arkade reports Bitcoin-style multi-input / multi-output Arkade transactions with distinct per-input signers. Second/Bark reports one-input OOR today, with multi-party options more natural in rounds or via future two-party VTXO policies. This note still explains *why* the primitive matters for any stack that lacks it.

---

## What we mean

A typical one-way offchain payment looks like:

```text
Alice VTXO  →  Bob payment + Alice change
```

Only one owner spends an input. The recipient only receives.

**Joint multi-input (bilateral atomic) update** means:

```text
Alice VTXO + Bob VTXO
        →
new Alice VTXO + new Bob VTXO
```

Two active input owners, one offchain package, both old outputs consumed, both parties receive updated allocations. Atomic: the joint redistribution succeeds as a whole, or not at all.

This is not “sats must flow both ways at once.” It is **two inputs, two outputs, one joint agreement**.

On stacks that already expose Bitcoin-like offchain transactions, this is simply a normal multi-input tx that is operator-co-signed and not mined. On stacks whose payment path is one-input OOR, it is an extension (or a round-time / policy workaround).

---

## Why this primitive matters

Ark-family systems typically provide:

- **Fast collaborative transfers** between users on the same operator
- **Batch lifecycle settlement** to renew expiry / compress trees / strengthen finality

What applications still need—when the stack does not already provide it—is a fast way for **two (or more) existing VTXO owners to rewrite a shared allocation** without waiting for batch settlement and without reducing the update to a one-way payment.

That matters whenever the economic object is not “Alice pays Bob,” but “Alice and Bob jointly update who owns how much of a fixed pile of bitcoin.”

Examples that fall in that bucket:

1. **P2P derivatives / hedges** — after a price mark, both claims change together.
2. **Escrow and dual-funded contracts** — release, claw back, or rebalance without a batch.
3. **Two-party redistribution / settlement nets** — collapse obligations into new balances in one step.
4. **LP ↔ user collateral updates** — resize both sides when risk changes.
5. **App-level state channels on VTXOs** — advance shared state `n → n+1` by spending the prior outputs.

You can approximate some of these with sequential A→B (or B→A) sends. That moves value. It does **not** give you a clean, atomic replacement of a multi-party state.

---

## Comparison

| | One-way offchain payment | Joint multi-input update | Batch settlement only |
| --- | --- | --- | --- |
| Speed | Fast | Fast | Bound to operator schedule |
| Joint reallocation | Emulated; often fragments VTXOs | Native and clean | Native and clean |
| Clear latest state | App must track a bag of outpoints | Single successor state | Strong, but infrequent |
| Exit / recovery complexity | Grows with fragments and hops | One new allocation set | Best after renewal |
| Mutual authorization | Mostly app-layer | Encoded in the package | Encoded in the batch |
| Time-sensitive rebalance | Depends on payment direction and partial success | Both sides updated together | Weak if batches are delayed |

Batch settlement already solves consolidation well. It is the wrong **latency path** for frequent shared-state updates.  
Fast transfers already solve latency. If they are shaped only as one-way payments, they are incomplete for **multi-owner state rewrites**.

The natural split:

```text
Frequent joint reallocation     →  multi-input collaborative offchain tx
Expiry, compression, renewal    →  batch settlement
```

---

## Why this is good for Ark-family implementations

Framed for any client/operator stack — not for one product or one application:

1. **More expressive apps on unmodified operators**  
   Wallets and protocols can build contracts that feel like off-chain UTXO renegotiation, not only chat-style payments.

2. **Higher useful transfer volume**  
   Shared positions generate many small updates over their lifetime. That is recurring collaborative-tx demand for operators, not only one-shot payments.

3. **Less pressure to abuse batch settlement as an application clock**  
   If every joint update waits for a batch, apps become fragile to timing, availability, and fees.

4. **Cleaner client recovery stories**  
   Atomic successor states are easier to reason about than fragmented payment trails.

5. **Stronger protocol positioning**  
   “Programmable offchain bitcoin” is more convincing if two users can renegotiate ownership of existing VTXOs atomically.

6. **No need for app-specific operators**  
   The operator does not need to understand oracles, prices, or contract semantics. It only needs to co-sign a multi-owner package. Apps can stay on public servers.

7. **Fits existing collaborative security models**  
   Operator co-sign and statechain-like assumptions already exist for collaborative spends. Multi-owner inputs are richer ownership, not a new trust island.

---

## What this is not asking for

- Not replacing batch settlement / renewal
- Not putting business logic (prices, collateral rules, contract types) into the operator
- Not requiring a custom coordinator per application
- Not blocking apps that start with directional payments + later consolidation

One-way payments remain the right default for payments. Joint multi-input updates are the right primitive for **shared-state updates**.

---

## Concrete implementation questions

Useful against any current stack:

1. Can the offchain transaction builder accept **multiple inputs controlled by different owner keys**?
2. Does operator-side session / co-sign flow assume a single sender, or can it orchestrate multi-owner signing?
3. If not: what is the minimal RPC and state-machine change for an `N-input / M-output` package where two clients both sign?

Even a narrowly scoped 2-in / 2-out path unlocks a large class of applications.

---

## Bottom line

Ark-family systems already separate fast collaborative transfers from lifecycle renewal.  
Joint multi-input offchain updates complete that design: **fast joint reallocation between existing owners**.

That makes the stack more useful as a settlement fabric for contracts, not only as a payment rail — while keeping economics in clients and co-signing in the operator, where it already belongs.

For who already supports this and who does not, see [implementation landscape](implementation-landscape.md).
