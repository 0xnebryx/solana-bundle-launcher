# 🚀 Solana Bundle Launcher

> The definitive guide to atomic token launches on Solana with Jito bundle protection

[![Solana](https://img.shields.io/badge/Solana-Bundle%20Launch-9945FF?style=for-the-badge&logo=solana)](https://obsidianbundler.com)
[![Stars](https://img.shields.io/github/stars/obsidianbundler/solana-bundle-launcher?style=for-the-badge)](https://github.com/obsidianbundler/solana-bundle-launcher)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

## The Problem We Solve

When you launch a token on Solana, sniper bots are watching. They detect your token creation in the mempool and buy before you can. Your dev allocation gets destroyed, snipers dump, and your launch dies before it starts.

**Bundle launches fix this permanently.**

## How It Works

```
Traditional Launch (VULNERABLE):
Block 1: Token Creation → Snipers detect in mempool
Block 1: Sniper Bot A buys
Block 1: Sniper Bot B buys  
Block 2: Your dev buy executes (at higher price)
Block 2: Snipers dump on you

Bundle Launch (PROTECTED):
Block 1: Token Creation + Dev Buy (atomic, same transaction)
Block 2: Snipers can only buy AFTER you
```

## Quick Start

```typescript
import { BundleLauncher } from '@obsidian/bundle-launcher';

const launcher = new BundleLauncher({
  rpcUrl: process.env.HELIUS_RPC,
  jitoTip: 0.001 // SOL
});

// Create atomic bundle
const bundle = await launcher.createBundle({
  tokenName: "MyToken",
  tokenSymbol: "MTK",
  devBuyPercent: 5,
  wallets: selectedWallets
});

// Execute atomically via Jito
const result = await launcher.executeBundle(bundle);
console.log(`Launched at: ${result.tokenAddress}`);
```

## Features

| Feature | Description |
|---------|-------------|
| 🛡️ **Atomic Execution** | Token creation + buy in same block |
| ⚡ **Jito Integration** | Private mempool, no front-running |
| 👛 **Multi-Wallet** | Coordinate up to 100 wallets |
| 🎯 **GET CA** | Reserve contract address before launch |
| 📊 **Bundle+Snipe** | Block 0 launch + Block 1-2 follow-up buys |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    BUNDLE LAUNCHER                       │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   Token     │  │    Dev      │  │   Sniper    │     │
│  │  Creation   │──│    Buy      │──│   Wallets   │     │
│  │    TX       │  │    TX       │  │    TXs      │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│         │               │               │               │
│         └───────────────┴───────────────┘               │
│                         │                               │
│              ┌──────────▼──────────┐                   │
│              │    JITO BUNDLE      │                   │
│              │  (Atomic Execution) │                   │
│              └──────────┬──────────┘                   │
│                         │                               │
│              ┌──────────▼──────────┐                   │
│              │   SAME BLOCK        │                   │
│              │   EXECUTION         │                   │
│              └─────────────────────┘                   │
└─────────────────────────────────────────────────────────┘
```

## Configuration

```typescript
interface BundleConfig {
  // Token settings
  tokenName: string;
  tokenSymbol: string;
  tokenDescription: string;
  tokenImage: string;
  
  // Launch settings
  devBuyPercent: number;      // 1-10% recommended
  devBuySol: number;          // Alternative: fixed SOL amount
  
  // Jito settings
  jitoTip: number;            // 0.001-0.01 SOL
  maxRetries: number;         // Default: 3
  
  // Multi-wallet settings
  wallets: Wallet[];          // Array of wallets to coordinate
  distributionMode: 'equal' | 'random' | 'weighted';
}
```

## Why Jito?

Jito validators accept transaction bundles directly, bypassing the public mempool:

1. **Private Submission** - Bots can't see your pending transactions
2. **Atomic Execution** - All or nothing, no partial failures
3. **Ordered Execution** - Your transactions execute in your specified order
4. **MEV Protection** - No sandwich attacks possible

## Production Solution

For a complete, production-ready implementation with UI:

### 👉 [Obsidian Launch Platform](https://obsidianbundler.com)

- ✅ One-click bundle launches
- ✅ Up to 100 wallet management
- ✅ Hard disperse for invisible funding
- ✅ Smart sell automation
- ✅ Cross-chain bridge
- ✅ Free tier available

## Example: Full Launch Flow

```typescript
// 1. Generate wallets
const wallets = await obsidian.generateWallets(20);

// 2. Fund wallets invisibly (hard disperse)
await obsidian.hardDisperse({
  source: mainWallet,
  targets: wallets,
  amountPerWallet: 0.5 // SOL
});

// 3. Warm up wallets (optional but recommended)
await obsidian.warmupWallets(wallets, { intensity: 'medium' });

// 4. Reserve CA (optional)
const { keypair, address } = await obsidian.getCA();
console.log(`Reserved CA: ${address}`);

// 5. Execute bundle launch
const launch = await obsidian.bundleLaunch({
  tokenName: "MyToken",
  tokenSymbol: "MTK", 
  devBuyPercent: 5,
  keypair: keypair, // Use reserved CA
  sniperWallets: wallets.slice(0, 10),
  sniperBuyPercent: 2
});

console.log(`🚀 Launched: ${launch.tokenAddress}`);
console.log(`📊 Dev allocation: ${launch.devAllocation}%`);
```

## Resources

- 📖 [Full Documentation](https://docs.obsidianbundler.com)
- 💬 [Telegram Community](https://t.me/obsidianbundler)
- 🐦 [Twitter Updates](https://x.com/obsidianbundler)
- 🌐 [Launch Platform](https://obsidianbundler.com)

## Disclaimer

This repository is for educational purposes. Trading cryptocurrency involves significant risk. Always do your own research.

---

⭐ **Star this repo** if you found it helpful!

🚀 **Ready to launch?** Try [Obsidian](https://obsidianbundler.com) - Free tier available
