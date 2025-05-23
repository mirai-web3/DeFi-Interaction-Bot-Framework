# DeFi Interaction Bot Framework

A standardized, modular framework for creating DeFi interaction bots that automate blockchain operations across various testnets and mainnets. Built with consistency, reliability, and extensibility in mind.

![Framework Banner](https://img.shields.io/badge/DeFi-Framework-blue?style=for-the-badge&logo=ethereum)
![Node.js](https://img.shields.io/badge/Node.js-16+-green?style=for-the-badge&logo=node.js)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

## 🎯 Framework Overview

This framework provides a solid foundation for building DeFi automation bots with:

- **🔧 Modular Architecture** - Reusable components for common DeFi operations
- **🎨 Consistent UI/UX** - Standardized terminal display with emoji-based status indicators  
- **🛡️ Robust Error Handling** - Intelligent retry mechanisms and proxy management
- **📊 Comprehensive Tracking** - Detailed interaction results and performance monitoring
- **🚀 Protocol Agnostic** - Easy adaptation to different blockchain protocols
- **📱 TypeScript Ready** - Full JSDoc documentation and TS compatibility

## 🏗️ Architecture

### Core Components

```
DeFi Bot Framework/
├── 🎯 Base Classes (Extend These)
│   ├── BaseAPIClient        # Protocol API interactions
│   ├── BaseTransactionHandler # Blockchain transactions
│   └── InteractionTracker   # Results tracking & display
├── 🔧 Core Utilities (Use As-Is)
│   ├── Logger              # Consistent terminal output
│   ├── FileManager         # Configuration file loading
│   └── ProxyManager        # Intelligent proxy rotation
└── ⚙️ Configuration
    ├── CONFIG              # Network & contract settings
    ├── PARAMS              # Interaction parameters
    └── File-based config   # privatekeys.txt, wallets.txt, etc.
```

### Supported Interaction Types

| Category | Emoji | Examples |
|----------|-------|----------|
| **Authentication** | 🔐 | API login, wallet signature |
| **Faucet Operations** | 🚰 | Testnet token claims |
| **Token Operations** | 💸 | Transfers, approvals |
| **DeFi Protocols** | 🌾 | Staking, farming, liquidity |
| **Trading** | 💱 | Swaps, arbitrage |
| **Wrapping** | 🔄 | ETH↔WETH, token wrapping |
| **Unwrapping** | 🔓 | WETH↔ETH, token unwrapping |

## 🚀 Quick Start

### Prerequisites

- Node.js 16+ installed
- Basic understanding of DeFi protocols
- Private keys for wallet interactions

### Installation

1. **Clone or copy the framework:**
```bash
git clone <your-framework-repo>
cd defi-interaction-bot-framework
```

2. **Install dependencies:**
```bash
npm install ethers axios https-proxy-agent random-useragent
```

3. **Create configuration files:**
```bash
# Private keys (one per line, must start with 0x)
touch privatekeys.txt

# Target wallet addresses (optional)
touch wallets.txt

# Proxy list (optional)
touch proxies.txt

# Custom configuration (optional)
touch config.txt
```

### Basic Usage

```javascript
const { Logger, BaseTransactionHandler } = require('./framework');

// Extend the base classes for your protocol
class MyProtocolBot extends BaseTransactionHandler {
  // Add your protocol-specific logic here
}

// Initialize and run
const logger = new Logger();
const bot = new MyProtocolBot(wallet, logger);
```

## 🔧 Customization Guide

### 1. Network Configuration

Update the `CONFIG` object for your target protocol:

```javascript
const CONFIG = {
  network: {
    name: 'Arbitrum Sepolia',           // Display name
    chainId: 421614,                    // Chain ID
    rpcUrl: 'https://sepolia-rollup.arbitrum.io/rpc'
  },
  contract: {
    TOKEN: '0x...',                     // Main protocol token
    STAKING: '0x...',                   // Staking contract
    ROUTER: '0x...',                    // DEX router
    FARM: '0x...'                       // Farming contract
  },
  api: {
    baseUrl: 'https://api.yourprotocol.xyz',
    authEndpoint: '/auth',
    faucetEndpoint: '/faucet'
  }
};
```

### 2. Interaction Parameters

Customize interaction counts and amounts:

```javascript
const PARAMS = {
  // Transaction amounts
  TRANSFER_AMOUNT: '0.001',           // Amount per transfer
  STAKE_AMOUNT: '0.01',               // Amount per stake
  
  // Interaction counts  
  TRANSFER_COUNT: 10,                 // Transfers per wallet
  STAKE_COUNT: 5,                     // Stakes per wallet
  
  // Protocol-specific
  SLIPPAGE_TOLERANCE: 0.01,           // 1% slippage
  
  // Randomization
  RANDOMIZE: true,                    // Enable randomization
  VARIATION: 0.1                      // ±10% variation
};
```

### 3. Extending API Client

Create protocol-specific API interactions:

```javascript
class YourProtocolAPI extends BaseAPIClient {
  /**
   * Authenticate with your protocol
   */
  async authenticate() {
    try {
      this.logger.step('Authenticating with protocol...');
      
      const message = await this.getAuthMessage();
      const signature = await this.wallet.signMessage(message);
      
      const response = await this.makeRequest('POST', '/auth', {
        address: this.wallet.address,
        signature: signature
      });
      
      this.authToken = response.data.token;
      this.logger.success('Authentication successful');
      return true;
    } catch (error) {
      this.logger.error(`Authentication failed: ${error.message}`);
      return false;
    }
  }
  
  /**
   * Claim protocol rewards
   */
  async claimRewards() {
    try {
      if (!this.authToken && !(await this.authenticate())) {
        return false;
      }
      
      const response = await this.makeRequest('POST', '/rewards/claim');
      return response.data.success;
    } catch (error) {
      this.logger.error(`Reward claim failed: ${error.message}`);
      return false;
    }
  }
}
```

### 4. Extending Transaction Handler

Add blockchain-specific operations:

```javascript
class YourTransactionHandler extends BaseTransactionHandler {
  /**
   * Initialize protocol contracts
   */
  initializeContracts() {
    super.initializeContracts(); // Call parent initialization
    
    // Add your protocol contracts
    this.contracts.staking = new ethers.Contract(
      CONFIG.contract.STAKING,
      [
        'function stake(uint256 amount)',
        'function unstake(uint256 amount)',
        'function getReward()',
        'function balanceOf(address) view returns (uint256)'
      ],
      this.wallet
    );
  }
  
  /**
   * Stake tokens
   */
  async stake(amount, index) {
    this.logger.tx(`Stake ${index+1}: ${amount} tokens`);
    
    return this.retryableInteraction('Stake', async () => {
      // Check balances
      const balances = await this.getBalances();
      const required = ethers.parseEther(amount);
      
      if (balances.token.raw < required) {
        this.logger.warn(`Insufficient balance for staking`);
        return false;
      }
      
      // Approve if needed
      await this.ensureApproval(CONFIG.contract.TOKEN, CONFIG.contract.STAKING, required);
      
      // Execute stake
      const tx = await this.contracts.staking.stake(required);
      this.logger.tx(`Tx hash: 0x${tx.hash.slice(2, 6)}...${tx.hash.slice(-4)}`);
      
      await tx.wait();
      this.logger.success(`Stake ${index+1} completed`);
      return true;
    });
  }
  
  /**
   * Helper function to ensure token approval
   */
  async ensureApproval(tokenAddress, spenderAddress, amount) {
    const allowance = await this.contracts.token.allowance(
      this.wallet.address, 
      spenderAddress
    );
    
    if (allowance < amount) {
      this.logger.step('Approving tokens...');
      const approveTx = await this.contracts.token.approve(spenderAddress, ethers.MaxUint256);
      await approveTx.wait();
      this.logger.success('Token approval completed');
    }
  }
}
```

### 5. Custom Wallet Processing

Define your protocol's interaction flow:

```javascript
async function processWallet(wallet, proxy, targetAddresses, walletIndex, totalWallets, stats) {
  const shortAddress = `${wallet.address}`;
  logger.operation(walletIndex, totalWallets, shortAddress, "STARTING INTERACTIONS");
  
  // Initialize handlers
  const txHandler = new YourTransactionHandler(wallet, logger);
  const apiClient = new YourProtocolAPI(wallet, logger, proxy);
  
  // Track results
  const walletResult = {
    authSuccess: false,
    rewardsClaimed: false,
    stakeCount: 0,
    swapCount: 0
  };
  
  // STEP 1: Authentication
  logger.operation(walletIndex, totalWallets, shortAddress, "STEP 1: AUTHENTICATION");
  const authSuccess = await apiClient.authenticate();
  stats.recordInteraction('auth', authSuccess);
  walletResult.authSuccess = authSuccess;
  
  await sleep(...CONFIG.timing.betweenInteractions);
  
  // STEP 2: Staking Operations
  logger.operation(walletIndex, totalWallets, shortAddress, "STEP 2: STAKING");
  let stakeCount = 0;
  
  for (let i = 0; i < PARAMS.STAKE_COUNT; i++) {
    const success = await txHandler.stake(PARAMS.STAKE_AMOUNT, i);
    stats.recordInteraction('stakes', success);
    if (success) stakeCount++;
    
    await sleep(...CONFIG.timing.betweenInteractions);
  }
  
  walletResult.stakeCount = stakeCount;
  
  // Add more steps as needed...
  
  // Record final results
  stats.recordWalletResult(wallet.address, walletResult);
  stats.incrementWallet();
  logger.success(`Wallet ${walletIndex + 1} processing completed`);
}
```

## 📊 Results Display

The framework provides a standardized results screen:

```
====================================================
  INTERACTION RESULTS SUMMARY - by miraiweb3
====================================================

📋 WALLET DETAILS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
0x1234...5678   🔐✅  🚰✅  💸10/10  🌾5/5   💱3/3
0x9abc...def0   🔐✅  🚰❌  💸8/10   🌾4/5   💱3/3
0xfedc...ba98   🔐❌  🚰❌  💸0/10   🌾0/5   💱0/3

⏱  Waiting for next cycle in: 29m 37s
```

### Emoji Reference

| Emoji | Meaning | Usage |
|-------|---------|-------|
| 🔐 | Authentication | Login success/failure |
| 🚰 | Faucet | Testnet token claims |
| 💸 | Transfers | Token transfers |
| 🌾 | Farming/Staking | DeFi yield operations |
| 💱 | Swaps | Token exchanges |
| 🔄 | Wrapping | Token wrapping operations |
| 🔓 | Unwrapping | Token unwrapping operations |
| 🌊 | Liquidity | LP operations |
| ✅ | Success | General success indicator |
| ❌ | Failure | General failure indicator |

## 📁 File Structure

### Required Files

```
your-protocol-bot/
├── index.js              # Your main bot file
├── package.json          # Dependencies
├── privatekeys.txt       # Private keys (required)
├── wallets.txt           # Target addresses (optional)
├── proxies.txt           # Proxy list (optional)
├── config.txt            # Custom config (optional)
└── README.md             # Protocol-specific documentation
```

### Configuration File Formats

**privatekeys.txt:**
```
0x1234567890123456789012345678901234567890123456789012345678901234
0xabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcd
```

**wallets.txt:**
```
0x742d35Cc6635C0532925a3b8d6A9e6CC84c9dE9C
0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
```

**proxies.txt:**
```
http://user:pass@proxy1.example.com:8080
http://proxy2.example.com:3128
socks5://user:pass@proxy3.example.com:1080
```

**config.txt:**
```
MAX_GAS_PRICE=50000000000
CUSTOM_ENDPOINT=https://custom-rpc.example.com
ENABLE_LOGGING=true
```

## 🔧 Advanced Configuration

### Custom Timing

```javascript
const CONFIG = {
  timing: {
    betweenInteractions: [3000, 8000],    // 3-8 seconds between interactions
    betweenWallets: [10000, 30000],       // 10-30 seconds between wallets
    cycleInterval: 60,                    // 60 minutes between cycles
    requestTimeout: 45000                 // 45 second API timeout
  }
};
```

### Display Options

```javascript
const CONFIG = {
  display: {
    clearBetweenSteps: true,              // Clear screen between steps
    showStepSummary: true,                // Show completed steps summary
    compactMode: false,                   // Compact display mode
    showTransactionDetails: true          // Show detailed TX info
  }
};
```

### Error Handling

```javascript
const PARAMS = {
  MAX_RETRIES: 5,                         // Maximum retry attempts
  RETRY_DELAY_BASE: 3000,                 // Base retry delay (ms)
  GAS_MULTIPLIER: 1.5,                    // Gas limit safety multiplier
  SLIPPAGE_TOLERANCE: 0.02                // 2% slippage tolerance
};
```

## 🛠️ Best Practices

### 1. Protocol Research
- Study the protocol's documentation thoroughly
- Test interactions on testnet first
- Understand rate limits and restrictions
- Check for protocol-specific requirements

### 2. Error Handling
- Always implement proper balance checks
- Handle network congestion gracefully
- Provide clear error messages to users
- Log failures for debugging

### 3. Security
- Never hardcode private keys
- Use environment variables or secure files
- Implement proper input validation
- Test with small amounts first

### 4. Performance
- Use appropriate delays between operations
- Implement intelligent proxy rotation
- Monitor success rates and adjust parameters
- Optimize gas usage

### 5. User Experience
- Maintain consistent emoji usage
- Provide clear progress indicators
- Include helpful error messages
- Show meaningful statistics

## 🐛 Troubleshooting

### Common Issues

**"No private keys found"**
- Ensure `privatekeys.txt` exists and contains valid keys
- Check that keys start with `0x`
- Verify file permissions

**"Insufficient balance for transaction"**
- Check wallet has enough native tokens for gas
- Verify token balances are sufficient
- Consider adjusting transaction amounts

**"Transaction failed after retries"**
- Check network connectivity
- Verify contract addresses are correct
- Ensure gas settings are appropriate
- Check for protocol-specific restrictions

**"API authentication failed"**
- Verify API endpoints are correct
- Check wallet signature is working
- Ensure protocol requirements are met

### Debug Mode

Enable detailed logging by setting environment variables:

```bash
DEBUG=true node your-bot.js
```

Or modify the logger configuration:

```javascript
const logger = new Logger({
  logToFile: true,
  logDir: './debug-logs'
});
```

## 🔗 Integration Examples

### Popular DeFi Protocols

The framework can be adapted for various protocols:

- **Uniswap/SushiSwap**: DEX interactions, liquidity provision
- **Aave/Compound**: Lending and borrowing operations  
- **Curve**: Stable coin swaps and yield farming
- **Balancer**: Multi-token pools and arbitrage
- **Yearn**: Vault deposits and yield optimization
- **Synthetix**: Synthetic asset trading
- **1inch**: DEX aggregation and optimal routing

### Cross-Chain Support

Extend for multiple networks:

```javascript
const NETWORKS = {
  ETHEREUM: {
    chainId: 1,
    rpcUrl: 'https://eth-mainnet.g.alchemy.com/v2/...'
  },
  ARBITRUM: {
    chainId: 42161,
    rpcUrl: 'https://arb-mainnet.g.alchemy.com/v2/...'
  },
  POLYGON: {
    chainId: 137,
    rpcUrl: 'https://polygon-mainnet.g.alchemy.com/v2/...'
  }
};
```

## 📚 API Reference

### Core Classes

#### Logger
- `info(message)` - Info logging
- `success(message)` - Success logging  
- `error(message)` - Error logging
- `warn(message)` - Warning logging
- `tx(message)` - Transaction logging
- `step(message)` - Step logging

#### BaseAPIClient
- `makeRequest(method, endpoint, data, headers)` - HTTP requests
- `authenticate()` - Protocol authentication
- `claimFaucet()` - Faucet claiming

#### BaseTransactionHandler  
- `getBalances()` - Get wallet balances
- `retryableInteraction(name, fn)` - Retry wrapper
- `getRandomizedAmount(amount)` - Amount randomization
- `transfer(address, index)` - Token transfers

#### InteractionTracker
- `recordInteraction(type, success)` - Record interaction
- `recordWalletResult(address, results)` - Record wallet results
- `displayResults()` - Show results summary

## 🤝 Contributing

When extending the framework:

1. **Maintain Consistency**: Follow established patterns and emoji usage
2. **Document Changes**: Add JSDoc comments for new methods
3. **Test Thoroughly**: Test on testnet before mainnet deployment
4. **Share Improvements**: Consider contributing back to the framework

## 📄 License

MIT License - see LICENSE file for details.

## 🙏 Acknowledgments

- Built with [ethers.js](https://docs.ethers.org/) for blockchain interactions
- Developed by [miraiweb3](https://github.com/mirai-web3)
- Inspired by the DeFi community's automation needs

---

**Ready to build your next DeFi interaction bot?** 🚀

Start by copying the framework, customizing the configuration for your target protocol, and extending the base classes with your specific logic. The consistent patterns will ensure your bot feels familiar to users while being optimized for your protocol's unique requirements.
