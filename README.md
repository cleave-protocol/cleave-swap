# Cleave for Agents: token swaps, cross-chain swaps and gas top-ups

[![Listed on mcpservers.org](https://mcpservers.org/badge.svg)](https://mcpservers.org/servers/cleave-protocol/cleave-swap) [![mcptry](https://mcptry.com/badge/fi.cleave/swap)](https://mcptry.com/servers/fi.cleave/swap)

Swaps, bridges and gas top-ups on Ethereum, Arbitrum, Base, BNB Chain, Polygon and Optimism. cleave_plan_swap returns unsigned calls and states the outcome kind first: an exact amount or one that can vary; with a swap on arrival, the requested token or, possibly, a named fallback token; and where a transfer can go unfilled, a possible refund of the bridged token on the source chain, which can take hours or longer and may not happen. cleave_get_gas offers orders for native gas to a wallet that holds USDC but no gas, with one USDC authorization to sign. cleave_track_swap reports progress the same way on every route; cleave_find_token resolves symbols and flags look-alikes. Routes that need a claim step are left out. No API key; your wallet signs.

Give the agent a swap tool that returns unsigned transactions for its wallet to sign. Cleave Protocol (cleave.fi) offers one over MCP or HTTP on six EVM chains: each plan states the outcome kind and fallback before signing; tracking reports arrival.

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

Plans a token swap on one chain, a cross-chain swap or bridge, or a native-gas top-up on Ethereum, Arbitrum, Base, BNB Chain, Polygon or Optimism, and returns unsigned transactions for the given wallet to sign. It fits when the user wants to exchange tokens, move them between these chains, or get ETH, BNB or POL for gas. It sends nothing and moves no funds. The answer leads with blocking notices and warnings, then gives the route's outcome kind (an exact amount or one that can vary; with a swap on arrival, the token or, possibly, a named fallback; where a transfer can go unfilled, a possible refund of the bridged token on the source chain, which can take hours or longer and may not happen), every third-party cost except network gas, the expected time, validUntil and the calls in signing order. The main call can carry gas, the recommended gas limit: use it as the transaction's gas limit, and without it estimate with a priority fee above zero. A blocking notice returns no calls; a warning, or a recipient other than the wallet, sets requiresConfirmation, and the connection's settings can turn either into a refusal. Calls sent after validUntil are stale: they can revert or, on some routes, end in a refund later, or not at all. Amounts are decimal strings in token units; tokens are listed symbols or contract addresses; the recipient defaults to the wallet. Limits set in the connection (a pinned wallet, a per-plan USD cap, listed tokens only) bound every argument. For an ambiguous symbol, cleave_find_token lists the candidates; after sending, cleave_track_swap reports progress. A wallet with no gas can order it with cleave_get_gas.

### cleave_track_swap

Track a planned swap or transfer.

Reports the status of a swap, bridge or gas top-up that was planned with cleave_plan_swap and then sent, or ordered with cleave_get_gas: its state (for example pending, in flight, delivered, refund pending or refunded), what arrived against the plan and on which chain, and the next action or when to check again. It takes the chain and transaction hash, plus the plan's requestId when known, or for a gas top-up the requestId alone; a transaction to a contract outside Cleave's published list is reported as such. It fits when the user asks whether such a transfer arrived, and before any re-send, because re-sending a plan that is still in flight can pay twice. It reports states only and returns no transactions.

### cleave_find_token

Find a token's contract address.

Finds a token by symbol or contract address on Ethereum, Arbitrum, Base, BNB Chain, Polygon or Optimism and returns its address, decimals and whether it is on Cleave's token list, with a flag when it resembles a listed token. A symbol shared by several listed tokens returns up to ten candidates by address instead of picking one. An unlisted token is found only by its address and comes back flagged, without a name or symbol. It fits before cleave_plan_swap when the user names a token by symbol and the address is unknown.

### cleave_get_gas

Order native gas with USDC.

Offers orders for native gas (ETH, BNB or POL) on Ethereum, Arbitrum, Base, BNB Chain, Polygon or Optimism for a wallet that holds USDC on Ethereum, Arbitrum, Base, Optimism or Polygon but has no gas to send transactions. Called with the paying chain, the receiving chain, the recipient, a USDC amount and the paying wallet (without it the offer has nothing to sign), it returns an offer: the fee, the minimum native amount, the deadline, the order and one USDC authorization for the wallet to sign; signing costs no gas. An offer needs 0.8 to 50 USDC (at least 5 USDC when Ethereum is the paying or receiving chain) that covers the fee, a recipient that accepts the native coin and is not a token contract, and, for a cross-chain order, the paying wallet or a refundTo on the paying chain; otherwise, or while the route is paused, no offer is returned and the answer says why. Called again with the offer, the signature and the same wallet, it passes the signed order to Cleave's relayer, which may submit it to a contract made for that one order, and returns a requestId for cleave_track_swap. An authorization Cleave accepts is set to end before the order deadline. Cleave does not promise to submit it before it expires, and expiry of an unused authorization is not a promise about funds already paid. No outcome or time is guaranteed, and a settled payment is not delivery. On one chain the order is written to swap the USDC toward the native coin for the recipient. From another chain it is written to bridge the USDC and swap on arrival. USDC on arrival is one possible ending if that swap fails, and it may not happen. If a bridge transfer is not filled, the USDC can come back to the order address on the paying chain, which can take hours or longer, or may not come back. Until someone sends it to the refund address, cleave_track_swap can show it as refund_pending. The relayer may not send it. When the wallet already holds some gas, cleave_plan_swap plans a swap to the native coin.

## Safety

The tools sign nothing. Every plan returns unsigned calls for the user's own wallet; check the outcome kind and `validUntil` before signing, and sign the calls in order.

## Terms

Use is subject to the Terms of Use (https://cleave.fi/terms) and the Privacy Policy (https://cleave.fi/privacy).
