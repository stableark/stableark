# Implementation landscape (July 2026)

How current Ark-family stacks relate to Stable Ark’s need for **joint multi-owner offchain reallocation**:

```text
Alice VTXO + Bob VTXO  →  new Alice VTXO + new Bob VTXO
```

This note records implementer guidance and public docs. It is not an official statement by any vendor.

For a broader product/architecture comparison (not only Stable Ark), see [stack-comparison.md](stack-comparison.md).

---

## Summary

| Stack | Fast collaborative transfers | Joint multi-owner inputs in one offchain tx | Lifecycle renewal |
| --- | --- | --- | --- |
| **Arkade** (Ark Labs) | Arkade transactions (UTXO-style) | **Yes** — distinct keys can sign distinct inputs; operator co-signs; not mined | Intents + batch swaps (not “send in round”) |
| **Second / Bark** | OOR / arkoor | **No today** — OOR is one-input structurally; multi-party more natural in a round; two-party VTXO policies possible but need added support | Periodic rounds for refresh |
| **Wavelength** (Lightning Labs) | Durable OOR client FSMs against a **Wavelength-compatible** Ark gateway | Not a documented multi-owner product path; strong packaging/recovery | Rounds + OOR; public presets use LL-hosted gateways on signet/testnet |

**Stable Ark primary PoC target: Arkade.**

---

## Arkade

Sources: [docs.arkadeos.com](https://docs.arkadeos.com), [contracts deep dive](https://docs.arkadeos.com/contracts/deep-dive), [demos](https://github.com/arkade-os/demos), implementer conversation (July 2026).

### Model

- Offchain execution uses a **Bitcoin-like UTXO model**: transactions have inputs and outputs.
- Collaborative path is co-signed by the operator and not broadcast to miners.
- Unilateral path remains available via timelocked exit scripts.
- Docs show `buildOffchainTx(inputs, outputs, …)` and a submit/finalize flow.
- Terminology: prefer **Arkade transaction**, **batch swap**, **intent**, **operator** — not “OOR/arkoor/round” from older Ark explainers.

### Joint reallocation

Implementer confirmation (July 2026):

- Input 0 can be spent by Alice’s key and input 1 by Bob’s key in the **same** Arkade transaction.
- “It’s like Bitcoin transactions” — unbroadcast, co-signed by Arkade.
- Cited as how integrations such as Boltz or HodlHodl-style flows can work.
- Future opcodes may enable richer on-script contracts (e.g. Liquid/Fuji-style patterns); **not required** for Stable Ark v1 rebalancing.

### Implications for Stable Ark

- Preferred shape (atomic joint replace) maps directly.
- No custom operator required for the P2P case.
- Next work: two-wallet signing coordination on top of the public SDK.

---

## Second / Bark

Sources: Second docs on VTXOs/rounds/arkoor; implementer conversation (July 2026).

### Model

- Instant transfers between rounds use **OOR / arkoor** (one party spends, recipient receives a spend VTXO).
- **Rounds** refresh VTXOs, renew expiry, and restore stronger refresh-VTXO security.
- Payments are not done “in round” as the primary path in current Bark design; rounds are for refresh.

### Joint reallocation

Implementer confirmation (July 2026):

- OOR payments are **one-input structurally** today.
- Doing a multi-party reallocation **in a round** is considered possible.
- **Two-party VTXO policies** could support related constructions but would require adding support.
- **Nested MuSig2** (as in Stutxo) was noted as a way to implement two-party control while keeping structure less visible to the server.

### Implications for Stable Ark

- v1 on Bark would likely use **directional OOR marks** (whoever owes pays) + round consolidation, or wait for multi-party primitives.
- Not the first PoC target for atomic joint replace.
- Remains important for Rust/Bark ecosystem reach once an adapter exists.

---

### Wavelength (Lightning Labs)

Sources: [wavelength](https://github.com/lightninglabs/wavelength) architecture (`oor`, `lib/tx/oor`, checkpoints, fraud, unroll); [system architecture](https://wavelength.lightning.engineering/introduction/system-architecture/).

### Model

- Embedded/self-custodial Ark **client** daemon with durable OOR session FSMs, checkpoint packages, round participation, fraud monitoring, unilateral unroll.
- Connects to three backends: Ark operator+mailbox (`arkServerAddress`), swap server, Esplora. Public signet/testnet presets use Lightning Labs–hosted gateways. There is **no documented** compatibility with Arkade `arkd` or Bark’s operator.
- Explicit separation of OOR vs rounds at the client layer.
- `BuildArkPSBT` accepts multiple checkpoint inputs and multiple recipients at the **transaction builder** level; whether production flows expose multi-**owner** signing as a first-class product path still needs evaluation.

### Implications for Stable Ark

- Excellent reference for durable session design even if Arkade is the first runtime.
- Revisit only against a Wavelength-compatible gateway—not by pointing `waved` at `arkd`.

---

## Terminology map

| Concept | Older / Second-style language | Arkade language |
| --- | --- | --- |
| Fast offchain transfer | OOR, arkoor | Arkade transaction |
| Batch lifecycle | Round | Batch swap (via intents) |
| Operator | ASP / Ark server | Operator / Arkade Service |
| Joint multi-in/out update | “Bilateral atomic OOR” | Multi-input Arkade transaction |

Stable Ark docs prefer **joint offchain reallocation** when speaking generically.

---

## Decision

1. Prototype on **Arkade** with two wallets and a fake oracle.
2. Keep an `ArkBackend` interface so Bark/Wavelength adapters can follow.
3. Do not assume one Ark explainer applies to all implementations — verify primitives per stack.
