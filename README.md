# solana-bundle-launcher

> Atomic multi-wallet token launches on Solana via Jito bundles — ALT compression, multi-TX packing, and same-block guarantees.

A reference for how atomic bundle launches actually work on Solana — the realistic limits, the failure modes that are silent in the SDK docs, and the contract a production-grade launcher needs to honor.

---

## The problem atomic bundles solve

If you launch a token and then immediately try to also buy from N wallets in separate transactions, sandwich attackers will see the create + first-buy and front-run every subsequent buy into the next slot. By the time wallet 4 buys, the price has moved by hundreds of basis points and wallet 4 gets a worse fill than wallet 1.

Atomic bundling forces every buy to land in the **same block** as the create — eliminating the front-run window entirely.

---

## How Jito bundles work

A Jito bundle is 1–5 transactions submitted together. Validators that run Jito's modified client accept these bundles and either include all 5 in the same block or skip the bundle entirely. The 5th TX is the **tip TX** that pays the Jito validator for inclusion.

```
┌────────────────────────────────────────────────┐
│  Jito Bundle (5 TX max)                        │
├────────────────────────────────────────────────┤
│  TX 1: create token + dev buy                  │
│  TX 2: bundle wallets 1–3 buy                  │
│  TX 3: bundle wallets 4–6 buy                  │
│  TX 4: bundle wallets 7–9 buy                  │
│  TX 5: tip TX (0.001–0.005 SOL to Jito)        │
└────────────────────────────────────────────────┘
```

## Address Lookup Tables (ALT)

Without ALT, each Solana TX can fit ~2 multi-wallet buys before hitting the 1232-byte size limit. With an ALT (which lets you compress 32-byte account pubkeys to 1-byte indices), you fit 3 per TX.

| Config       | Wallets/TX | Atomic wallets/bundle |
|--------------|------------|-----------------------|
| With ALT     | 3          | 12 (4 TXs × 3)        |
| Without ALT  | 2          | 8                     |

The ALT itself has to be **created and warmed** (extended with the wallet pubkeys) at least one slot before the launch bundle, because lookups only resolve against ALT versions that the validator has already seen.

## Atomicity as a product contract

In production you don't want a "best effort" bundler. If Jito accepts but fails to include, you've spent gas + tip and got nothing. The right principle: if atomicity was promised, atomicity must be delivered or the operation must refuse. A silent fallback to sequential (non-atomic) submission is a hidden product change — worse than failing loudly.

How you enforce this is up to you; the key invariants are (a) verify inclusion before claiming success, and (b) never quietly downgrade the execution mode.

## Multi-region routing

Jito operates regional block engines. Routing through the geographically closest one shaves meaningful latency. Whether you race them in parallel, fail over sequentially, or pin to one depends on your latency budget and your tolerance for the trade-off between coverage and request multiplication.

---

## Footguns

- **Tip TX wallet needs SOL** — the wallet paying the Jito tip needs enough lamports for both the tip and its own TX fees. If preflight only checks "wallet has 0.003 SOL" but the tip is 0.005 SOL, the bundle fails silently.
- **Blockhash freshness** — if you build all 5 TXs at T+0 and submit at T+30s, the blockhash may have expired. Refresh before submit.
- **Slippage limits** are atomic-aware — if any one wallet's slippage check fails, the whole bundle fails. Set slippage at least 30%+ for atomic same-block buys.
- **Compute budget per TX** — bundling many wallets per TX raises compute cost. Set `setComputeUnitLimit(400_000)` minimum for multi-wallet buys.

## Reading list

- [Jito documentation](https://docs.jito.wtf/)
- [Address Lookup Tables in Solana](https://solana.com/docs/advanced/lookup-tables)
- [Solana TX size analyzer](https://github.com/solana-labs/solana/blob/master/sdk/program/src/message/versions/v0.rs)

## License

MIT
