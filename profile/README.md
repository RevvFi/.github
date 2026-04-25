<div align="center">

<img src="./asset/RevvFi.png" width="200" height="200" alt="RevvFi Logo" />

<h1>RevvFi</h1>

**Liquidity-Backed Token Launch Infrastructure**

A decentralized protocol for fair, trustless token launches — where liquidity providers hold governance over every launch they back.

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
  <a href="https://uniswap.org">
    <img src="https://img.shields.io/badge/Uniswap-V2-FF007A?style=flat-square&logo=uniswap&logoColor=white" />
  </a>
  <a href="https://soliditylang.org">
    <img src="https://img.shields.io/badge/Solidity-^0.8.x-363636?style=flat-square&logo=solidity&logoColor=white" />
  </a>
  <img src="https://img.shields.io/badge/Status-In%20Development-f59e0b?style=flat-square" />
</p>

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

- **No governance token in v1.** LP shares are the governance instrument.
- **Creator zero-access treasury.** Only governance can move funds.
- **Permissionless launch execution.** Anyone can call `launch()`.
- **Explicit maturity formula.** `maturityTime = raiseEndTime + lockDuration`
- **Immutable token contracts.** No mint, no upgrade, no admin controls.

---

## Repository Structure

| Repository | Description |
|---|---|
| [`revvfi-contracts`](https://github.com/RevvFi/revvfi-contracts) | Core smart contracts |
| [`revvfi-docs`](https://github.com/RevvFi/revvfi-docs) | Documentation |
| [`revvfi-interface`](https://github.com/RevvFi/revvfi-interface) | Frontend app |
| [`.github`](https://github.com/RevvFi/.github) | Org profile & workflows |

---

## Networks

| Network | Priority | Status |
|---|---|---|
| Base | Primary | Planned |
| Arbitrum One | Primary | Planned |
| Ethereum Mainnet | Secondary | Planned |

---

## Integrations

<p>
  <img src="https://img.shields.io/badge/Uniswap%20V2-Liquidity%20Pairs-FF007A?style=flat-square&logo=uniswap&logoColor=white" />
  <img src="https://img.shields.io/badge/WETH-ETH%20Wrapping-FF007A?style=flat-square&logo=ethereum&logoColor=white" />
  <img src="https://img.shields.io/badge/OpenZeppelin-Security%20Standards-4E5EE4?style=flat-square&logo=openzeppelin&logoColor=white" />
  <img src="https://img.shields.io/badge/Hardhat-Development%20Framework-F0DB4F?style=flat-square&logo=hardhat&logoColor=black" />
</p>

---

## Security

RevvFi v1 contracts follow a strict security model:

- Per-launch contract isolation  
- Non-upgradeable contracts  
- Governance timelocks (7–14 days)  
- No admin keys on tokens  
- Emergency pause via multisig  

Audit reports will be published in `revvfi-contracts/audits`.

---

## Documentation

Full technical specification is available in `revvfi-docs`.

---

## License

RevvFi is released under the [MIT License](./LICENSE).

---

<div align="center">
<sub>Built with intention. Governed by liquidity providers.</sub>
</div>
