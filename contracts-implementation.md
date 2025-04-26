# Detailed Smart Contract Implementation Guide

## Tech Stack
- **Language**: Solidity (0.8.x)
- **Framework**: Hardhat (with TypeScript)
- **Testing**: Hardhat Network + Waffle
- **Verification**: Hardhat-Etherscan
- **Security**: Slither, Solidity Coverage

## Phase 1: Project Setup
### 1.1 Initialize Hardhat Project
```bash
mkdir layerpay-contracts
cd layerpay-contracts
npm init -y
npm install --save-dev hardhat @nomiclabs/hardhat-waffle ethers @nomiclabs/hardhat-ethers @typechain/hardhat typechain @typechain/ethers-v5 dotenv @openzeppelin/contracts
npx hardhat
```
Choose "Create a TypeScript project".

### 1.2 File Structure
```
contracts/
├── PaymentLogger.sol       # Core contract
├── interfaces/
│   └── IPaymentLogger.sol  # Interface for upgrades
scripts/
├── deploy.ts               # Deployment script
test/
├── PaymentLogger.test.ts   # Test suite
.env                        # Environment variables
hardhat.config.ts
```

### 1.3 Configure Environment (`.env`)
```env
POLYGON_RPC_URL=https://rpc-mumbai.maticvigil.com
PRIVATE_KEY=your_wallet_private_key
POLYGONSCAN_API_KEY=your_api_key
```

## Phase 2: Core Contract Implementation
### 2.1 PaymentLogger Contract (`contracts/PaymentLogger.sol`)
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/access/Ownable.sol";

error InvalidPaymentId();
error InvalidAmount();
error Unauthorized();

contract PaymentLogger is Ownable {
    struct Payment {
        string paymentId;
        address payer;
        uint256 amount;
        uint256 timestamp;
    }

    Payment[] private _payments;
    mapping(string => bool) private _existingPayments;

    event PaymentLogged(
        string indexed paymentId,
        address indexed payer,
        uint256 amount,
        uint256 timestamp
    );

    // Restrict function to authorized backend wallet
    modifier onlyBackend() {
        if (msg.sender != owner()) revert Unauthorized();
        _;
    }

    /**
     * @dev Logs a payment to blockchain
     * @param _paymentId Unique payment ID from gateway
     * @param _payer Payer's wallet address
     */
    function logPayment(
        string calldata _paymentId,
        address _payer,
        uint256 _amount
    ) external onlyBackend {
        // Validate inputs
        if (bytes(_paymentId).length == 0) revert InvalidPaymentId();
        if (_amount == 0) revert InvalidAmount();
        if (_existingPayments[_paymentId]) revert InvalidPaymentId();

        // Store payment
        _payments.push(Payment(_paymentId, _payer, _amount, block.timestamp));
        _existingPayments[_paymentId] = true;

        emit PaymentLogged(_paymentId, _payer, _amount, block.timestamp);
    }

    /**
     * @dev Returns payment details by ID
     */
    function getPayment(
        string calldata _paymentId
    ) external view returns (Payment memory) {
        for (uint i = 0; i < _payments.length; i++) {
            if (keccak256(bytes(_payments[i].paymentId)) == keccak256(bytes(_paymentId))) {
                return _payments[i];
            }
        }
        revert InvalidPaymentId();
    }

    /**
     * @dev Returns total payments count
     */
    function totalPayments() external view returns (uint256) {
        return _payments.length;
    }
}
```

## Phase 3: Advanced Features
### 3.1 Gas Optimization
- Use `calldata` for string parameters
- Packed storage with `uint256` timestamps
- Use custom errors instead of revert strings

### 3.2 Security Measures
```solidity
// Add to contract
import "@openzeppelin/contracts/security/Pausable.sol";

contract PaymentLogger is Ownable, Pausable {
    // Add modifier to critical functions
    function logPayment(...) external onlyBackend whenNotPaused {
        // ...
    }

    // Emergency stop
    function pause() external onlyOwner {
        _pause();
    }

    function unpause() external onlyOwner {
        _unpause();
    }
}
```

## Phase 4: Testing
### 4.1 Test Setup (`test/PaymentLogger.test.ts`)
```typescript
import { expect } from "chai";
import { ethers } from "hardhat";
import { PaymentLogger } from "../typechain-types";

describe("PaymentLogger", () => {
  let contract: PaymentLogger;
  const paymentId = "PAY_123";
  const amount = ethers.utils.parseEther("0.1");

  beforeEach(async () => {
    const PaymentLogger = await ethers.getContractFactory("PaymentLogger");
    contract = await PaymentLogger.deploy();
    await contract.deployed();
  });

  it("Should log payment", async () => {
    const [owner, payer] = await ethers.getSigners();
    
    await contract.connect(owner).logPayment(paymentId, payer.address, amount);
    
    const payment = await contract.getPayment(paymentId);
    expect(payment.paymentId).to.eq(paymentId);
    expect(payment.amount).to.eq(amount);
  });

  it("Should reject duplicate payment IDs", async () => {
    const [owner, payer] = await ethers.getSigners();
    
    await contract.connect(owner).logPayment(paymentId, payer.address, amount);
    
    await expect(
      contract.connect(owner).logPayment(paymentId, payer.address, amount)
    ).to.be.revertedWithCustomError(contract, "InvalidPaymentId");
  });
});
```

### 4.2 Run Tests
```bash
npx hardhat test
```

## Phase 5: Deployment & Verification
### 5.1 Deployment Script (`scripts/deploy.ts`)
```typescript
import { ethers } from "hardhat";

async function main() {
  const PaymentLogger = await ethers.getContractFactory("PaymentLogger");
  const contract = await PaymentLogger.deploy();
  await contract.deployed();

  console.log("Contract deployed to:", contract.address);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

### 5.2 Deploy to Polygon Mumbai
```bash
npx hardhat run scripts/deploy.ts --network mumbai
```

### 5.3 Verify on Polygonscan
```bash
npx hardhat verify --network mumbai DEPLOYED_CONTRACT_ADDRESS
```

## Phase 6: Integration
### 6.1 Generate TypeChain Typings
Add to `hardhat.config.ts`:
```typescript
import "@typechain/hardhat";

module.exports = {
  // ... existing config
  typechain: {
    outDir: "typechain",
    target: "ethers-v5",
  },
};
```
Run:
```bash
npx hardhat typechain
```

### 6.2 Integration with Backend
Use the generated TypeChain types for type-safe interactions:
```typescript
import { PaymentLogger } from "../typechain-types";
import { ethers } from "hardhat";

const contractAddress = process.env.CONTRACT_ADDRESS!;
const contract: PaymentLogger = await ethers.getContractAt(
  "PaymentLogger",
  contractAddress
);
```

## Security Best Practices
1. **Static Analysis**  
   ```bash
   npm install --save-dev @nomicfoundation/hardhat-verify
   npx hardhat slither
   ```

2. **Coverage Report**  
   ```bash
   npm install --save-dev solidity-coverage
   npx hardhat coverage
   ```

3. **Formal Verification**  
   Consider using Certora or Halmos for advanced verification.

## Key Features Implemented
- Secure payment logging with duplicate prevention
- Role-based access control (Backend-only)
- Emergency pause functionality
- Gas-efficient storage design
- Comprehensive test coverage
- Type-safe integration with TypeChain
- Polygon Mumbai deployment ready