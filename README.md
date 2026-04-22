# 🤖 Multi-Chain Crypto Wallet Monitor Bot

A Telegram bot that monitors crypto wallet addresses across multiple EVM-compatible blockchains and Solana, sending real-time notifications for incoming and outgoing transactions — including native token transfers and ERC-20/token transfers.

---

## Features

- **Multi-chain support** — monitor wallets across 35+ blockchain networks simultaneously
- **Real-time alerts** — get notified of new transactions as they happen
- **Token transfer detection** — tracks both native coin transfers and ERC-20/token transfers
- **Smart deduplication** — avoids sending duplicate notifications for the same transaction
- **Token type recognition** — distinguishes stablecoins, wrapped tokens, DeFi tokens, and meme coins with distinct icons
- **Per-chat wallet management** — each Telegram chat/user manages their own list of monitored wallets
- **On-demand transaction check** — view recent transactions for any address without adding it to monitoring
- **Solana support** — monitors Solana wallets via the Solscan API alongside EVM chains

---

## Supported Chains

| Chain | Symbol | Chain ID |
|---|---|---|
| Ethereum Mainnet | ETH | 1 |
| BNB Smart Chain | BNB | 56 |
| Polygon Mainnet | MATIC | 137 |
| Arbitrum One | ETH | 42161 |
| Arbitrum Nova | ETH | 42170 |
| Avalanche C-Chain | AVAX | 43114 |
| Base Mainnet | BASE | 8453 |
| Optimism (OP Mainnet) | OP | 10 |
| zkSync Mainnet | ZKS | 324 |
| Scroll Mainnet | SCR | 534352 |
| Linea Mainnet | LIN | 59144 |
| Blast Mainnet | BLAST | 81457 |
| Mantle Mainnet | MNT | 5000 |
| Celo Mainnet | CELO | 42220 |
| Gnosis | GNO | 100 |
| Moonbeam | GLMR | 1284 |
| Moonriver | MOVR | 1285 |
| Cronos | CRO | 25 |
| Sonic Mainnet | S | 146 |
| Taiko Mainnet | TK | 167000 |
| Berachain Mainnet | BERA | 80094 |
| ApeChain Mainnet | APE | 33139 |
| Unichain Mainnet | UNI | 130 |
| World Mainnet | WORLD | 480 |
| Fraxtal Mainnet | frxETH | 252 |
| Swellchain Mainnet | SW | 1923 |
| BitTorrent Chain | BTT | 199 |
| HyperEVM | HYPE | 999 |
| Memecore Mainnet | MEME | 4352 |
| Sophon Mainnet | SOPH | 50104 |
| Katana Mainnet | KAT | 747474 |
| opBNB Mainnet | opBNB | 204 |
| WEMIX3.0 Mainnet | WEMIX | 1111 |
| Xai Mainnet | XAI | 660279 |
| **Solana** | SOL | solana |

---

## Prerequisites

- Python 3.9+
- A Telegram Bot Token (from [@BotFather](https://t.me/BotFather))
- An [Etherscan API key](https://etherscan.io/apis) (used for all EVM chains via the v2 API)
- A [Solscan API key](https://pro-api.solscan.io/) (for Solana monitoring)

---

## Installation

1. **Clone the repository**

```bash
git clone https://github.com/your-username/multichain-wallet-monitor-bot.git
cd multichain-wallet-monitor-bot
```

2. **Create a virtual environment and install dependencies**

```bash
python -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate
pip install python-telegram-bot aiohttp
```

---

## Configuration

> ⚠️ **Never hardcode API keys in source code.** Use environment variables to keep your credentials secure.

Set the following environment variables before running the bot:

```bash
export BOT_TOKEN="your_telegram_bot_token"
export ETHERSCAN_API_KEY="your_etherscan_api_key"
export SOLSCAN_API_KEY="your_solscan_api_key"
```

On Windows (Command Prompt):

```cmd
set BOT_TOKEN=your_telegram_bot_token
set ETHERSCAN_API_KEY=your_etherscan_api_key
set SOLSCAN_API_KEY=your_solscan_api_key
```

Then update the bottom of `TelegramCrytoBot.py` to read from environment variables:

```python
import os

BOT_TOKEN = os.environ["BOT_TOKEN"]
ETHERSCAN_API_KEY = os.environ["ETHERSCAN_API_KEY"]
SOLSCAN_API_KEY = os.environ["SOLSCAN_API_KEY"]
```

Alternatively, use a `.env` file with the `python-dotenv` package:

```
BOT_TOKEN=your_telegram_bot_token
ETHERSCAN_API_KEY=your_etherscan_api_key
SOLSCAN_API_KEY=your_solscan_api_key
```

---

## Running the Bot

```bash
python TelegramCrytoBot.py
```

The bot will start polling for Telegram updates and begin the background monitoring loop.

---

## Bot Commands

| Command | Description | Example |
|---|---|---|
| `/start` | Start the bot and view welcome message | `/start` |
| `/add <address> [chain_id]` | Add a wallet to monitor (defaults to Ethereum) | `/add 0xABC...123 56` |
| `/remove <address> [chain_id]` | Stop monitoring a wallet | `/remove 0xABC...123 56` |
| `/list` | List all wallets you are monitoring | `/list` |
| `/check <address> [chain_id]` | View the 5 most recent transactions for any address | `/check 0xABC...123 137` |
| `/chains` | Show all supported chains and their IDs | `/chains` |
| `/help` | Show the help message | `/help` |

If no `chain_id` is provided, **Ethereum Mainnet (ID: 1)** is used by default.

---

## Notification Format

When a new transaction is detected, the bot sends a message like this:

```
🔔 NEW TRANSACTION DETECTED

Network: BNB Smart Chain
📥 INCOMING
💰 Amount: 1,200.00 USDT
🏠 From: 0xSenderAddress...
🏠 To: 0xYourAddress...
🔗 Hash: 0xTxHash...
🕒 Time: 2025-01-15 10:30:00 UTC
💵 Token: Tether USD (USDT)
🏠 Contract: 0xContractAddress...

[View on Explorer]
```

Token type icons:
- 💵 Stablecoins (USDT, USDC, DAI, etc.)
- 🔄 Wrapped tokens (WETH, WBTC, etc.)
- 🏛️ DeFi tokens (UNI, AAVE, LINK, etc.)
- 🐕 Meme tokens (DOGE, SHIB, PEPE, etc.)
- 🪙 All other ERC-20 tokens
- 🔔 Native coin transfers (ETH, BNB, MATIC, etc.)

---

## How It Works

1. **Monitoring loop** — a background async task polls each watched wallet every ~5 seconds, fetching the latest 5 normal transactions and 5 token transfers via Etherscan's API.
2. **Deduplication** — the bot records the hash of the most recently seen transaction per wallet. A notification is only sent when a newer transaction hash is detected.
3. **Token priority** — when a transaction appears in both the normal and token transfer lists (which is common for ERC-20 transfers), the token transfer record is preferred since it carries richer metadata (token name, symbol, decimals, contract address).
4. **Rate limiting** — a 2-second delay between address checks and a 5-second cycle delay help stay within API rate limits.

---

## Project Structure

```
TelegramCrytoBot.py
│
├── SUPPORTED_CHAINS         # Dict of chain IDs → name, symbol, explorer URL
├── Transaction (dataclass)  # Unified data model for a single transaction
├── MultiChainEtherscanAPI   # Async API client for Etherscan (all EVM chains) and Solscan
│   ├── get_latest_transactions()   # Fetches native coin transfers
│   ├── get_token_transactions()    # Fetches ERC-20/token transfers
│   └── get_all_transactions()      # Merges and deduplicates both
└── MultiChainWalletMonitor  # Telegram bot logic and monitoring orchestration
    ├── Command handlers     # /start, /add, /remove, /list, /check, /chains, /help
    ├── monitor_loop()       # Background polling loop
    ├── format_transaction_message()  # Formats notification text
    └── send_transaction_notification()  # Dispatches alerts to Telegram chats
```

---

## Security Notes

- **Rotate any API keys that have been exposed** in source code or version control immediately.
- Add `.env` to your `.gitignore` to avoid accidentally committing credentials.
- Consider restricting your Etherscan API key to specific IP addresses in the Etherscan dashboard.
- The bot stores monitored wallets **in memory only** — data is lost on restart. For persistence, consider adding a database (e.g., SQLite or Redis).

---

## Limitations

- Wallet data is stored **in memory** and does not persist across restarts.
- The monitoring interval is approximately every 5–7 seconds per wallet depending on the number of addresses being watched.
- Etherscan free-tier API keys have rate limits; heavy usage may require a paid plan.
- Solana monitoring requires a Solscan Pro API key.

---

## License

MIT License — feel free to use, modify, and distribute.
