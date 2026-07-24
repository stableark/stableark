# Second (Bark) vs Arkade vs Wavelength

A practical comparison of three Ark-family efforts as of July 2026.

This is an independent research note for builders. It is **not** an official statement by Second, Ark Labs, or Lightning Labs. Implementations move quickly; verify against current docs before shipping.

| Project | Org | What it primarily is |
| --- | --- | --- |
| **Bark** | [Second](https://second.tech) | Full Ark **implementation**: client SDK + wallet daemon + operator (`captaind`) |
| **Arkade** | [Ark Labs](https://arklabs.xyz) | Full **Arkade** stack (`arkd` + SDKs): Ark-family implementation focused on programmable offchain execution, intents/batch swaps, contracts/assets |
| **Wavelength** | [Lightning Labs](https://lightning.engineering) | Embedded Ark **client** daemon/SDK (`waved`) that talks to a **Wavelength-compatible Ark gateway**, plus Lightning↔Ark swaps |

They share ancestry in the “VTXO / shared UTXO / operator-coordinated offchain bitcoin” design space, but they currently expose **distinct client/operator APIs and SDKs** and should **not** be treated as wire-compatible. There is no published compatibility layer that lets a client from one ecosystem speak to another’s operator. Transaction templates and serialization also diverge ([Second community RFC](https://community.second.tech/t/rfc-standardizing-virtual-transaction-templates-for-cross-implementation-interoperability/181)).

Treat them as sibling ecosystems, not one shared wire protocol with three skins.

**Precision note:** “Distinct APIs / no published cross-compat” is well evidenced (see [Interoperability](#interoperability)). That is *not* the same as proving every byte of every underlying operator RPC is unrelated—avoid claiming a wholly separate invented protocol unless you have done a full protobuf diff. What matters for builders is: **you cannot assume `waved` ↔ `arkd` or Bark ↔ Arkade works.**

---

## One-line positioning

| | Positioning |
| --- | --- |
| **Bark / Second** | Make self-custodial bitcoin payments (Ark + Lightning + onchain) easy to embed in apps — “no channels, no liquidity management.” |
| **Arkade / Ark Labs** | Arkade implementation focused on **programmable offchain execution**: UTXO-style Arkade txs, intents/batch swaps, contracts/assets roadmap (“programmable money”). |
| **Wavelength / Lightning Labs** | Production-minded **Ark client toolkit** for apps/agents: embedded wallet daemon + optional Lightning swap engine, driven by a high-level SDK/CLI rather than raw operator protos. |

---

## Architecture roles

```text
                    ┌──────────────┐
                    │   App / UX   │
                    └──────┬───────┘
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
     Bark SDK/barkd   Arkade SDKs    Wavelength waved
           │               │               │
           ▼               ▼               ▼
      Second operator   Arkade operator   Wavelength-compatible
        (captaind)         (arkd)         Ark gateway + swap + Esplora
           │               │               │
           └───────────────┴───────────────┘
                           ▼
                      Bitcoin chain
```

| | Client | Operator / backends | Notes |
| --- | --- | --- | --- |
| **Bark** | Bark SDK, barkd, apps (Noah, etc.) | Second runs / ships operator side | End-to-end product from one org |
| **Arkade** | TS / Go / Rust / C# SDKs | `arkd` (+ signer/TEE model in docs) | End-to-end Arkade product; `arkd` describes itself as an Arkade server that builds on Ark ideas |
| **Wavelength** | Embedded `waved` (native/WASM) + SDK/CLI | Configures **three** gateways: Ark operator+mailbox (`arkServerAddress`), swap server, Esplora | Public `defaultConfig('signet'/'testnet')` presets point at Lightning Labs–hosted test infrastructure ([system architecture](https://wavelength.lightning.engineering/introduction/system-architecture/), [networks & config](https://wavelength.lightning.engineering/concepts/networks-and-config/)) |

Wavelength is still an **Ark client** (rounds, VTXOs, OOR, exits). It packages that behind an embedded daemon and a high-level app API. Do **not** read “connects to an operator” as “works with any Arkade/Bark operator”—there is no documented cross-stack gateway compatibility.

See [ARCHITECTURE.md](https://github.com/lightninglabs/wavelength/blob/main/ARCHITECTURE.md) for the Go daemon map.

---

## Mental model: payments vs execution

| | Dominant mental model |
| --- | --- |
| **Bark** | Payment rail: receive VTXOs, pay Ark/Lightning/onchain, refresh before expiry |
| **Arkade** | Offchain **Bitcoin UTXO machine**: craft multi-in/out Arkade txs, preconfirm via cosign, settle via intents/batch swaps |
| **Wavelength** | Hardened wallet client: board → VTXO lifecycle → OOR/in-round send → refresh/leave/exit, with crash-safe FSMs |

Arkade docs deliberately reject older “OOR / round / ASP” vocabulary for their system ([glossary](https://docs.arkadeos.com/glossary)). Bark/Wavelength still use OOR + rounds language aligned with classic Ark explainers ([ark-protocol.org OOR](https://ark-protocol.org/intro/oor/index.html), Second VTXO/round docs).

---

## Terminology cheat sheet

| Concept | Bark / Wavelength-style | Arkade-style |
| --- | --- | --- |
| Fast offchain transfer | OOR / arkoor | Arkade transaction |
| Batch lifecycle | Round | Batch swap (via **intents**) |
| Operator | ASP / Ark server | Operator / Arkade Service |
| Onchain anchor | Round / commitment tx | Commitment transaction |
| Renew VTXO lifetime | Refresh | Renewal |
| Exit without cooperation | Unilateral exit / unroll | Unilateral exit |

If you mix terms across docs, you will argue past people. Translate first.

---

## How value moves

### Fast path (between settlements)

| | Mechanism | Shape |
| --- | --- | --- |
| **Bark** | OOR / arkoor | Typically **one-input** payment: spend VTXO → recipient (+ change). Implementer guidance (July 2026): OOR is one-input structurally. |
| **Arkade** | Arkade transaction | **UTXO-like**: multiple inputs and outputs; submit/finalize with operator cosign; not mined in the collaborative path ([contracts deep dive](https://docs.arkadeos.com/contracts/deep-dive)). Implementer guidance: different owners can sign different inputs in one tx. |
| **Wavelength** | `ark send oor` (+ durable `oor` FSM) | Product surface is payment-style OOR; builder layer can assemble multi-checkpoint packages, but multi-**owner** bilateral flow is not the advertised path. |

### Settlement / renewal path

| | Mechanism | Cadence intuition |
| --- | --- | --- |
| **Bark** | Rounds: forfeit old VTXOs, receive refresh VTXOs | Periodic / demand-driven refresh; required before expiry |
| **Arkade** | BIP322 **intents** aggregated into **batch swaps** | User (or delegate) can seek settlement when online/economic — not “everyone sends inside a round” |
| **Wavelength** | Round participation FSM (`round` package) + refresh/leave | Client joins operator rounds; rich fee preview / consent gating in CLI |

Arkade’s public stance: early “send only in rounds” explanations were misleading for their system; collaborative transfers are Arkade txs, batches are for settlement/finality compression.

---

## Multi-party and contracts

| Capability | Bark | Arkade | Wavelength |
| --- | --- | --- | --- |
| One-way P2P offchain pay | Yes (OOR) | Yes (Arkade tx) | Yes (OOR) |
| Multi-input / multi-output offchain tx | Limited in OOR (one-input today) | First-class UTXO model | Builder-capable; product path is send-oriented |
| Distinct owners signing distinct inputs in one collaborative tx | Not OOR today; round / two-party policies / nested MuSig2 discussed as paths | Yes (implementer + UTXO model) | Not documented as a product feature |
| Script / policy surface | VTXO policies; two-party policies possible with added support | Bitcoin Script in VTXOs; Arkade Script / assets / richer opcodes on roadmap | `arkscript` policy compiler in-tree |
| Issued assets | Not the Bark headline | **Arkade Assets** | Taproot Assets path discussed in Wavelength roadmap / PoCs |

For **shared-state apps** (escrow, dual-funded deals, synthetic hedges), Arkade’s model is currently the most direct. Bark can still do directional marks + round cleanup, or invest in multi-party primitives. Wavelength is strongest as a durable client if the operator/protocol under it exposes the needed tx shape.

---

## Lightning

| | Lightning story |
| --- | --- |
| **Bark** | First-class: pay/receive Lightning from Ark balance via server gateway; major product promise for app builders |
| **Arkade** | Integrations exist / documented as integration surface (e.g. Lightning bridges); core pitch is execution + VTXOs more than “LN without channels” |
| **Wavelength** | Optional **swapruntime**: Lightning ↔ Ark atomic swaps behind `send`/`recv --offchain`; LND can be a wallet backend |

If your app’s #1 job is “pay any Lightning invoice from a simple self-custodial balance,” Bark and Wavelength are aimed squarely at that UX. If your #1 job is “run bitcoin-like offchain contracts,” Arkade is aimed at that.

---

## Developer experience

| | Languages / entry points | Docs / demos |
| --- | --- | --- |
| **Bark** | Rust core + bindings (RN, Flutter, Swift, Kotlin, Go, WASM, …), barkd REST | [second.tech docs](https://second.tech/docs/learn/intro), Bark SDK guides |
| **Arkade** | Native SDKs: TypeScript, Go, Rust, C# (no bindings framing) | [docs.arkadeos.com](https://docs.arkadeos.com), [arkade-os/demos](https://github.com/arkade-os/demos) |
| **Wavelength** | Go daemon + Go SDKs; WASM/mobile variants; MCP for agents | [README](https://github.com/lightninglabs/wavelength), [ARCHITECTURE.md](https://github.com/lightninglabs/wavelength/blob/main/ARCHITECTURE.md), daemon CLI guide |

### Operational client quality

Wavelength stands out for **crash-safe actors**, durable OOR sessions, fraud monitoring, unilateral exit planning, and agent-friendly CLI contracts (structured errors, dry-run, consent gates). Even if you build on another stack, its client engineering is a useful reference.

Bark stands out for **embedding payments in consumer apps** with a wide binding matrix and Lightning included.

Arkade stands out for **protocol expressiveness** (UTXO txs, intents, contracts/assets direction) and multi-language native SDKs.

---

## Trust and security themes (shared + different accents)

Common across the family (details vary):

- User keeps keys; collaborative spends need operator cosign.
- Unilateral exit exists but can be slower/more expensive than collaborative paths.
- VTXOs / batches expire; users (or delegates) must renew or risk operator sweep policy.
- Collaborative offchain history can carry statechain-like trust assumptions along ancestor chains.

Different accents:

| | Accent |
| --- | --- |
| **Bark** | Classic clArk/Ark payment security story: refresh VTXOs vs spend VTXOs, round atomicity via forfeits |
| **Arkade** | Virtual mempool / DAG, preconfirmation, signer isolation (TEE / attestation in docs), intent delegation for offline renewal |
| **Wavelength** | Client-side fraud watch + unroll machinery; mailbox transport to operator; package relay / CPFP discipline |

---

## Maturity and surface area (rough)

| | Rough read (July 2026) |
| --- | --- |
| **Bark** | Mainnet-oriented payment product + SDK ecosystem; rounds/OOR language familiar to Ark readers |
| **Arkade** | Public docs + SDKs + programmable/assets roadmap; terminology and mechanics diverge from “classic Ark” blog posts on purpose |
| **Wavelength** | Young public alpha client (`v0.1.0`); very complete internal architecture; public presets use LL-hosted Ark/swap gateways on signet/testnet; mainnet is gated / bring-your-own endpoints |

Do not assume mainnet readiness, fee schedules, or operator SLAs from this table alone.

---

## Interoperability

Although all three implement Ark-family concepts (VTXOs, collaborative offchain spends, batch settlement/renewal, unilateral exit), they expose **different client/operator APIs and SDKs**. Today there is **no published compatibility layer** allowing clients from one ecosystem to communicate directly with another’s operator.

Concrete API divergence (illustrative, not exhaustive):

- Arkade publishes `api-spec` under protobuf package `ark.v1` (e.g. `RegisterIntent`, batch RPCs) with automated breaking-change policy ([arkd api-spec](https://github.com/arkade-os/arkd/tree/master/api-spec)).
- Wavelength’s in-tree operator stubs live under `package arkrpc` in [`arkrpc/`](https://github.com/lightninglabs/wavelength/tree/main/arkrpc) (different package path and method surface; e.g. version negotiation via `supported_ark_versions` on `GetInfo`).
- Bark/Second maintain their own client/operator stack (Rust SDK, barkd, captaind) rather than documenting Arkade `ark.v1` or Wavelength `arkrpc` as the wire format.

You also generally **cannot** move a VTXO from Second’s operator to Arkade’s operator as if they were one network. Community discussion has noted template/serialization divergences ([Second community RFC](https://community.second.tech/t/rfc-standardizing-virtual-transaction-templates-for-cross-implementation-interoperability/181)).

Practical rule: pick a **stack + matching gateway/operator**; design your app against that pair. Do not assume `waved → arkd` or Bark client → Arkade operator. Abstract only at your own `Backend` interface if you need portability later.

---

## Choosing a stack

| If you want… | Lean toward |
| --- | --- |
| Embed Ark + Lightning payments in a mobile/web app fast | **Bark** |
| Multi-input offchain contracts, escrow, dual-party state, assets/programmability | **Arkade** |
| An embedded Go/WASM Ark client with durable sessions, swaps, MCP/agent tooling, against a Wavelength-compatible gateway | **Wavelength** |
| Study client crash-safety / exit / OOR FSMs regardless of runtime | Read **Wavelength** ARCHITECTURE even if you ship on Arkade/Bark |
| One mental model from 2023–2024 Ark explainers | **Bark/Wavelength** vocabulary will feel closer; still verify current Second docs |
| “It’s just bitcoin txs, unbroadcast + cosigned” | **Arkade** vocabulary will feel closer |

---

## For Stable Ark specifically

See also [implementation-landscape.md](implementation-landscape.md).

Short version:

- Preferred primitive = joint multi-owner reallocation in one collaborative offchain tx.
- **Arkade** currently matches that primitive most directly.
- **Bark** can approximate with directional OOR + round consolidation, or grow multi-party support.
- **Wavelength** is an excellent Ark **client** reference (and a possible later path if a Wavelength-compatible gateway exposes the needed tx shape), not the first place to expect bilateral atomic product APIs.

Stable Ark still should **not** require a custom app-specific operator: two clients on a **normal public gateway for the chosen stack** (Arkade operator, Second operator, or Wavelength-compatible Ark gateway—not cross-wired).

---

## Sources (starting points)

- Bark / Second: [second.tech](https://second.tech), [intro](https://second.tech/docs/learn/intro), [VTXOs](https://second.tech/docs/learn/vtxo), [rounds](https://second.tech/docs/learn/rounds)
- Arkade: [docs.arkadeos.com](https://docs.arkadeos.com), [glossary](https://docs.arkadeos.com/glossary), [contracts deep dive](https://docs.arkadeos.com/contracts/deep-dive), [demos](https://github.com/arkade-os/demos)
- Wavelength: [github.com/lightninglabs/wavelength](https://github.com/lightninglabs/wavelength), [ARCHITECTURE.md](https://github.com/lightninglabs/wavelength/blob/main/ARCHITECTURE.md), [system architecture](https://wavelength.lightning.engineering/introduction/system-architecture/), [networks & config](https://wavelength.lightning.engineering/concepts/networks-and-config/)
- Classic OOR explainer (not Arkade-authoritative): [ark-protocol.org OOR](https://ark-protocol.org/intro/oor/index.html)

---

## Changelog

- **2026-07-24:** Initial public comparison from docs + implementer clarifications (Arkade multi-owner txs; Bark one-input OOR).
- **2026-07-24 (later):** Strengthen API/non-interop wording; clarify Wavelength as embedded Ark client + Wavelength-compatible gateway (LL-hosted presets on signet/testnet); soften “execution layer” rollup implication; avoid over-claiming a wholly separate Wavelength protocol.
