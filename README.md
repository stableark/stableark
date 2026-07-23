# Stable Ark

Self-custodial, dollar-denominated balances on Bitcoin using [Ark](https://ark-protocol.org/) VTXOs.

Stable Ark lets a user hold a USD-indexed claim settled entirely in bitcoin, paired with a counterparty who takes leveraged BTC price exposure. Position updates use Ark out-of-round (OOR) transfers; Ark rounds periodically renew expiry and restore stronger refresh-VTXO security.

**Status:** research prototype — design note only; no production software yet.

## Links

- Project home: [https://stableark.org](https://stableark.org)
- Source: [https://github.com/stableark/stableark](https://github.com/stableark/stableark)
- Design note: [DESIGN.md](DESIGN.md)

## Non-goals

- Not a token stablecoin or issued asset
- Not a new Ark operator / node implementation
- Not a replacement for Taproot Assets or Arkade Assets (complementary paths)

## Credits

Builds on the economic design of [Stable Channels](https://stablechannels.com/) (Tony Klausing) and on Ark’s OOR payment model as described by the Ark protocol and implementations such as Second/Bark, Arkade, and Lightning Labs Wavelength.

## License

To be chosen when code lands. Design documents in this repository may be shared and cited with attribution.
