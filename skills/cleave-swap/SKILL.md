---
name: cleave-swap
description: Plans and tracks token swaps, cross-chain swaps, bridges and native-gas top-ups on Ethereum, Arbitrum, Base, BNB Chain, Polygon and Optimism through Cleave's MCP server or HTTP API, which return unsigned transactions for the user's own wallet to sign. Use when the user wants to swap or bridge tokens on these chains, get ETH, BNB or POL for gas (also from a wallet that holds USDC and no gas), move a balance back to USDC, check whether a transfer sent through Cleave arrived, or look up a token's contract address. The skill signs and sends nothing; the user's wallet approves every signature.
---

# Cleave swaps, bridges and gas top-ups

Use the Cleave MCP server at `https://api.cleave.fi/mcp`, or the HTTP API at `https://api.cleave.fi/agent/v1/`.

1. If the user names a token by symbol and its address is unknown, call `cleave_find_token`. When several tokens share the symbol, ask the user which address.
2. Call `cleave_plan_swap` with the chains, the tokens, the amount as a decimal string and the user's wallet.
3. Read the plan before anything is signed: blocking notices and warnings first, then the outcome kind, the minimum, any fallback token, the third-party costs and `validUntil`. A blocked plan has no calls. When `requiresConfirmation` is true, ask the user.
4. Hand the calls to the user's wallet in order, before `validUntil`. Never sign or send on the user's behalf.
5. After the wallet sends, call `cleave_track_swap` with the chain, the mined transaction hash and the plan's `requestId`. Do not send the plan again while it is in flight.
6. For a wallet that holds USDC but no native gas, use `cleave_get_gas`: it returns an offer and one USDC authorization for the wallet to sign; signing costs no gas. Call it again with the offer and the signature, then track it by `requestId`.

Use is subject to the Terms of Use (https://cleave.fi/terms) and the Privacy Policy (https://cleave.fi/privacy).
