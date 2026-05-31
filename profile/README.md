<div align="center">

<img src="./asset/RevvFi.png" width="200" height="200" alt="RevvFi Logo" />
<h1>RevvFi</h1>
 
**Decentralized Peer-to-Peer Lending Infrastructure**
 
Isolated borrower markets · Senior/Junior tranching · Dutch auction liquidations · Epoch-based withdrawals
 
</div>
<br/>
<p align="center">
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/License-MIT-7c3aed?style=flat-square" />
  </a>
  <a href="https://base.org">
    <img src="https://img.shields.io/badge/Base-Mainnet-0052FF?style=flat-square&logo=coinbase&logoColor=white" />
  </a>
  <a href="https://arbitrum.io">
    <img src="https://img.shields.io/badge/Arbitrum-One-2D374B?style=flat-square&logo=arbitrum&logoColor=96BEDC" />
  </a>
  <a href="https://chain.link">
    <img src="https://img.shields.io/badge/Chainlink-Oracles-375BD2?style=flat-square&logo=chainlink&logoColor=white" />
  </a>
  <a href="https://soliditylang.org">
    <img src="https://img.shields.io/badge/Solidity-0.8.33-363636?style=flat-square&logo=solidity&logoColor=white" />
  </a>
  <img src="https://img.shields.io/badge/Status-In%20Development-f59e0b?style=flat-square" />
</p>

 
## What is RevvFi?
 
RevvFi is a decentralized lending marketplace where borrowers deploy their own isolated lending markets and lenders compete to provide liquidity through a competitive offer mechanism. Interest rates are discovered on-chain through APR-based offer matching — no governance token, no shared risk pools.
 
Every market is borrower-specific. Every lender position is a transferable NFT. Every liquidation goes through a transparent Dutch auction.
 
---
 
## How It Works
 
### For Borrowers
 
**1. Register** — Get whitelisted by the ArchController (protocol admin).
 
**2. Deploy a market** — Pay the deployment fee; a full set of contracts is cloned automatically: Market, CollateralEscrow, OfferBook, and LiquidityQueue.
 
**3. Deposit collateral** — Lock collateral (e.g., WETH) into escrow. The protocol uses Chainlink oracles to value it in real-time.
 
**4. Draw down** — Call `borrow()` specifying amount, max APR, and whether to use senior offers only. The OfferBook matches the cheapest available offers.
 
**5. Repay** — Repay in any amount. Interest is distributed first, then principal. Full repayment closes all positions and improves your on-chain reputation score.
 
### For Lenders
 
**1. Submit an offer** — Specify amount, APR (in basis points), seniority (Senior or Junior), and duration. Funds are locked in the OfferBook.
 
**2. Get matched** — When a borrower draws down, your offer is filled at your stated APR. A Position NFT (ERC721) is minted to your wallet representing your claim.
 
**3. Earn interest** — Interest accrues continuously using a Compound-style index. No action required.
 
**4. Withdraw** — Request withdrawal via the LiquidityQueue. Requests are processed each 7-day epoch, pro-rata against available liquidity.
 
---
 
## Protocol Architecture
 
```
RevvFiFactory
├── RevvFiArchController         (permission registry, asset blacklist)
├── RevvFiPositionNFT            (ERC721, all markets share one contract)
├── RevvFiLiquidator             (singleton Dutch auction engine)
└── ReputationRegistry           (on-chain borrower scoring, 0–1000)
 
Per Market (EIP-1167 clones, deployed by Factory):
├── RevvFiMarket                 (core borrow / repay / interest logic)
├── RevvFiCollateralEscrow       (Chainlink-valued collateral custody)
├── RevvFiOfferBook              (APR bucket-based offer matching)
└── RevvFiLiquidityQueue         (epoch-based withdrawal queue)
```
 
Each borrower operates a fully isolated cluster of contracts. Risk does not cross market boundaries.
 
---
 
## Key Design Decisions
 
- **Isolated markets per borrower.** Borrower defaults cannot affect other markets. Collateral, offers, and positions are scoped per-market.
- **Senior / Junior tranching.** Lenders choose their risk tier. Senior positions (seniority = 0) are repaid first in liquidation. Junior positions (seniority = 1) absorb losses first but may command higher APRs.
- **O(1) interest accrual.** A Compound-style global borrow index means interest is computed without looping over all positions. Gas cost is constant regardless of active position count.
- **APR bucket matching.** Offers are organized in 100 bps (1%) buckets. The OfferBook always fills from the lowest available APR, giving borrowers the best rate the market will bear.
- **EIP-1167 minimal proxies.** Each market deployment clones four implementation contracts. Gas cost per deployment is ~100 bytes per clone versus a full deploy.
- **Position NFTs are transferable.** Lender positions are ERC721 tokens. A secondary market for positions is possible without any protocol changes.
- **Dutch auction liquidations.** When a market's collateral ratio falls below its liquidation threshold, collateral is auctioned starting at 100% of debt, declining 5% per hour, with an 80% reserve floor. Late bids extend the auction by 15 minutes to prevent sniping.
- **Epoch-based withdrawals.** Withdrawal requests are queued and fulfilled pro-rata every 7 days. This prevents flash-loan-style bank runs and ensures orderly liquidity management.
- **On-chain reputation.** Borrowers start with a score of 500. Successful repayments improve it; defaults penalize it by 50 points each. Risk labels range from AAA (900+) to D (<300).
---
 
## Interest & Collateral Formulas
 
**Interest accrual (per second):**
```
Interest = Principal × APR (bps) × Δt (seconds) / (365 days × 10,000)
```
 
**Collateral ratio:**
```
Collateral Ratio (bps) = (Collateral Value × 10,000) / Total Debt
```
 
**Collateral value (oracle-adjusted):**
```
Collateral Value = (collateralAmount × oraclePrice) / (10 ^ collateralDecimals)
```
 
Oracle prices follow Chainlink's 8-decimal standard. Staleness threshold: 2 hours.
 
**Reputation score:**
```
successRate     = (successfulLoans × 1000) / totalLoans
defaultPenalty  = defaultedLoans × 50
reputationScore = clamp(successRate − defaultPenalty, 0, 1000)
```
 
---
 
## Key Constants
 
| Parameter | Value | Description |
|---|---|---|
| `MAX_APR_BPS` | 5000 (50%) | Maximum lender offer rate |
| `MAX_ACTIVE_POSITIONS` | 100 | Max concurrent positions per market |
| `MAX_OFFERS_TO_MATCH` | 50 | Max offers consumed per drawdown |
| `EPOCH_DURATION` | 7 days | Withdrawal processing window |
| `DUST_THRESHOLD` | 1e6 | Auto-settle threshold (0.01 USDC) |
| `AUCTION_DURATION` | 3 days | Dutch auction window |
| `DUTCH_DECREMENT` | 500 bps/hr | Hourly price decay |
| `RESERVE_PRICE` | 80% of debt | Minimum auction recovery |
| `TIMELOCK` | 2 days | ArchController update delay |
| `MIN_OFFER` | 100e6 | Minimum offer (100 USDC) |
 
---
 
## Repository Structure
 
| Repository | Description |
|---|---|
| [`revvfi-contracts`](https://github.com/RevvFi/revvfi-contracts) | Core Solidity contracts (Foundry) |
| [`revvfi-docs`](https://github.com/RevvFi/revvfi-docs) | Protocol documentation (Nextra) |
| [`revvfi-interfaces`](https://github.com/RevvFi/revvfi-interfaces) | Frontend application |
| [`revvfi-monitoring`](https://github.com/RevvFi/revvfi-monitoring) | Observability stack (Loki) |
 
---
 
## Networks
 
| Network | Priority | Status |
|---|---|---|
| Base | Primary | Planned |
| Arbitrum One | Primary | Planned |
| Ethereum Mainnet | Secondary | Planned |
 
---
 
## Tech Stack
 
<p>
  <img src="https://img.shields.io/badge/Solidity-0.8.33-363636?style=flat-square&logo=solidity&logoColor=white" />
  <img src="https://img.shields.io/badge/Foundry-Test%20%26%20Deploy-FF6600?style=flat-square" />
  <img src="https://img.shields.io/badge/OpenZeppelin-4.9%2B-4E5EE4?style=flat-square&logo=openzeppelin&logoColor=white" />
  <img src="https://img.shields.io/badge/Chainlink-Oracles-375BD2?style=flat-square&logo=chainlink&logoColor=white" />
  <img src="https://img.shields.io/badge/Next.js-Frontend-000000?style=flat-square&logo=next.js&logoColor=white" />
  <img src="https://img.shields.io/badge/TypeScript-wagmi-3178C6?style=flat-square&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/Nextra-v3%20Docs-000000?style=flat-square" />
  <img src="https://img.shields.io/badge/Cloudflare-Pages-F38020?style=flat-square&logo=cloudflare&logoColor=white" />
</p>
---
 
## Getting Started
 
**Prerequisites:** Node.js 16+, [Foundry](https://foundry.paradigm.xyz)
 
```bash
git clone https://github.com/RevvFi/revvfi-contracts
cd revvfi-contracts
forge install
forge build
```
 
**Run tests:**
 
```bash
forge test
forge test --gas-report
forge coverage --report lcov
```
 
**Deploy to Sepolia:**
 
```bash
source .env
forge script script/DeployRevvFi.s.sol \
  --rpc-url $SEPOLIA_RPC \
  --private-key $DEPLOYER_PRIVATE_KEY \
  --broadcast
```
 
---
 
## Security
 
RevvFi contracts follow a strict security model:
 
- Per-market contract isolation — no shared risk pools
- `ReentrancyGuard` on all state-changing functions
- Market pauses automatically during active liquidation
- 2-hour Chainlink oracle staleness check before any price-sensitive operation
- 2-day timelock on all ArchController configuration changes
- Core singleton contracts (PositionNFT, Liquidator, ReputationRegistry) set once and immutable
- No admin keys on deployed markets — only the borrower controls their market
> Audit reports will be published in `revvfi-contracts/audits` prior to mainnet deployment.
 
---
 
## Documentation
 
Full technical specification, API layer design, database schema, event indexing guide, and off-chain calculation reference are available at [docs.revvfi.xyz](https://docs.revvfi.xyz).
 
---
 
## Team
 
| Role | Name |
|---|---|
| CEO / Blockchain Architect | Preet Singh |
| CTO / Backend Architect | Anitesh Kumar |
 
---
 
## License
 
RevvFi is released under the [MIT License](./LICENSE).
---
 
<div align="center">
  <sub>Built on-chain. Governed by math. Zero trusted intermediaries.</sub>