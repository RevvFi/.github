
<div align="center">
<br/>
<img src="./asset/RevvFi.png" width="200" height="200" alt="RevvFi Logo" />
<br/>
<br/>

# <center>RevvFi</center>

**Liquidity-Backed Token Launch Infrastructure**
 
A decentralized protocol for fair, trustless token launches — where liquidity providers hold governance over every launch they back.
 
<br/>
[![License: MIT](https://img.shields.io/badge/License-MIT-7c3aed?style=flat-square)](https://opensource.org/licenses/MIT)
[![Built on Base](https://img.shields.io/badge/Base-Mainnet-0052FF?style=flat-square&logo=coinbase&logoColor=white)](https://base.org)
[![Built on Arbitrum](https://img.shields.io/badge/Arbitrum-One-2D374B?style=flat-square&logo=arbitrum&logoColor=96BEDC)](https://arbitrum.io)
[![Powered by Uniswap V2](https://img.shields.io/badge/Uniswap-V2-FF007A?style=flat-square&logo=uniswap&logoColor=white)](https://uniswap.org)
[![Solidity](https://img.shields.io/badge/Solidity-^0.8.x-363636?style=flat-square&logo=solidity&logoColor=white)](https://soliditylang.org)
[![Status: In Development](https://img.shields.io/badge/Status-In%20Development-f59e0b?style=flat-square)]()
 
</div>
---
 
## What is RevvFi?
 
RevvFi is a smart contract protocol that enables token creators to launch tokens through a structured, liquidity-backed crowdfund. Liquidity providers (LPs) deposit ETH, retain governance over treasury funds, and withdraw proportionally after a defined lock period — without trusting the creator.
 
Every launch is isolated in its own set of contracts. Every treasury is LP-controlled. Creators cannot access LP funds at any point.
 
---
 
## How It Works
 
**1. Creator deploys a launch** — pays a 0.1 ETH launch fee, configures token supply, lock duration, and allocation splits. A full set of contracts is deployed automatically.
 
**2. LPs deposit ETH** — during the raise window. Shares are minted 1:1 with ETH deposited and serve directly as governance instruments.
 
**3. Launch executes permissionlessly** — once the ETH target is met, anyone can trigger `launch()`. Liquidity is deployed to Uniswap V2. Vaults receive their allocations.
 
**4. LPs govern the treasury** — LP shares control voting power. No separate governance token. Proposals require majority thresholds to move treasury or strategic reserve funds.
 
**5. Withdrawal after maturity** — once `maturityTime` (raise end + lock duration) passes, LPs can withdraw as proportional ETH + tokens, or auto-swap to ETH only.
 
**6. Refunds on failure** — if the ETH target is not reached before the raise window closes, LPs claim full refunds proportionally.
 
---
 
## Protocol Architecture
 
```
RevvFiFactory
├── RevvFiBootstrapper          (per-launch orchestrator)
├── TokenTemplateFactory        (immutable ERC20 deployment)
├── CreatorVestingVault         (cliff + linear vesting)
├── TreasuryVault               (LP-governed, 60% threshold, 7d timelock)
├── StrategicReserveVault       (LP-governed, 66% threshold, 14d timelock)
├── RewardsDistributor          (fixed schedule emissions)
├── RevvFiGovernance            (share-weighted voting)
├── CreatorProfileRegistry      (on-chain reputation)
└── PopularityOracle            (launch scoring, 0–100)
```
 
Each launch deploys its own isolated Bootstrapper and vault contracts. The Factory maintains a global registry.
 
---
 
## Key Design Decisions
 
**No governance token in v1.** LP shares are the governance instrument. Voting power equals share balance. No REVV token, no additional tokenomics overhead.
 
**Creator zero-access treasury.** The TreasuryVault is deployed for every launch. Only RevvFiGovernance can execute releases. The creator has no access, ever.
 
**Permissionless launch execution.** Any address can call `launch()` once conditions are met. A 0.01 ETH incentive is paid to the caller from platform fees.
 
**Explicit maturity formula.** `maturityTime = raiseEndTime + lockDuration`. Lock duration is bounded between 30 and 730 days, set by the creator at launch and immutable thereafter.
 
**Immutable token contracts.** Tokens deployed via `TokenTemplateFactory` have no mint function after construction, no upgrade proxy, no blacklist, and no transfer pause.
 
---
 
## Repository Structure
 
| Repository | Description |
|---|---|
| [`revvfi-contracts`](https://github.com/RevvFi/revvfi-contracts) | Core smart contracts — Factory, Bootstrapper, Vaults, Governance |
| [`revvfi-docs`](https://github.com/RevvFi/revvfi-docs) | Documentation for the RevvFi protocol |
| [`interfaces `](https://github.com/RevvFi/interfaces) | Frontend interface — launch dashboard, LP portal |
| [`.github`](https://github.com/RevvFi/.github) | Organization profile, templates, and shared workflows |
 
---
 
## Networks
 
| Network | Priority | Status |
|---|---|---|
| Base | Primary | Planned |
| Arbitrum One | Primary | Planned |
| Ethereum Mainnet | Secondary | Planned |
 
---
 
## Integrations
 
<div>
[![Uniswap V2](https://img.shields.io/badge/Uniswap%20V2-Liquidity%20Pairs-FF007A?style=flat-square&logo=uniswap&logoColor=white)](https://uniswap.org)
&nbsp;
[![Uniswap V2](https://img.shields.io/badge/WETH-ETH%20Wrapping-FF007A?style=flat-square&logo=ethereum&logoColor=white)](https://uniswap.org)
&nbsp;
[![OpenZeppelin](https://img.shields.io/badge/OpenZeppelin-Security%20Standards-4E5EE4?style=flat-square&logo=openzeppelin&logoColor=white)](https://openzeppelin.com)
&nbsp;
[![Hardhat](https://img.shields.io/badge/Hardhat-Development%20Framework-F0DB4F?style=flat-square&logo=hardhat&logoColor=black)](https://hardhat.org)
&nbsp;
[![Base](https://img.shields.io/badge/Base-L2%20Deployment-0052FF?style=flat-square&logo=coinbase&logoColor=white)](https://base.org)
&nbsp;
[![Arbitrum](https://img.shields.io/badge/Arbitrum-L2%20Deployment-2D374B?style=flat-square&logo=arbitrum&logoColor=96BEDC)](https://arbitrum.io)
 
</div>
---
 
## Security
 
RevvFi v1 contracts follow a strict security model:
 
- Per-launch contract isolation — no cross-contamination of funds
- Non-upgradeable Bootstrapper and vault instances
- Emergency pause via guardian multisig
- Governance timelocks on all treasury actions (7–14 days)
- No admin keys on token contracts post-deployment
Audit reports will be published in `revvfi-contracts/audits` prior to mainnet deployment.
 
---
 
## Documentation
 
Full technical specification is available in `revvfi-docs`. This includes contract interfaces, state variable definitions, event specifications, deployment flows, and gas estimates.
 
---
 
## License
 
RevvFi protocol contracts are released under the [MIT License](./LICENSE).
 
---
 
<div align="center">
<sub>Built with intention. Governed by liquidity providers.</sub>
 
</div>