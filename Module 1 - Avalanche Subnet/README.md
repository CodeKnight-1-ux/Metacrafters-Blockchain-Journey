# Avalanche Subnet - Metacrafters 

Welcome to the **DeFi Kingdom Clone** project! In this thrilling blockchain-based game, you will create a decentralized finance (DeFi) gaming experience, similar to **DeFi Kingdoms**, built on the **Avalanche Network**. Players can collect digital assets, participate in battles, explore new realms, and earn rewards through various in-game activities. The project utilizes **EVM Subnet** to deploy smart contracts on a custom chain with low fees and custom tokens, allowing for a highly customizable gaming world.

## Project Overview

### Objective
Create a DeFi Kingdom clone on the **Avalanche** blockchain using a custom **EVM Subnet**. This project focuses on the fundamentals of subnet development, setting up a native in-game currency, and deploying smart contracts to manage game activities like battling, exploring, and trading.

### Key Features
- **EVM Subnet**: Build on Avalanche’s custom blockchain for optimal performance and low fees.
- **ERC20 Tokens**: In-game currency and assets will be represented using custom ERC20 tokens.
- **Vault Smart Contract**: Manage deposits and withdrawals securely, rewarding players for their participation.
- **Game Activities**: Develop game logic for battling, exploring, and trading within the DeFi Kingdom clone.
- **User Experience**: Enhance the experience with leaderboards, achievements, and wallet integration via Avalanche's native wallet.

## Setup Instructions

### Prerequisites
Before starting, ensure you have the following tools and knowledge:
- **Unix Computer**: Either MacOS or Linux.
- **Solidity**: Familiarity with Solidity programming language for writing smart contracts.
- **Remix IDE**: Browser-based IDE for smart contract development.
- **Metamask**: A browser wallet to connect to the Avalanche EVM subnet.
- **Web Browser**: For deploying and interacting with your contracts.

### Step-by-Step Guide

1. **Set Up Your EVM Subnet**  
   Use the Avalanche CLI to create your custom EVM subnet on the Avalanche network. This subnet will act as your own blockchain tailored to your game.

2. **Define Your Native Currency**  
   Create a custom ERC20 token contract in Solidity. This token will be used as the in-game currency for purchases, trading, and rewarding players.

3. **Connect to Metamask**  
   Add your custom subnet to Metamask by following the Avalanche documentation. Ensure your custom network is selected.

4. **Deploy Smart Contracts**  
   Use Remix IDE to deploy your smart contracts.  
   - Deploy the **ERC20 token** smart contract to define your in-game currency.
   - Deploy the **Vault contract** to handle deposits, withdrawals, and token rewards.

### Example Smart Contracts

#### ERC20 Token Contract
This contract defines the custom token used in your game.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ERC20 {
    uint public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;
    string public name = "In-Game Token";
    string public symbol = "IGT";
    uint8 public decimals = 18;

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function transfer(address recipient, uint amount) external returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function approve(address spender, uint amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint amount) external returns (bool) {
        allowance[sender][msg.sender] -= amount;
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }

    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    function burn(uint amount) external {
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }
}
```

The ERC20 contract implements a basic version of the ERC20 token standard, which is used for creating fungible tokens on the Ethereum Virtual Machine (EVM). These tokens are transferable, fungible (meaning one token is identical to another), and can be traded or used in various decentralized applications (dApps).

Here’s a breakdown of the key components and functions:

1. **State Variables:**
   - `totalSupply`: Tracks the total number of tokens in circulation.
   - `balanceOf`: Maps each address to its token balance.
   - `allowance`: Maps owner addresses to spender addresses and their allowed amount of tokens to transfer.
   - `name`, `symbol`, `decimals`: Defines the token’s name, symbol, and number of decimals (usually set to 18).

2. **Events:**
   - `Transfer`: Emitted when tokens are transferred between addresses.
   - `Approval`: Emitted when an address approves another address to spend tokens on its behalf.

3. **Functions:**
   - `transfer(address recipient, uint amount)`: Allows a user to send tokens to another address. It deducts the sender's balance and adds the amount to the recipient's balance.
   - `approve(address spender, uint amount)`: Allows the caller to approve another address (a spender) to spend a certain amount of tokens on their behalf.
   - `transferFrom(address sender, address recipient, uint amount)`: Allows a spender to transfer tokens from the owner’s account to another account, using the approved allowance.
   - `mint(uint amount)`: Allows the caller to create (mint) new tokens, increasing the total supply and the caller's balance.
   - `burn(uint amount)`: Allows the caller to destroy (burn) tokens, reducing the total supply and their balance.

**Key Points:**
- **Minting** and **burning** are mechanisms to increase or decrease the total supply of tokens. This is often used in games for reward distribution or asset management.
- `allowance` and `approve` let users delegate spending power to other smart contracts or users, a core feature in decentralized finance (DeFi) systems.
- All transfers and approvals emit events for tracking token movement, which can be monitored by blockchain explorers or dApps.

---

#### Vault Contract
This contract manages token deposits and withdrawals.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IERC20 {
    function transferFrom(address sender, address recipient, uint amount) external returns (bool);
    function balanceOf(address account) external view returns (uint);
}

contract Vault {
    IERC20 public immutable token;
    uint public totalSupply;
    mapping(address => uint) public balanceOf;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function deposit(uint _amount) external {
        uint shares = (_amount * totalSupply) / token.balanceOf(address(this));
        balanceOf[msg.sender] += shares;
        totalSupply += shares;
        token.transferFrom(msg.sender, address(this), _amount);
    }

    function withdraw(uint _shares) external {
        uint amount = (_shares * token.balanceOf(address(this))) / totalSupply;
        balanceOf[msg.sender] -= _shares;
        totalSupply -= _shares;
        token.transfer(msg.sender, amount);
    }
}
```

The Vault contract is designed to handle deposits and withdrawals of tokens, allowing users to stake their tokens in exchange for shares (representing ownership in the vault). This is common in DeFi applications for liquidity pooling, staking, and yield farming.

Here’s a breakdown of the key components and functions:

1. **State Variables:**
   - `token`: This is the address of the ERC20 token that the vault will manage. It refers to the token contract (usually deployed beforehand).
   - `totalSupply`: Tracks the total amount of shares issued by the vault (representing staked tokens).
   - `balanceOf`: Maps addresses to their share balances (representing how much of the vault’s total token pool they own).

2. **Constructor:**
   - `constructor(address _token)`: This initializes the Vault contract and sets the ERC20 token that the Vault will manage. The token’s contract address is passed as `_token`.

3. **Private Functions:**
   - `_mint(address _to, uint _shares)`: Mints new shares and assigns them to the specified address. This is called when users deposit tokens into the vault.
   - `_burn(address _from, uint _shares)`: Burns shares from the specified address, reducing their ownership in the vault. This is called when users withdraw tokens from the vault.

4. **Public Functions:**
   - `deposit(uint _amount)`: Allows users to deposit a specific amount of ERC20 tokens into the vault. In exchange, they receive a proportional number of shares based on the current balance of the vault and the total supply of shares. The calculation ensures that each share represents a proportional amount of the total tokens held by the vault.
   
     - **Formula for minting shares:**  
       If it’s the first deposit, the shares minted equal the amount deposited. Otherwise, shares minted are proportional to the total token balance in the vault:
       ```
       shares = (amount * totalSupply) / token.balanceOf(address(this))
       ```

   - `withdraw(uint _shares)`: Allows users to withdraw tokens by burning a specified amount of their shares. The vault calculates how much of the token balance corresponds to the burned shares and transfers that amount to the user.

     - **Formula for withdrawing tokens:**  
       When a user withdraws, the amount of tokens they get is proportional to the total supply of shares:
       ```
       amount = (shares * token.balanceOf(address(this))) / totalSupply
       ```

**Key Points:**
- **Deposit and Withdraw:** When users deposit tokens, they receive shares representing their ownership in the vault. When they withdraw, they exchange shares for tokens. The shares represent a proportional share of the vault's total assets, ensuring fairness when the vault’s balance fluctuates.
- **Minting and Burning Shares:** Shares are dynamically minted when users deposit and burned when they withdraw, maintaining a 1:1 representation of their stake in the vault’s token pool.
- The contract ensures fairness with proportionate rewards based on the vault’s total assets and the user's shareholding.

---

### Final Steps
Once your contracts are deployed:
- **Test your contracts**: Use Remix IDE or your own frontend to interact with the smart contracts.
- **Launch game logic**: Set up liquidity pools, explore features like battling and trading.
- **Reward system**: Distribute rewards using your custom token and Vault contract.

## Tools Used
- **Avalanche CLI**: For deploying and managing your custom EVM subnet.
- **Solidity**: For writing the ERC20 token and Vault contracts.
- **Remix IDE**: To compile, deploy, and interact with your smart contracts.
- **Metamask**: To connect your wallet to the Avalanche subnet and interact with the game.

---

### **Real-World Use Case:**

In a DeFi Kingdoms clone, the **ERC20 contract** could represent the in-game currency (e.g., "Kingdom Token"). Players earn, trade, and spend this currency within the game. The **Vault contract** could be used for staking mechanics, where players deposit their tokens in exchange for shares. Over time, they can withdraw their tokens with added rewards or interest, representing the growth of their assets within the kingdom.

By using these contracts, you can implement key DeFi mechanics like token creation, trading, and staking, which are fundamental to building a blockchain-based game with financial incentives.

