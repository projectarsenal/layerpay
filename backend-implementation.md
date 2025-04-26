# Detailed Backend Implementation Guide

## Tech Stack
- **Runtime**: Node.js
- **Framework**: Express.js
- **Payment Gateway**: Razorpay SDK
- **Blockchain**: Web3.js (Polygon Mumbai)
- **Database**: None (MVP uses in-memory storage; replace with PostgreSQL/MongoDB later)

## Phase 1: Project Setup
### 1.1 Initialize Node.js Project
```bash
mkdir layerpay-backend
cd layerpay-backend
npm init -y
```

### 1.2 Install Dependencies
```bash
npm install express dotenv razorpay web3 cors axios body-parser
npm install -D nodemon typescript @types/express @types/cors
```

### 1.3 Configure TypeScript
```bash
npx tsc --init
```
Update `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "strict": true
  }
}
```

### 1.4 File Structure
```
src/
├── config/
│   ├── razorpay.ts     # Razorpay client setup
│   └── web3.ts         # Web3.js configuration
├── controllers/
│   ├── payment.ts      # Order creation and verification
│   └── webhook.ts      # Razorpay webhook handler
├── routes/
│   └── api.ts          # API endpoints
├── types/
│   └── index.ts        # Type definitions
├── utils/
│   ├── logger.ts       # Logging middleware
│   └── helpers.ts      # Validation/formatting
├── app.ts              # Express app setup
└── server.ts           # Server entry
```

## Phase 2: Core Configuration
### 2.1 Environment Variables (`.env`)
```env
PORT=3001
RAZORPAY_KEY=your_razorpay_key
RAZORPAY_SECRET=your_razorpay_secret
POLYGON_RPC_URL=https://rpc-mumbai.maticvigil.com
CONTRACT_ADDRESS=0x...  # Deployed PaymentLogger contract
PRIVATE_KEY=your_wallet_private_key
```

### 2.2 Razorpay Setup (`config/razorpay.ts`)
```typescript
import Razorpay from "razorpay";

export const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY!,
  key_secret: process.env.RAZORPAY_SECRET!,
});
```

### 2.3 Web3 Setup (`config/web3.ts`)
```typescript
import { Web3 } from "web3";
import PaymentLoggerABI from "../abis/PaymentLogger.json";

const web3 = new Web3(process.env.POLYGON_RPC_URL!);
const contract = new web3.eth.Contract(
  PaymentLoggerABI,
  process.env.CONTRACT_ADDRESS!
);

// Initialize wallet signer
const account = web3.eth.accounts.privateKeyToAccount(
  process.env.PRIVATE_KEY!
);
web3.eth.accounts.wallet.add(account);

export { web3, contract };
```

## Phase 3: API Implementation
### 3.1 Create Order Endpoint (`controllers/payment.ts`)
```typescript
import { Request, Response } from "express";
import { razorpay } from "../config/razorpay";

export const createOrder = async (req: Request, res: Response) => {
  try {
    const { amount } = req.body; // Amount in paise (e.g., ₹10 = 1000 paise)

    const order = await razorpay.orders.create({
      amount: amount,
      currency: "INR",
      payment_capture: 1,
    });

    res.status(200).json(order);
  } catch (error) {
    res.status(500).json({ error: "Failed to create order" });
  }
};
```

### 3.2 Webhook Handler (`controllers/webhook.ts`)
```typescript
import { Request, Response } from "express";
import { contract, web3 } from "../config/web3";
import { razorpay } from "../config/razorpay";

export const handleWebhook = async (req: Request, res: Response) => {
  const { event, payload } = req.body;

  // Verify Razorpay signature (omitted for brevity; use razorpay.webhooks.verify())

  if (event === "payment.captured") {
    try {
      // Log payment to blockchain
      const tx = await contract.methods
        .logPayment(payload.payment.entity.id, payload.payment.entity.amount)
        .send({ from: account.address, gas: 300000 });

      console.log("Payment logged to blockchain:", tx.transactionHash);
      res.status(200).send("OK");
    } catch (error) {
      console.error("Blockchain logging failed:", error);
      res.status(500).send("Error");
    }
  } else {
    res.status(200).send("Unhandled event");
  }
};
```

## Phase 4: Routes & Middleware
### 4.1 API Routes (`routes/api.ts`)
```typescript
import { Router } from "express";
import { createOrder } from "../controllers/payment";
import { handleWebhook } from "../controllers/webhook";

const router = Router();

// Payment routes
router.post("/orders", createOrder);
router.post("/webhook", handleWebhook);

export default router;
```

### 4.2 Express App (`app.ts`)
```typescript
import express from "express";
import cors from "cors";
import bodyParser from "body-parser";
import apiRouter from "./routes/api";
import { logger } from "./utils/logger";

const app = express();

// Middleware
app.use(cors({ origin: "http://localhost:5173" })); // Allow Vite frontend
app.use(bodyParser.json());
app.use(logger); // Custom request logging

// Routes
app.use("/api", apiRouter);

export default app;
```

### 4.3 Logger Middleware (`utils/logger.ts`)
```typescript
import { Request, Response, NextFunction } from "express";

export const logger = (req: Request, _res: Response, next: NextFunction) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
};
```

## Phase 5: Blockchain Integration
### 5.1 Contract ABI (`abis/PaymentLogger.json`)
```json
[
  {
    "inputs": [
      { "internalType": "string", "name": "_paymentId", "type": "string" },
      { "internalType": "uint256", "name": "_amount", "type": "uint256" }
    ],
    "name": "logPayment",
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  }
]
```

### 5.2 Transaction Validation (`utils/helpers.ts`)
```typescript
export const validatePayment = (payload: any): boolean => {
  // Add checks for amount, currency, and payment ID
  return payload.payment.entity.amount > 0;
};
```

## Phase 6: Testing & Deployment
### 6.1 Start Dev Server
Add to `package.json`:
```json
"scripts": {
  "dev": "nodemon src/server.ts",
  "build": "tsc",
  "start": "node dist/server.js"
}
```
Run:
```bash
npm run dev
```

### 6.2 Test with Postman
- **Create Order**:  
  `POST http://localhost:3001/api/orders`  
  Body: `{ "amount": 1000 }`  

- **Simulate Webhook**:  
  Use Razorpay's test webhook payloads or tools like `ngrok` for local testing.

### 6.3 Deploy to Production
1. Build the project:
   ```bash
   npm run build
   ```

2. Deploy to Heroku/AWS:
   - Add `Procfile`:  
     ```procfile
     web: npm start
     ```
   - Set environment variables in hosting platform.

## Key Features Implemented
- Razorpay order creation and webhook handling
- Blockchain transaction logging via Web3.js
- TypeScript support for type safety
- Request logging and error handling
- CORS configuration for frontend integration

This setup ensures scalability (replace in-memory storage with a DB later) and maintainability.