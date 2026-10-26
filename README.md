# Cleave for Agents: token swaps, cross-chain swaps and gas top-ups

Swaps, bridges and gas top-ups on Ethereum, Arbitrum, Base, BNB Chain, Polygon and Optimism, worst case first. cleave_plan_swap returns unsigned calls and states the outcome first: an exact amount or at least a minimum; with a swap on arrival, the token, else a named fallback token; and where a transfer can go unfilled, a refund of the bridged token on the source chain hours later. cleave_get_gas gets native gas for a wallet that holds USDC but no gas, from one signed USDC authorization. cleave_track_swap reports progress the same way on every route; cleave_find_token resolves symbols and flags look-alikes. Routes that need a claim step are left out. No API key; your wallet signs.

Give the agent a swap tool that returns unsigned transactions for its wallet to sign. Cleave Protocol (cleave.fi) offers one over MCP or HTTP on six EVM chains: each plan states the outcome and fallback before signing; tracking reports arrival.

## Connect

MCP (streamable HTTP, no API key):

```
https://api.cleave.fi/mcp
```

Claude Code:

```
claude mcp add --transport http cleave https://api.cleave.fi/mcp
```

Cursor, Windsurf and other clients: add a remote MCP server with the URL above.

A connection can pin one wallet: `https://api.cleave.fi/mcp/0xYourWallet`. Query settings: `maxUsd` (a per-plan USD cap) and `listedOnly` (`1` by default; `0` allows tokens outside the token list).

HTTP API: `POST https://api.cleave.fi/agent/v1/{plan_swap,track_swap,find_token,get_gas}` with the same JSON input as the tools. OpenAPI: `https://api.cleave.fi/agent/v1/openapi.json`.

## Tools

### cleave_plan_swap

Plan a swap, bridge or gas top-up.

Plans a token swap on one chain, a cross-chain swap or bridge, or a native-gas top-up on Ethereum, Arbitrum, Base, BNB Chain, Polygon or Optimism, and returns unsigned transactions for the given wallet to sign. It fits when the user wants to exchange tokens, move them between these chains, or get ETH, BNB or POL for gas. It sends nothing and moves no funds. The answer leads with blocking notices and warnings, then gives the route's outcome kind (exact or at least a minimum; with a swap on arrival, the token, else a named fallback; where a transfer can go unfilled, a refund of the bridged token on the source chain hours later), every third-party cost except network gas, the expected time, validUntil and the calls in signing order. A blocking notice returns no calls; a warning, or a recipient other than the wallet, sets requiresConfirmation, and the connection's settings can turn either into a refusal. Calls sent after validUntil are stale: they can revert or, on some routes, end in a refund hours later. Amounts are decimal strings in token units; tokens are listed symbols or contract addresses; the recipient defaults to the wallet. Limits set in the connection (a pinned wallet, a per-plan USD cap, listed tokens only) bound every argument. For an ambiguous symbol, cleave_find_token lists the candidates; after sending, cleave_track_swap reports progress. A wallet with no gas can get it with cleave_get_gas.

### cleave_track_swap

Track a planned swap or transfer.

Reports the status of a swap, bridge or gas top-up that was planned with cleave_plan_swap and then sent, or ordered with cleave_get_gas: its state (for example pending, in flight, delivered, refund pending or refunded), what arrived against the plan and on which chain, and the next action or when to check again. It takes the chain and transaction hash, plus the plan's requestId when known, or for a gas top-up the requestId alone; a transaction to a contract outside Cleave's published list is reported as such. It fits when the user asks whether such a transfer arrived, and before any re-send, because re-sending a plan that is still in flight can pay twice. It reports states only and returns no transactions.

### cleave_find_token

Find a token's contract address.

Finds a token by symbol or contract address on Ethereum, Arbitrum, Base, BNB Chain, Polygon or Optimism and returns its address, decimals and whether it is on Cleave's token list, with a flag when it resembles a listed token. A symbol shared by several listed tokens returns every candidate by address instead of picking one. An unlisted token is found only by its address and comes back flagged, without a name or symbol. It fits before cleave_plan_swap when the user names a token by symbol and the address is unknown.

### cleave_get_gas

Get native gas with USDC.

Gets native gas (ETH, BNB or POL) on Ethereum, Arbitrum, Base, BNB Chain, Polygon or Optimism for a wallet that holds USDC on Ethereum, Arbitrum, Base, Optimism or Polygon but has no gas to send transactions. Called with the paying chain, the receiving chain, the recipient and a USDC amount, it returns an offer: the fee, the minimum native amount, the deadline, the order and one USDC authorization for the wallet to sign; signing costs no gas. Called again with the offer and the signature, it passes the signed order to Cleave's relayer, which executes it through a contract made for that one order, and returns a requestId for cleave_track_swap. On one chain the recipient gets at least the minimum or nothing moves; from another chain it gets the native coin at the minimum, else USDC on arrival if the arrival swap fails. An order not executed by its deadline is refunded to the refund address, or, if paid from another chain and not filled, refunded hours later. When the wallet already holds some gas, cleave_plan_swap plans a swap to the native coin.

## Safety

The tools sign nothing. Every plan returns unsigned calls for the user's own wallet; check the outcome, the minimum and `validUntil` before signing, and sign the calls in order.

## Terms

Use is subject to the Terms of Use (https://cleave.fi/terms) and the Privacy Policy (https://cleave.fi/privacy).
