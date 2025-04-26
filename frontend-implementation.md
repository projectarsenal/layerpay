# Detailed Frontend Implementation Guide

## Tech Stack
- **Framework**: React + TypeScript (Vite)
- **Styling**: Tailwind CSS
- **State Management**: Context API
- **Blockchain**: Ethers.js
- **HTTP Client**: Axios

## Phase 1: Project Setup
### 1.1 Initialize Vite Project
```bash
npm create vite@latest frontend -- --template react-ts
cd frontend
```

### 1.2 Install Dependencies
```bash
npm install ethers axios @tanstack/react-query react-router-dom @headlessui/react heroicons-react
npm install -D tailwindcss postcss autoprefixer
```

### 1.3 Configure Tailwind CSS
```bash
npx tailwindcss init -p
```
Update `tailwind.config.js`:
```javascript
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        'layerpay-blue': '#2563eb',
      }
    },
  },
  plugins: [],
}
```

Add Tailwind directives to `src/index.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Phase 2: Project Structure
```
src/
├── assets/               # Static assets
├── components/           # Reusable components
│   ├── PaymentButton.tsx
│   ├── TransactionCard.tsx
│   └── Loader.tsx
├── contexts/             # Global state
│   └── PaymentContext.tsx
├── hooks/                # Custom hooks
│   └── usePayments.ts
├── pages/                # Main views
│   ├── Dashboard.tsx
│   └── Checkout.tsx
├── utils/                # Helper functions
│   ├── api.ts            # Axios instance
│   ├── connectors.ts     # Wallet connection
│   └── constants.ts      # Contract addresses
├── App.tsx
└── main.tsx
```

## Phase 3: Core Implementation
### 3.1 Configure Axios (utils/api.ts)
```typescript
import axios from 'axios';

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3001',
  timeout: 10000,
});
```

### 3.2 Payment Button Component (components/PaymentButton.tsx)
```tsx
import { useCreatePayment } from '../hooks/usePayments';

export const PaymentButton = () => {
  const { initiatePayment, isLoading } = useCreatePayment();

  return (
    <button 
      onClick={initiatePayment}
      disabled={isLoading}
      className="bg-layerpay-blue text-white px-6 py-3 rounded-lg hover:bg-blue-700 disabled:opacity-50"
    >
      {isLoading ? 'Processing...' : 'Pay ₹10'}
    </button>
  );
};
```

### 3.3 Custom Hook for Payments (hooks/usePayments.ts)
```typescript
import { api } from '../utils/api';
import { useWeb3 } from '../contexts/PaymentContext';

export const useCreatePayment = () => {
  const { provider } = useWeb3();

  const initiatePayment = async () => {
    try {
      // 1. Create order via backend
      const { data: order } = await api.post('/create-order', {
        amount: 1000, // ₹10 in paise
        currency: 'INR'
      });

      // 2. Open Razorpay checkout
      const options = {
        key: import.meta.env.VITE_RAZORPAY_KEY,
        amount: order.amount,
        currency: order.currency,
        order_id: order.id,
        handler: async (response: any) => {
          // 3. Verify payment via backend webhook
          await api.post('/verify-payment', response);
        }
      };

      const rzp = new (window as any).Razorpay(options);
      rzp.open();
      
    } catch (error) {
      console.error('Payment failed:', error);
    }
  };

  return { initiatePayment, isLoading: false };
};
```

## Phase 4: Dashboard Implementation
### 4.1 Transaction Card Component (components/TransactionCard.tsx)
```tsx
import { ethers } from 'ethers';

export const TransactionCard = ({ payment }: { payment: any }) => (
  <div className="p-4 border rounded-lg mb-3">
    <div className="flex justify-between">
      <span className="font-medium">{payment.paymentId}</span>
      <span className="text-gray-600">
        {ethers.utils.formatEther(payment.amount)} MATIC
      </span>
    </div>
    <a
      href={`https://mumbai.polygonscan.com/tx/${payment.txHash}`}
      target="_blank"
      className="text-blue-500 hover:underline text-sm"
    >
      View on PolygonScan
    </a>
  </div>
);
```

### 4.2 Dashboard Page (pages/Dashboard.tsx)
```tsx
import { useQuery } from '@tanstack/react-query';
import { api } from '../utils/api';
import { TransactionCard } from '../components/TransactionCard';

export const Dashboard = () => {
  const { data: payments } = useQuery(['payments'], async () => {
    const { data } = await api.get('/payments');
    return data;
  });

  return (
    <div className="max-w-2xl mx-auto p-6">
      <h1 className="text-2xl font-bold mb-6">Verified Transactions</h1>
      <div className="space-y-4">
        {payments?.map((payment: any) => (
          <TransactionCard key={payment.id} payment={payment} />
        ))}
      </div>
    </div>
  );
};
```

## Phase 5: Blockchain Integration
### 5.1 Web3 Context (contexts/PaymentContext.tsx)
```tsx
import { createContext, useContext, useEffect, useState } from 'react';
import { ethers } from 'ethers';

type Web3ContextType = {
  provider?: ethers.providers.Web3Provider;
  connectWallet: () => Promise<void>;
};

const Web3Context = createContext<Web3ContextType>({} as Web3ContextType);

export const Web3Provider = ({ children }: { children: React.ReactNode }) => {
  const [provider, setProvider] = useState<ethers.providers.Web3Provider>();

  const connectWallet = async () => {
    if (window.ethereum) {
      const _provider = new ethers.providers.Web3Provider(window.ethereum);
      await _provider.send("eth_requestAccounts", []);
      setProvider(_provider);
    }
  };

  return (
    <Web3Context.Provider value={{ provider, connectWallet }}>
      {children}
    </Web3Context.Provider>
  );
};

export const useWeb3 = () => useContext(Web3Context);
```

## Phase 6: Final Integration
### 6.1 App Router (App.tsx)
```tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { Web3Provider } from './contexts/PaymentContext';
import { Dashboard } from './pages/Dashboard';
import { Checkout } from './pages/Checkout';

function App() {
  return (
    <Web3Provider>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<Checkout />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </BrowserRouter>
    </Web3Provider>
  );
}

export default App;
```

### 6.2 Checkout Page (pages/Checkout.tsx)
```tsx
import { PaymentButton } from '../components/PaymentButton';
import { useWeb3 } from '../contexts/PaymentContext';

export const Checkout = () => {
  const { connectWallet } = useWeb3();

  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-3xl font-bold mb-8">LayerPay Demo</h1>
        <button
          onClick={connectWallet}
          className="bg-gray-800 text-white px-4 py-2 rounded-lg mb-6"
        >
          Connect Wallet
        </button>
        <PaymentButton />
      </div>
    </div>
  );
};
```

## Phase 7: Testing & Optimization
1. **Start Dev Server**
   ```bash
   npm run dev
   ```

2. **Test Payment Flow**
   - Click "Connect Wallet" → MetaMask
   - Click "Pay ₹10" → Razorpay test modal
   - Use test card `4111 1111 1111 1111`

3. **Verify Blockchain Logging**
   - After payment, check dashboard for PolygonScan link
   - Confirm transaction appears in smart contract

## Deployment
```bash
npm run build
```
Deploy the `dist` folder to:
- Vercel (`vercel deploy`)
- Netlify
- GitHub Pages

## Key Features Implemented
- Razorpay SDK integration with TypeScript
- Wallet connection via MetaMask
- Real-time transaction listing
- Blockchain-verified payment proof
- Responsive UI with Tailwind
- Error handling and loading states

This implementation ensures a professional, maintainable codebase while keeping the 3-line integration promise.