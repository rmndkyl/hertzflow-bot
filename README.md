# HertzFlow Testnet Bot

Automated trading & liquidity bot for HertzFlow testnet (BNB Chain). Backed by YZi Labs (Binance Labs).

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
    "privateKey": "0xYOUR_PRIVATE_KEY"
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
Referral link: https://testnet.hertzflow.xyz/referral?ref=OGLYID

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
| ExchangeRouter | `0xc82ceF15311ff3B4a8ab576f43677662378D9F52` |
| SyntheticsRouter | `0xEb460f6DDb55822015b7792854aDd4423f232686` |
| OrderVault | `0x4cDe676F61Dc2F85c83b9404833004B822721c0f` |
| DepositVault | `0x07860Cc65DeB99Cb12d4582a7ae8123030C2d5C1` |
| WithdrawalVault | `0x78c9c8BD4ad8EbC8e65c78B77AD3F19f0b1326b4` |
| HlvRouter | `0x9A8958B6b3B945C71157E39DE969c95231F83181` |
| HlvDepositVault | `0x02cf5def6007e0e247a39571881eda95e0108b29` |
| ReferralStorage | `0xB7812a1399FA6C9D40966F07F4B8f5C88A319F8D` |
| Test USDT | `0x6335881872FEcab922d1d83c6Bae6E27C5a9209c` |

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
