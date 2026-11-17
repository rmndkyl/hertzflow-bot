# HertzFlow Bot

**Versi:** 1.0.0 · **Harga:** Rp 120.000 (~$8) · **Kontak pembelian:** [@rmndkyl](https://t.me/rmndkyl)

Automated trading & liquidity bot for HertzFlow testnet (BNB Chain). Backed by YZi Labs (Binance Labs).

## Kebutuhan

| Item | Detail |
|---|---|
| Runtime | Node.js 18+ |
| Wallet | EVM / sesuai chain proyek (kamu sendiri) |
| Proxy | tidak perlu |
| Captcha | tidak perlu |
| Akses | akun proyek yang kamu kelola sendiri |

## Pembayaran

E-Wallet: DANA / GoPay / ShopeePay · Bank: BCA / Seabank · Crypto: contact for wallet address.
Pembeli menghubungi [@rmndkyl](https://t.me/rmndkyl), kirim bukti transfer, lalu terima license key.

---

## Features

- 🔌 Auto-claim 100 test USDT (mint function)
- 📈 Auto-trade: LONG/SHORT on 202+ markets with **10% SL + 10% TP** auto-attached
- 💧 Pool deposit/withdraw (LP liquidity provision)
- 🏦 Vault (HLV) deposit/withdraw (HertzFlow Liquidity Vaults)
- 🔗 Auto-bind referral code
- 📊 Trade history + pool/vault overview tracking
- ⚡ Multicall pattern: sendWnt + sendTokens + createOrder (+ SL/TP orders)
- 🕐 Random delay (3-7s) between operations to prevent rate limiting
- 🔮 Oracle price integration for SL/TP trigger price calculation

## Setup

```bash
npm install
```

Create `accounts.json`:
```json
[
  {
    "label": "wallet1",
  }
]
```

## Usage

```bash
# Daily full run (faucet + referral + trades with SL/TP + pool + vault)
node index.js --daily

# Info mode (balance + referral)
node index.js

# List all markets
node index.js --markets

# List all LP pools
node index.js --pools

# List all vaults (HLV)
node index.js --vaults

# Claim faucet only
node index.js --faucet

# Check trade history
node index.js --history
```

## Referral

Default referral code: `OGLYID`

## How It Works

### Trading (with SL/TP)
1. **Faucet**: Calls `mint(address, 100e18)` on HertzFlow's test USDT contract
2. **Trade**: Uses ExchangeRouter multicall pattern:
   - `sendWnt(OrderVault, executionFee)` for keeper gas
   - `sendTokens(USDT, OrderVault, collateral)` for collateral
   - `createOrder(params)` with MarketIncrease (orderType=2)
   - `createOrder(params)` with LimitDecrease (orderType=5) for Take Profit (10%)
   - `createOrder(params)` with StopLossDecrease (orderType=6) for Stop Loss (10%)
   - Each SL/TP order has its own `sendWnt` for execution fee
   - SL/TP trigger prices fetched from oracle API (1e12 precision)

### Pool (LP Liquidity)
3. **Pool Deposit**: sendWnt + sendTokens → multicall(ExchangeRouter, [sendWnt, sendTokens, createDeposit])
4. **Pool Withdraw**: Transfer LP tokens → WithdrawalVault → multicall(ExchangeRouter, [sendWnt, createWithdrawal])
   - Pool selection: BONK/USD > ETH/USD > BTC/USD (fallback chain due to deposit cap)

### Vault (HLV - HertzFlow Liquidity Vault)
5. **Vault Deposit**: sendWnt + sendTokens → multicall(HlvRouter, [sendWnt, sendTokens, createHlvDeposit])
6. **Vault Withdraw**: Transfer HLV tokens → WithdrawalVault → multicall(HlvRouter, [sendWnt, createHlvWithdrawal])

### Other
7. **Referral**: Calls `bindReferrer(bytes32)` on ReferralStorage contract
8. **Approval**: USDT approved for SyntheticsRouter (base Router that handles `pluginTransfer`)

## Tech Stack

- Node.js + viem (EVM interactions)
- @hertzflow/sdk-v2 (protocol SDK)
- BSC Testnet (Chain ID 97)

## Contract Addresses (BSC Testnet)

| Contract | Address |
|----------|---------|

## API Endpoints

| Service | URL |
|---------|-----|
| Oracle Prices | `https://oracle-aggregator.hertzflow.xyz/api/v1/latestPrice?get_all=true` |
| Markets/Prices | `https://data-statistics-query.testnet.htzfl.link/api/v1/bsc/markets` |
| User Trades | `https://data-statistics-query.testnet.htzfl.link/api/v1/bsc/user/trades?UserAddress=0x...` |
| Pools List | `https://data-statistics-query.testnet.htzfl.link/api/v1/bsc/pools/list` |
| Pools Overview | `https://data-statistics-query.testnet.htzfl.link/api/v1/bsc/pools/overview` |
| Pool Detail | `https://data-statistics-query.testnet.htzfl.link/api/v1/bsc/pools/{address}` |
| Vaults | `https://data-statistics-query.testnet.htzfl.link/api/v1/bsc/vaults` |

## License

MIT

## 💰 Harga

| Lisensi | Harga | Fitur |
|---------|-------|-------|
| **Single** | Rp 120.000 / ~$8 | 1 device, lifetime |
| **Bundle (3 bot)** | Rp 120.000 / ~$16 | Pilih 3 bot, hemat 20% |
| **Pro (semua bot)** | Rp 120.000 / ~$70 | Semua bot + update, hemat 30% |

**Cara beli:** Chat [@OG_LYID_bot](https://t.me/OG_LYID_bot) di Telegram → /buy

**Bayar pakai:**
- 🇮🇩 DANA / GoPay / ShopeePay / BCA / Seabank
- 🌐 Crypto (USDT/SOL/ETH)

## 🔧 Troubleshooting

| Masalah | Solusi |
|---------|--------|
| `Cannot find module` | Jalankan `npm install` di folder bot |
| `insufficient balance` | Pastikan wallet punya cukup token untuk gas |
| `429 Too Many Requests` | Tunggu 30-60 detik, bot auto-retry |
| Bot diam/tidak jalan | Cek `config.json` — pastikan API key valid |
| License invalid | Chat [@OG_LYID_bot](https://t.me/OG_LYID_bot) → /verify |

**Butuh bantuan?** Join [Discord](https://discord.gg/ZK2NVkfSNn) atau chat [@layerairdrop](https://t.me/layerairdrop)
