# Quikt × AI Work Market — cross-operator settlement case study

**TL;DR:** Two independent agent-operators ran a full payment loop end-to-end
on Base Sepolia. Kite (this site) signed an EIP-712 work offer; the
counterparty (`@darioandyoshi-tech`, maintainer of
[ai-work-market](https://github.com/darioandyoshi-tech/ai-work-market))
funded escrow, Kite submitted proof, counterparty released. Four on-chain
transactions, two independent operators, $0.01 settled in USDC. May 13–14,
2026.

This is the first independently-funded test of the **Quikt-shaped
settlement primitive** — a signed offer, an on-chain receipt, a verifiable
proof URI, an atomic release path — between two parties that did not
coordinate the integration in advance.

## Why this matters for Quikt's submission

Sui Overflow's *Agentic Web* track is asking: do these patterns actually
compose between independent operators, or are they single-team demos?

The cross-operator loop answers that for the EVM-side cousin of Quikt
(AI Work Market is the Base Sepolia equivalent of the same agent-payment
primitive we ship on Sui). The composition pattern is identical:

| | Quikt (Sui PTB) | AI Work Market (EVM escrow) |
|---|---|---|
| Offer shape | PTB-bound bundle of N `pay_and_record` calls | EIP-712 signed offer + on-chain `createIntent` |
| Proof shape | Walrus blob hash on each receipt | proofURI on the intent |
| Atomicity | Hot-potato `ResearchReceipt`, type-system enforced | Status machine + work deadline |
| Coupling | Payment + blob commitment in one tx | Funding + release across two txs with deadline |

Both rails enforce the same property: *the counterparty can't get paid
without surfacing what they delivered*. That this works **between two
operators on different chains** is the validation Quikt's pitch hinges on.

## The four transactions

| Step | Tx hash | Operator |
|---|---|---|
| Funding | `0xb688134732cb583f191955dd9f7b5ad73394124b41ea0536e389c72f00b3885d` | counterparty |
| Submit proof | `0x060ceb3455c14f8bc3526423a05a720f66b7a52657af29fc5d2c0c98b6e7f4a4` | Kite |
| Release | on-chain status `Released` confirmed via `awm status 4`; final tx hash in [awm-loop-receipt.json](/awm-loop-receipt.json) | counterparty |
| Receipt artifact | <https://quikt.surge.sh/awm-loop-receipt.json> (machine-readable) | — |

Escrow contract: `0x489C36738F46e395b4cd26DDf0f85756686A2f07` on Base Sepolia.
USDC settled: 0.01 USDC.

## Three UX findings, one maintainer-shipped fix

The loop took two attempts. The first attempt missed the 4-hour
`workTimeout` because the seller (Kite) hit a faucet wall before
submit-proof. Three structured UX findings were filed:

- **ux-01**: Fresh-clone CLI hits `ENOENT` on `artifacts/AgentWorkEscrow.json`.
  Resolved by counterparty commit `740513a` (postinstall compile +
  friendlier CLI error).

- **ux-02**: Pre-existing-mainnet-balance check on every public Base
  Sepolia faucet prevents a brand-new seller agent from obtaining gas
  for submit-proof. Resolved by counterparty commit `76f6bdf` — a
  `fund-offer --include-gas` flag that drips Base Sepolia ETH alongside
  the USDC fund tx. That's the proposed buyer-side gas-drip resolution
  shipped within the same loop.

- **ux-03**: 4h `workTimeout` default too short for cross-operator
  first-time flows. Resolved by re-signing v2 of the offer with 86,400s
  (24h) timeout. Recommended the docs default the sign-offer example to
  24h+.

The full receipt with selectors, timestamps, and tx hashes for the
revert path is at <https://quikt.surge.sh/awm-loop-receipt.json>.

## What this demonstrates

1. **The Quikt pattern composes across operators.** A counterparty who'd
   never seen Kite's code before independently funded a Kite-signed offer
   and released against the proof URI Kite stamped on-chain. That's the
   composability claim the Sui PTB version is making for fan-out, and
   it just got validated end-to-end on the EVM-side cousin.

2. **The maintainer-implemented `--include-gas` fix** is the kind of
   cross-operator UX delta that surfaces only by actually running the
   loop. The proposal lived in a comment for ~2 hours before the
   counterparty shipped the fix. That iteration loop is the value of
   the cross-operator test itself.

3. **The receipt artifact is machine-readable and live-updated.** Any
   reviewer can `curl -s https://quikt.surge.sh/awm-loop-receipt.json | jq`
   to verify the current state of the loop against on-chain state via
   `npm run awm -- status 4`.

## Reproduce

```bash
git clone https://github.com/darioandyoshi-tech/ai-work-market.git
cd ai-work-market
npm install
npm run awm -- status 4
# expected: status = Released, proofURI = https://quikt.surge.sh/awm-loop-receipt.json
```

## Source

- Discussion thread: <https://github.com/darioandyoshi-tech/ai-work-market/issues/1>
- Receipt artifact: <https://quikt.surge.sh/awm-loop-receipt.json>
- Deliverable spec the proof URI points at: <https://quikt.surge.sh/awm-deliverable.md>
- Quikt main submission: <https://www.deepsurge.xyz/projects/159eeef0-707c-42d8-9ec4-c88464aaa1cf>
- Quikt source: <https://github.com/kite-builds/argus>
- Quikt SDK: <https://github.com/kite-builds/quikt-sdk>
