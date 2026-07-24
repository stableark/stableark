# Stable Ark

Self-custodial, dollar-denominated balances on Bitcoin using Ark-family VTXOs.

Stable Ark lets a user hold a USD-indexed claim settled entirely in bitcoin, paired with a counterparty who takes leveraged BTC price exposure. Position updates are **joint multi-input / multi-output offchain transactions**; lifecycle renewal uses each stack’s batch settlement path.

**Status:** research prototype — design note stage, **Arkade-first** PoC planned. No production software yet.

## Links

- Project home: [https://stableark.org](https://stableark.org)
- Source: [https://github.com/stableark/stableark](https://github.com/stableark/stableark)
- Design note: [DESIGN.md](DESIGN.md)
- User onboarding flow: [notes/user-onboarding.md](notes/user-onboarding.md)
- Stack comparison (Bark vs Arkade vs Wavelength): [notes/stack-comparison.md](notes/stack-comparison.md)
- Implementation landscape (Stable Ark–focused): [notes/implementation-landscape.md](notes/implementation-landscape.md)
- Joint multi-input primitive: [notes/bilateral-atomic-oor.md](notes/bilateral-atomic-oor.md)
- Domain / Pages setup: [DOMAIN.md](DOMAIN.md)

## Non-goals

- Not a token stablecoin or issued asset
- Not a new Ark operator / node implementation
- Not a replacement for Taproot Assets or Arkade Assets (complementary paths)

## Credits

Builds on the economic design of [Stable Channels](https://stablechannels.com/) (Tony Klausing) and on Ark-family VTXO systems as implemented by Arkade, Second/Bark, and Lightning Labs Wavelength.

## License

To be chosen when code lands. Design documents in this repository may be shared and cited with attribution.
