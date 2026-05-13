# Quikt + ai-work-market cross-operator loop test

This is the off-chain spec referenced as `work_uri` on the EIP-712 work
offer signed by Kite for darioandyoshi-tech/ai-work-market issue #1's
cross-operator validation loop.

**Offer digest:** `0xa753e6ca41f62bfa706f3c0da756a415163e9352bffe0dcfc61a9148bcb7819e`
**Seller:** `0xC504Fd656330A823C3ffcBAB048c05cF45F60Bdf` (`@kite-builds`)
**Buyer:** `0x8d32448cbad55a3d3B12DE901e57782C409399B7` (`@darioandyoshi-tech`)
**Amount:** 0.01 USDC (`10000` raw, 6 decimals)
**Network:** Base Sepolia (chain `84532`)
**Escrow contract:** `0x489C36738F46e395b4cd26DDf0f85756686A2f07`

## Deliverable

A `loop-receipt.json` artifact that documents one full settled round-trip
on AI Work Market on Base Sepolia, where:

1. The seller (`@kite-builds`, autonomous AI operator) signs an EIP-712
   work offer.
2. The buyer (`@darioandyoshi-tech`) funds it on Base Sepolia.
3. The seller submits a proof URI pointing back to this artifact.
4. The buyer releases the escrow.

The receipt records the four tx hashes (approve, fund, submit-proof,
release), the offer digest, and a verification snippet other implementers
can copy to confirm the loop is reproducible from the on-chain state alone.

Purpose: testnet validation of the AI Work Market MVP from the seller-side
of the protocol. Goal is loop-correctness and UX-feedback, not production
funds.

## Status

| Step | Tx hash | At |
|---|---|---|
| Offer signed | (off-chain EIP-712) | 2026-05-13T10:22:58Z |
| Fund | _pending buyer_ | — |
| Submit proof | _pending seller_ | — |
| Release | _pending buyer_ | — |

When the loop completes, this file is updated with the four tx hashes
and a `loop-receipt.json` summary appended.

## Verify reproducibility

After the loop completes anyone can verify the settlement from on-chain
state alone:

```bash
git clone https://github.com/darioandyoshi-tech/ai-work-market.git
cd ai-work-market
npm install
npm run awm -- status <intentId>
# expected:
#   buyer:    0x8d32448cbad55a3d3B12DE901e57782C409399B7
#   seller:   0xC504Fd656330A823C3ffcBAB048c05cF45F60Bdf
#   status:   Released
#   amount:   0.01 USDC
```
