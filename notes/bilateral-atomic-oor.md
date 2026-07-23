# Why Ark should support bilateral atomic OOR

**To:** Second / Bark team  
**From:** Stable Ark  
**Re:** Bilateral atomic out-of-round spends as a protocol primitive  
**Links:** https://stableark.org · https://github.com/stableark/stableark

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

Two active input owners, one OOR package, both old outputs consumed, both parties receive updated allocations. Atomic: the shared state advances as a whole, or not at all.

This is not “sats must flow both ways at once.” It is **two inputs, two outputs, one joint agreement**.

---

## Why it matters (beyond Stable Ark)

This is a missing primitive for any **shared off-chain contract** on VTXOs:

1. **P2P derivatives / hedges** (Stable Channels → Stable Ark): after an oracle price update, both claims change together.
2. **Escrow / dual-funded deals**: release or rebalance without waiting for a round.
3. **Two-party redistribution**: reallocate and consolidate without a round.
4. **LP ↔ user collateral adjustments** off-round.
5. **Lightweight state updates** between two users on the same Ark server: advance `n → n+1` by spending the prior state.

You can approximate many flows with a normal send (A→B or B→A). That moves value. It is **not** the same as atomically replacing a shared position state.

---

## Comparison

| | Normal OOR (A→B) | Bilateral atomic OOR | Round only |
| --- | --- | --- | --- |
| Speed | Fast | Fast | Slow (operator schedule) |
| Updates both claims cleanly | Partial / fragments VTXOs | Yes | Yes |
| Clear “latest state” | Bag of VTXOs + app logic | Single state `n+1` | Good, but slow/costly |
| Exit path | Grows with fragments | One clean new pair | Best (refresh VTXOs) |
| Mutual consent | App-layer only | In the tx package + app | In the round |
| Urgent rebalance / liquidation | Depends on who owes | Both sides rewritten now | Bad if the round is delayed |

Rounds already consolidate well — but they are the wrong **clock** for frequent mark-to-market.  
OOR is already fast — but today it optimizes 1→1 payments, not **two-party state rewrites**.

Intended split (fits Ark’s own design):

```text
Frequent shared-state updates  →  bilateral atomic OOR
Expiry / tree compression / stronger refresh security  →  rounds
```

---

## Why Second / Bark should care

- Matches Bark’s model: **OOR for frequent movement, rounds for refresh**.
- Bilateral OOR is the “channel update” missing between two users on the same server, without pretending a one-way payment is a state transition.
- Unlocks financial apps on your stack **without a custom operator**, if multi-owner multi-input OOR is available (or with a small RPC/FSM extension).
- Makes “Ark is programmable off-chain UTXOs” more concrete than send/receive alone.
- Recurring small updates (hedges, stables, escrow) are natural OOR volume for an ASP.

Important: the operator does **not** need to understand USD, oracles, or “stable vs leveraged.” Clients enforce economics. The operator only co-signs a multi-input OOR package the way it co-signs any other OOR.

Two users running app software can connect to a **normal** Ark server. No app-specific coordinator required for the P2P case.

---

## What we are *not* asking for

- Not replacing rounds.
- Not requiring operator-side oracle or collateral validation.
- Not a Stable Ark–specific opcode or policy in captaind.
- Not blocking a PoC that uses normal directional OOR + later round consolidation (that is a fine v1).

Bilateral atomic OOR is the **right primitive** for shared state. Without it, every financial protocol on Ark reinvents fragile app-level consensus and accumulates fragmented VTXOs until the next round.

---

## Concrete questions for Bark / captaind

1. Can the OOR builder already take **multiple checkpoint inputs controlled by different keys**?
2. Does server co-sign / session flow assume a single sender, or can it handle multi-owner signing?
3. If not: what is the minimal RPC/FSM delta for an `N-input / M-output` package with two clients signing?

---

## Context: Stable Ark

Stable Ark explores self-custodial USD-indexed bitcoin balances on Ark (synthetic dollars via rebalancing, no issued token), inspired by Stable Channels on Lightning.

Design note: https://github.com/stableark/stableark/blob/main/DESIGN.md  
Project: https://stableark.org

Happy to discuss constraints on the Bark/captaind side and adapt the client design to what you already support.
