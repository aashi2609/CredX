<p align="center">
  <img src="public/icon.svg" width="80" alt="CredX Logo" />
</p>

<h1 align="center">CredX</h1>
<h3 align="center">Decentralized Invoice Factoring Protocol</h3>

<p align="center">
  <em>Transforming stagnant corporate invoices into liquid, yield-bearing on-chain assets.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Next.js-16-black?logo=next.js" alt="Next.js" />
  <img src="https://img.shields.io/badge/Solidity-0.8.26-363636?logo=solidity" alt="Solidity" />
  <img src="https://img.shields.io/badge/Polygon-Amoy_Testnet-7B3FE4?logo=polygon" alt="Polygon" />
  <img src="https://img.shields.io/badge/OpenZeppelin-v5-4E5EE4?logo=openzeppelin" alt="OpenZeppelin" />
  <img src="https://img.shields.io/badge/License-MIT-green" alt="License" />
</p>

---

## What is CredX?

CredX is a full-stack Web3 protocol that bridges real-world cash flow with on-chain liquidity. It enables **Micro, Small, and Medium Enterprises (MSMEs)** to tokenize outstanding invoices as NFTs and receive instant working capital from investors — while generating sustainable **"Real Yield"** backed by actual trade settlement.

### The Three Actors

| Role | What They Do |
|------|-------------|
| 🏭 **MSME** | Mints an invoice as an ERC-721 NFT and lists it on the marketplace for funding |
| 💰 **Investor** | Funds discounted invoices and earns principal + interest upon repayment |
| 🏢 **Big Buyer** | Confirms & repays invoices directly to the smart contract escrow |

Every repayment triggers an automated **95/5 split**: 95% flows to the investor, 5% is routed as a protocol fee to the `StakingRewards` vault — creating sustainable, non-inflationary yield for `$CGOV` stakers.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     CredX Protocol Flow                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MSME ──► createInvoice() ──► InvoiceMarketplace (Proxy)       │
│                                      │                          │
│                               mintInvoice()                     │
│                                      ▼                          │
│                                InvoiceNFT (ERC-721)             │
│                                      │                          │
│  Big Buyer ──► confirmInvoice() ─────┘                          │
│                      │                                          │
│               Status: Fundraising                               │
│                      │                                          │
│  Investor ──► investInInvoice() ──► Funds held in escrow        │
│                      │                                          │
│               Status: Funded → MSME receives capital            │
│                      │                                          │
│  Big Buyer ──► repayInvoice() ──► InvoiceNFT                   │
│                      │                                          │
│              ┌───────┴───────┐                                  │
│              ▼               ▼                                  │
│     5% → StakingRewards   95% → Marketplace                    │
│       (CGOV stakers)      (Investor claims)                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3-Tier Real-Yield Tokenomics

CredX implements a three-tier token economy where value flows from real-world trade settlement to on-chain stakers:

### Tier 1 — Access (`$CREDX` Utility Token)

> **Contract:** `CredXUtilityToken.sol` · ERC-20

Lightweight access token admin-minted to MSMEs and Investors. Acts as the entry point into the protocol's staking flywheel.

### Tier 2 — Governance Extraction (`$CGOV`)

> **Contracts:** `CGOV.sol` (ERC-20 Votes) + `UtilityStaker.sol`

Stake `$CREDX` in the `UtilityStaker` to earn `$CGOV` governance tokens over time. CGOV uses OpenZeppelin's `ERC20Votes` for on-chain DAO governance. Rewards are distributed via a gas-efficient **Synthetix-style O(1)** reward index.

### Tier 3 — Real-Yield Vault (`StakingRewards`)

> **Contract:** `StakingRewards.sol`

Stake `$CGOV` to earn **native POL/MATIC** from the 5% protocol fee on every invoice repayment. Unlike inflationary DeFi protocols, yield is backed by actual business transactions.

```
$CREDX ──stake──► UtilityStaker ──mint──► $CGOV ──stake──► StakingRewards ──earn──► POL/MATIC
                                                                    ▲
                                                    5% of every repayment
```

---

## Smart Contracts

All contracts are written in **Solidity 0.8.26** with optimizer enabled (`200 runs`, `viaIR`), using **OpenZeppelin v5** and custom errors for gas efficiency.

| Contract | Type | Purpose |
|----------|------|---------|
| `InvoiceMarketplace.sol` | Upgradeable (Proxy) | Core escrow: invoice lifecycle, investment, repayment claims, refunds |
| `InvoiceNFT.sol` | ERC-721 | Tokenizes invoices, handles repayment with 5% fee routing |
| `StakingRewards.sol` | Vault | Synthetix-style O(1) MATIC reward distribution for CGOV stakers |
| `UtilityStaker.sol` | Staking | Stake CREDX → earn CGOV (time-weighted reward rate) |
| `CGOV.sol` | ERC-20 Votes | Governance token with on-chain voting support |
| `CredXUtilityToken.sol` | ERC-20 | Protocol access token, admin-minted |

### Key Contract Mechanisms

- **Double-Financing Prevention** — Each invoice is a unique ERC-721 NFT, making duplicate funding impossible
- **Reentrancy Protection** — Custom `nonReentrant` guard on all fund-moving functions
- **Exclusive Investor Mode** — Invoices can be set as private (single investor) or public (open market)
- **Automated Escrow** — Funds flow through the contract; MSMEs receive capital only when fully funded
- **Refund Safety Net** — Investors can reclaim funds if an invoice expires before reaching its funding goal

### Deployed Addresses (Sepolia Testnet)

```
CredXUtilityToken  : 0xA649c1E3Cd062A8363CE8d3FB25B0fE79704870B
CGOV               : 0x5DBF0A01BEC99C53132323f36ee1bDF73B6aAa12
UtilityStaker      : 0xf29d320Fa7D18d5ac8BD6F754272Ef25633eAda8
StakingRewards     : 0x0Fa43Ff7FAe9FFFaC95BEe819e2A13aF9de52df7
InvoiceNFT         : 0x66ed7Ef08364E738678578f09396B11aa8c57aAE
InvoiceMarketplace : 0x3fd09Af8110eF5b490A8A98df9BFFcf22ba4C2A2
```

---

## Features

### For MSMEs
- 📝 Create & mint invoices as NFTs with IPFS-pinned metadata
- 💸 Receive instant working capital when invoices are fully funded
- 📊 On-chain reputation via Trust Score (repaid / total invoices ratio)
- 💬 Real-time chat with investors on invoice listings

### For Investors
- 🛒 Browse & filter the invoice marketplace
- 🔒 Fund public or private (exclusive) invoice opportunities
- 📈 Claim principal + interest upon buyer repayment
- 🛡️ Refund protection on expired unfunded invoices

### For Big Buyers
- ✅ Confirm invoices and repay directly to smart contract escrow
- 🧾 Dynamic success receipt with principal, interest, and protocol fee breakdown

### Platform-Wide
- 🔐 Role-based auth (MSME / Investor / Big Buyer) with email verification
- 👛 MetaMask wallet connection via RainbowKit
- 🏛️ Governance staking dashboard (CREDX → CGOV → MATIC yield)
- 💬 Real-time notifications via Pusher
- 🌙 Dark-themed, animation-rich UI with liquid background effects

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Framework** | Next.js 16 (App Router) |
| **Web3** | Wagmi v2, Viem, RainbowKit, Ethers.js v6 |
| **Smart Contracts** | Solidity 0.8.26, Hardhat, OpenZeppelin v5 |
| **Styling** | Tailwind CSS v4, Shadcn/UI, Framer Motion |
| **Database** | MongoDB (Mongoose) |
| **Decentralized Storage** | IPFS via Pinata |
| **Real-Time** | Pusher (WebSockets) |
| **Email** | Nodemailer (Gmail SMTP) |
| **Auth** | Custom (bcryptjs + cookie-based sessions) |
| **Forms** | React Hook Form + Zod validation |
| **Charts** | Recharts |
| **Analytics** | Vercel Analytics |

---

## Repository Structure

```
CredX/
├── app/                          # Next.js App Router
│   ├── page.tsx                  # Landing page (liquid animation hero)
│   ├── layout.tsx                # Root layout with Web3Provider
│   ├── globals.css               # Global styles
│   ├── auth/                     # Sign in / Sign up pages
│   ├── connect-wallet/           # Wallet connection page
│   ├── dashboard/
│   │   ├── layout.tsx            # Dashboard shell with sidebar
│   │   ├── msme/                 # MSME dashboard
│   │   ├── investor/             # Investor dashboard
│   │   ├── bigbuyer/             # Big Buyer dashboard
│   │   ├── admin/                # Admin dashboard
│   │   ├── my-invoices/          # User's invoice management
│   │   └── governance-staking/   # 3-Tier staking interface
│   ├── demo/                     # Public marketplace demo
│   ├── api/                      # API routes
│   │   ├── auth/                 # Login, signup, email verification
│   │   ├── invoices/             # Invoice CRUD + IPFS pinning
│   │   ├── chat/                 # Real-time messaging
│   │   ├── trust/                # Trust score computation
│   │   ├── notifications/        # Pusher notifications
│   │   └── user/                 # User management
│   ├── about/                    # About page
│   ├── privacy/                  # Privacy policy
│   └── terms/                    # Terms of service
├── contracts/                    # Solidity smart contracts
│   ├── InvoiceMarketplace.sol    # Core escrow & lifecycle
│   ├── InvoiceNFT.sol            # ERC-721 + fee routing
│   ├── StakingRewards.sol        # Real-Yield vault
│   ├── UtilityStaker.sol         # CREDX → CGOV staking
│   ├── CGOV.sol                  # Governance token
│   ├── CredXUtilityToken.sol     # Access token
│   └── ProxyImports.sol          # Proxy pattern imports
├── components/
│   ├── ui/                       # 60+ Shadcn/UI primitives
│   ├── marketplace/              # Invoice cards, filters, chat
│   ├── repayment/                # Success receipt modal
│   ├── dashboard/                # Sidebar navigation
│   ├── msme/                     # Reputation section
│   ├── chat/                     # Chat widget components
│   ├── providers/                # Web3Provider (Wagmi/RainbowKit)
│   ├── AppSidebar.tsx            # Main sidebar
│   └── WalletDropdown.tsx        # Wallet connection dropdown
├── hooks/                        # Custom React hooks
│   ├── use-auth.ts               # Authentication hook
│   ├── useWallet.ts              # Wallet state management
│   └── useIsContractOwner.ts     # Admin detection
├── lib/
│   ├── contracts/                # ABIs, addresses, contract hooks
│   │   ├── addresses.ts          # Deployed contract addresses
│   │   ├── network.ts            # Polygon Amoy chain config
│   │   ├── useInvoiceContract.ts # Marketplace interactions
│   │   ├── useStakingRewards.ts  # Staking vault interactions
│   │   └── ...                   # Per-contract hook + ABI pairs
│   ├── wagmi.ts                  # Wagmi/RainbowKit configuration
│   ├── invoice.ts                # Invoice fetching utilities
│   ├── trustScore.ts             # Trust score computation engine
│   ├── pinata.ts                 # IPFS file/JSON upload via Pinata
│   ├── email.ts                  # Nodemailer transactional emails
│   ├── pusher.ts                 # Pusher real-time config
│   ├── dbConnect.ts              # MongoDB connection singleton
│   └── pendingInvoices.ts        # Pending invoice management
├── models/                       # Mongoose schemas
│   ├── User.ts                   # User (roles, wallet, email verification)
│   ├── ChatMessage.ts            # Real-time chat messages
│   └── UserStats.ts              # Trust score statistics
├── scripts/                      # Deployment scripts
│   ├── DeploymentManager.js      # Full 3-tier system deployer
│   ├── deploy-marketplace-fresh.js
│   └── upgrade-marketplace.js    # Proxy upgrade script
├── middleware.ts                 # Role-based route protection
├── hardhat.config.js             # Hardhat: Sepolia, optimizer, Sourcify
├── tailwind.config.ts            # Tailwind CSS configuration
└── next.config.mjs               # Next.js configuration
```

---

## Getting Started

### Prerequisites

- **Node.js** v20+
- **MetaMask** browser extension (connected to Sepolia or Polygon Amoy Testnet)
- **MongoDB** instance (local or Atlas)

### 1. Clone & Install

```bash
git clone https://github.com/your-org/CredX.git
cd CredX
npm install
```

### 2. Environment Variables

Create a `.env.local` file in the project root:

```env
# MongoDB
MONGODB_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/credx

# Web3
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_walletconnect_project_id
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/YOUR_KEY
SEPOLIA_PRIVATE_KEY=your_deployer_private_key

# IPFS (Pinata)
PINATA_JWT=your_pinata_jwt_token
NEXT_PUBLIC_PINATA_GATEWAY_BASE_URL=https://gateway.pinata.cloud/ipfs/

# Email Verification (Gmail)
GMAIL_USER=your_email@gmail.com
GMAIL_PASS=your_app_password
GMAIL_FROM=no-reply@credx.io

# Real-time (Pusher)
PUSHER_APP_ID=your_pusher_app_id
PUSHER_KEY=your_pusher_key
PUSHER_SECRET=your_pusher_secret
NEXT_PUBLIC_PUSHER_KEY=your_pusher_key

# Verification
ETHERSCAN_API_KEY=your_etherscan_api_key
```

### 3. Run Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) — you'll be greeted by the liquid-animation landing page.

### 4. Compile & Deploy Contracts (Optional)

```bash
# Compile
npm run contracts:compile

# Deploy full 3-tier system to Sepolia
npx hardhat run scripts/DeploymentManager.js --network sepolia

# Update lib/contracts/addresses.ts with the new addresses
```

---

## Protocol Fee Flow

```
  Buyer repays invoice (100 POL)
              │
              ▼
      InvoiceNFT.repayInvoice()
              │
       ┌──────┴──────┐
       │              │
    5 POL          95 POL
       │              │
       ▼              ▼
  StakingRewards  Marketplace
  (CGOV stakers)  (Investors claim)
```

---

## License

This project is licensed under the [MIT License](LICENSE).

---

<p align="center">
  <strong>CredX Protocol</strong> · Built for the Real-World Asset economy
</p>
