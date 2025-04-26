
# LayerPay 🚀

**LayerPay** is a next-gen payment gateway that combines the simplicity of Web2 payment systems (UPI, Cards) with the trust and transparency of Web3 blockchain. Designed for developers and merchants, it enables instant payment integration with just **3 lines of code** while logging every transaction to an immutable blockchain ledger.

---

## 📌 Table of Contents
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Demo](#-demo)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Deployment](#-deployment)
- [Testing](#-testing)
- [Contributing](#-contributing)
- [Roadmap](#-roadmap)
- [License](#-license)

---

## 🚀 Features

- **3-Line Integration**: Add UPI/Card payments to your app with minimal code.  
- **Blockchain-Verified Logging**: Every payment is recorded on Polygon Mumbai Testnet.  
- **Unified Checkout**: Seamless UI for UPI, Cards, and future payment methods.  
- **Merchant Dashboard**: View real-time, blockchain-verified transactions.  
- **Developer-First**: TypeScript support, modular architecture, and clean SDK.  

---

## 🛠 Tech Stack

| Component       | Tools                                                                 |
|-----------------|-----------------------------------------------------------------------|
| **Frontend**    | React + Vite, Tailwind CSS, Ethers.js, Axios                          |
| **Backend**     | Node.js, Express, Razorpay SDK, Web3.js                               |
| **Contracts**   | Solidity, Hardhat, Polygon Mumbai Testnet                            |
| **Database**    | In-memory storage (MVP)                                               |
| **DevOps**      | GitHub Actions, Vercel/Heroku                                         |

---

## 🎥 Demo

### Live Demo
- **Frontend**: [https://layerpay-demo.vercel.app](https://layerpay-demo.vercel.app)  
- **Contract**: [`0x...` (Polygon Mumbai)](https://mumbai.polygonscan.com/address/0x...)  

### Demo Video
[Watch 2-Minute Demo](https://youtu.be/...)  

---

## 🛠 Installation

### Prerequisites
- Node.js v18+
- MetaMask (with Mumbai Testnet configured)
- Razorpay Test API Keys

### Clone the Repository
```bash
git clone https://github.com/shubhamkumar/layerpay.git
cd layerpay
```

### Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

### Backend Setup
```bash
cd backend
npm install
npm run dev
```

### Contracts Setup
```bash
cd contracts
npm install
npx hardhat compile
```

---

## 🔧 Configuration

### Frontend (`.env`)
```env
VITE_API_URL=http://localhost:3001
VITE_RAZORPAY_KEY=rzp_test_1234567890abcdef
VITE_CONTRACT_ADDRESS=0x1234567890abcdef1234567890abcdef12345678
```

### Backend (`.env`)
```env
RAZORPAY_KEY=rzp_test_1234567890abcdef
RAZORPAY_SECRET=1234567890abcdef1234567890abcdef
POLYGON_RPC_URL=https://rpc-mumbai.maticvigil.com
PRIVATE_KEY=0xabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdef
```

### Contracts (`.env`)
```env
POLYGONSCAN_API_KEY=1234567890abcdef1234567890abcdef
POLYGON_RPC_URL=https://rpc-mumbai.maticvigil.com
PRIVATE_KEY=your_wallet_private_key
```

---

## 🚀 Deployment

### Frontend (Vercel)
1. Link your GitHub repo to Vercel.  
2. Set environment variables in Vercel dashboard.  

### Backend (Heroku)
```bash
heroku create
git push heroku main
```

### Contracts (Polygon Mumbai)
```bash
npx hardhat run scripts/deploy.ts --network mumbai
npx hardhat verify --network mumbai DEPLOYED_CONTRACT_ADDRESS
```

---

## 🧪 Testing

### Frontend
```bash
npm run test  # Runs React Testing Library
```

### Backend
```bash
npm run test  # Tests API endpoints with Jest
```

### Contracts
```bash
npx hardhat test
```

---

## 🤝 Contributing

1. Fork the repository.  
2. Create a feature branch: `git checkout -b feature/your-feature`.  
3. Commit changes: `git commit -m "Add your feature"`.  
4. Push to the branch: `git push origin feature/your-feature`.  
5. Open a pull request.  

---

## 📅 Roadmap

| Phase          | Goal                                      |
|----------------|-------------------------------------------|
| **Q3 2024**    | Support Paytm, Stripe gateways            |
| **Q4 2024**    | Launch NFT payment receipts               |
| **Q1 2025**    | Expand to Southeast Asia (GrabPay)        |

---

## 📜 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgements

- Razorpay for payment gateway APIs.  
- Polygon for blockchain infrastructure.  
- Hardhat for smart contract tooling.  
```

---

### 🔍 **How to Use This README**
1. Replace placeholder URLs (e.g., `your-username`, `0x...`).  
2. Add screenshots/demo GIFs under the [Demo](#-demo) section.  
3. Customize the roadmap and tech stack as needed.  

**You’re all set!** This README covers setup, configuration, and future plans while keeping it concise and hackathon-ready. 🎉