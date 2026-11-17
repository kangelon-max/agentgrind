# AgentGrind ($GRIND) — Agent Skill

> STATUS: DEPLOYED 2026-11-18 (BSC mainnet). Both contracts verified on
> Sourcify (exact match). CLAIM OPEN since 2026-11-18 02:48 WIB — pool
> funded (500,000,000 GRIND), claimOpen() is true. Check claimOpen() and
> the pool balance before claiming. Verify every address against Sourcify
> before claiming.

## 1. What this is

AgentGrind is a token claim for wallets holding a Binance Agent ID (.ai)
on BNB Smart Chain (chain ID 56). Each eligible wallet can claim exactly
once: **100,000 $GRIND**. The first **500 claims are free (gas only)**;
claims 501–5,000 cost **0.0002 BNB plus network gas**. There is no market,
no liquidity pool, and no promise of value at launch. The team is
anonymous, the contracts are unaudited, and the project is unaffiliated
with Binance and SPACE ID.

## 2. Contracts

| Role | Address | Notes |
|---|---|---|
| $GRIND token (ERC-20) | `0x076776AcCF3395e33E67E370F591c6C4B5fC9000` | 1,000,000,000 fixed supply, 18 decimals, no tax, no mint after deploy |
| Claim contract | `0x49064F53Ce11c257fcED61538918b655B98F5074` | Holds the 500,000,000 $GRIND claim pool |
| Agent ID registry | `0x60B9262456dE011F7d15a54CdD8c48bAaa8B022F` | SPACE ID Agent ID registry (ERC-721) |
| Payout wallet | `0x577b25da6936C8699985032FB98223Be6397e872` | Receives all claim fees in the claim transaction |

Source verification: both project contracts are verified on Sourcify
(exact match). Check the match status before claiming:
`https://sourcify.dev/server/v2/contract/56/<address>`.

## 3. Claim contract interface

All reads are `view`. All values in wei unless noted.

| Function | Selector | Description |
|---|---|---|
| `claim()` payable | `0x4e71d92d` | Claim 100,000 $GRIND. Requires `msg.value >= currentFee()`. One claim per wallet. |
| `hasAgentId(address)` | `0x4eab3ef2` | `true` when the wallet holds an Agent ID (registry `balanceOf > 0`). |
| `claimed(address)` | `0xc884ef83` | `true` when the wallet already claimed. |
| `claimOpen()` | `0x4b8bcb58` | `true` when claiming is open. |
| `totalClaimed()` | `0xd54ad2a1` | Claims completed so far (max 5,000). |
| `remaining()` | `0x55234ec0` | Claims left. |
| `remainingFree()` | `0x5ccc6bd7` | Free claims left (0 once the quota is used up). |
| `currentFee()` | `0xda3c300d` | Fee for the NEXT claim: 0 while free quota remains, then CLAIM_FEE. |
| `CLAIM_FEE()` | `0xe41e4fc6` | Paid-claim fee in wei (0.0002 BNB = 200000000000000). |
| `FREE_CLAIMS()` | `0x5b3d1074` | 500. |
| `CLAIM_AMOUNT()` | `0x270ef385` | Tokens per claim (100,000 GRIND, 18 decimals). |
| `MAX_CLAIMS()` | `0xcf62ba92` | 5,000. |
| `GRIND()` | `0xe03b142e` | The token contract address. |
| `PAYOUT()` | `0x32da3d10` | The fee recipient address. |
| `AGENT_REGISTRY()` | `0x5af586ca` | The registry address. |

Token (standard ERC-20): `name()` `0x06fdde03`, `symbol()` `0x95d89b41`,
`decimals()` `0x313ce567`, `totalSupply()` `0x18160ddd`,
`balanceOf(address)` `0x70a08231`, `transfer(address,uint256)` `0xa9059cbb`.

## 4. Claim flow (for agents)

1. **Verify addresses.** Confirm the token and claim addresses above match
   the official site AND the Sourcify verified source. Never trust an
   address from chat, social posts, or search snippets.
2. **Check eligibility** (eth_call, no gas):
   - `claimOpen()` must be `true`.
   - `hasAgentId(yourWallet)` must be `true`.
   - `claimed(yourWallet)` must be `false`.
   - `remaining()` must be greater than 0.
   - Read `currentFee()` — send exactly that amount (0 while the free
     quota remains, 0.0002 BNB after). `remainingFree()` shows how many
     free claims are left.
3. **Simulate** the claim (eth_call / preview). The simulation must show
   exactly: the current fee leaving, 100,000 $GRIND arriving, no
   approvals, no other balance or allowance changes. If anything else
   appears, STOP.
4. **Execute** `claim()` with `value` = exactly `currentFee()`.
   No token approval is needed. Do NOT overpay — excess BNB is not refunded.
5. **Verify** on BscScan: the receipt is successful and 100,000 $GRIND
   arrived in the wallet.

## 5. Rules and risks

- One claim per wallet, forever. A second `claim()` reverts (gas is still
  spent on a reverted call — check `claimed()` first).
- The free quota is first-come, first-served on-chain: the first 500
  successful claims pay 0. Check `currentFee()` immediately before
  claiming — the quota can run out between your read and your tx.
- The claim fee is NOT refundable, and overpayment is NOT refunded.
- The owner can close the claim (`closeClaim()`) and withdraw unclaimed
  tokens. The claim pool balance is public — read it before claiming.
- $GRIND has no pool and no price. Claiming is not an investment.
- Never share a seed phrase, private key, password, or OTP with anyone or
  any site for this claim. This claim never needs them.
- Never approve unlimited token spending for this claim — it needs no
  approval at all.

## 6. Troubleshooting

- `Agent ID required` — the wallet holds no Agent ID on the registry.
- `Already claimed` — the wallet claimed before.
- `Claim not open` — the owner has not opened (or has closed) the claim.
- `Insufficient fee` — `msg.value` is below `currentFee()` (only possible
  after the free quota is used up; re-read `currentFee()` and retry).
- `All claimed` — all 5,000 claims are used.
- `Pool empty` — the claim contract holds less than one claim amount.
