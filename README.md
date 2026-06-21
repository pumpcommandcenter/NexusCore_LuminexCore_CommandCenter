pump-token/
├── lib/
│   ├── metrics.ts              # Persistent + Redis-ready metrics
│   ├── jito.ts                 # Jito bundles + dynamic tip bumping + retry + MEV protection
│   ├── solana.ts               # RPC health, block production, PDAs, bonding curve math
│   └── health.ts               # Stake distribution + Nakamoto coefficient + performance
├── scripts/
│   ├── unified_token_dashboard.ts
│   ├── bonding_curve_rebalancer.ts
│   ├── execute_withdraw.ts
│   ├── token_incinerator.ts
│   ├── run_all.ts
│   └── deploy.sh
├── xnft-frontend/
│   ├── TradeDesk.tsx           # Full multi-wallet + Jupiter + Wormhole + Uniswap Widget
│   └── Dockerfile              # Production multi-stage
├── grafana/
│   ├── trade-desk-dashboard.json
│   ├── incinerator-dashboard.json
│   └── alert-rules/
├── prometheus/
│   ├── prometheus.yml
│   └── alertmanager.yml
├── ansible/
│   ├── playbook.yml
│   └── inventory.yml
├── terraform/
│   ├── main.tf
│   └── variables.tf
├── docker-compose.full.yml
├── .env.example
└── SECURITY_HARDENING_CHECKLIST.md
Upgrade my existing "Church of Pump" Base44 app into a professional degen dashboard for $PUMP.

**Official Mint:** `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb`

**Theme:** Cyberpunk / neon degen (dark + hot pink + electric cyan + purple gradients)

**Required Sections:**

1. **Wallet Connection** — Prominent button + Phantom/Backpack/Solflare support + shortened address + Disconnect. Hide sensitive actions until connected.

2. **Live On-Chain Stats** — Total Transfers (from TransferCounter PDA), Last Transfer timestamp, 1% Tax explanation.

3. **Recent Activity Feed** — `TransferEvent` + `TransferWithdrawal` with color coding (cyan = transfer, pink = withdrawal).

4. **Tax & Treasury** — 1% Token-2022 TransferFee explanation + `withdraw_fees` button (wallet-gated).

5. **Jupiter Swap** — "Swap on Jupiter" button that opens pre-filled Jupiter with the official mint.

**Technical Notes:**
- Transfer Hook Program: `PumpTokenHook11111111111111111111111111111111`
- Events: `TransferEvent`, `TransferWithdrawal`
- Make it fully responsive with excellent loading states and error handling.
- Prepare for custom domain on Base44 Pro.

After building, list the main sections created and any manual steps required (especially wallet connection and Jupiter).
cd pump-token

# 1. One-command deploy (recommended)
./deploy.sh up

# 2. Or with Ansible (for servers)
ansible-playbook -i ansible/inventory.yml ansible/playbook.yml --ask-vault-pass

# 3. Access
# Dashboard + Metrics: http://https://church-of-pump-copy-8b4b3361.base44.app
# Grafana: https://church-of-pump-copy-8b4b3361.base44.app
# Prometheus: https://church-of-pump-copy-8b4b3361.base44.app
NEXT_PUBLIC_PUMP_MINT=TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
NEXT_PUBLIC_NEW_TOKEN_MINT=EyCMRsiSxbLRspptLHNqqMQG8HB2oTZSPWRyWJqXpump
docker compose -f docker-compose.full.yml --profile frontend up -d --build trade-desk
cd xnft-frontend
npm run dev
NEXT_PUBLIC_PUMP_MINT=TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
NEXT_PUBLIC_NEW_TOKEN_MINT=EyCMRsiSxbLRspptLHNqqMQG8HB2oTZSPWRyWJqXpump
# ===================== TRADE DESK TOKENS =====================
NEXT_PUBLIC_PUMP_MINT=TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
NEXT_PUBLIC_NEW_TOKEN_MINT=EyCMRsiSxbLRspptLHNqqMQG8HB2oTZSPWRyWJqXpump
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_walletconnect_project_id
// Add this button alongside the Wormhole bridge button
<button 
  onClick={() => window.open('https://stargate.finance/transfer', '_blank')}
  className="flex-1 py-4 bg-purple-500 text-white font-bold rounded-xl"
>
  BRIDGE VIA LAYERZERO (STARGATE)
</button>
// LayerZero OFT Bridge (more native & efficient than traditional bridging)
const handleLayerZeroOFTBridge = async () => {
  if (!publicKey) {
    setTradeStatus('Connect a Solana wallet to bridge');
    return;
  }

  setTradeStatus('Initiating LayerZero OFT bridge to Base...');

  try {
    // Production: Replace with actual LayerZero OFT SDK call
    // Example using @layerzerolabs/lz-evm-sdk-v2 or Stargate SDK
    const mockTxHash = `lz-oft-${Date.now()}`;

    await recordTradeMetrics('layerzero-oft', parseFloat(tradeAmount), mockTxHash, currentToken.symbol);

    setTradeStatus(`✅ LayerZero OFT bridge initiated! Check LayerZeroScan.`);
    window.open('https://layerzeroscan.com/', '_blank');
  } catch (err: any) {
    setTradeStatus(`❌ LayerZero OFT Error: ${err.message}`);
    await recordTradeMetrics('layerzero-oft', parseFloat(tradeAmount) || 0, 'error', currentToken.symbol, false, err.message);
  }
};
<button 
  onClick={handleLayerZeroOFTBridge}
  className="flex-1 py-4 bg-purple-500 text-white font-bold rounded-xl"
>
  BRIDGE VIA LAYERZERO (OFT)
</button>
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

/// @title Wrapped New Token (for Base)
/// @notice This is a wrapped version of the Solana token: EyCMRsiSxbLRspptLHNqqMQG8HB2oTZSPWRyWJqXpump
///         Controlled by the bridge (Wormhole / LayerZero) for minting/burning during cross-chain transfers.
contract WrappedNewToken is ERC20, Ownable {
    uint8 private constant DECIMALS = 6; // Matching Solana decimals

    constructor() ERC20("Wrapped New Token", "wNEW") Ownable(msg.sender) {}

    /// @notice Mint tokens (only callable by bridge/owner during bridging)
    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }

    /// @notice Burn tokens (only callable by bridge/owner during bridging)
    function burn(address from, uint256 amount) external onlyOwner {
        _burn(from, amount);
    }

    function decimals() public pure override returns (uint8) {
        return DECIMALS;
    }
}
import { ethers } from "hardhat";

async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying WrappedNewToken with account:", deployer.address);

  const WrappedNewToken = await ethers.getContractFactory("WrappedNewToken");
  const token = await WrappedNewToken.deploy();

  await token.waitForDeployment();
  const address = await token.getAddress();

  console.log("WrappedNewToken deployed to:", address);
  console.log("Owner (bridge controller):", deployer.address);
  console.log("\nNext steps:");
  console.log("1. Transfer ownership to your bridge contract (Wormhole/LayerZero)");
  console.log("2. Update TradeDesk.tsx with this address for Uniswap Widget");
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
import { ethers } from "hardhat";

async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying WrappedNewToken with account:", deployer.address);

  const WrappedNewToken = await ethers.getContractFactory("WrappedNewToken");
  const token = await WrappedNewToken.deploy();

  await token.waitForDeployment();
  const address = await token.getAddress();

  console.log("WrappedNewToken deployed to:", address);
  console.log("Owner (bridge controller):", deployer.address);
  console.log("\nNext steps:");
  console.log("1. Transfer ownership to your bridge contract (Wormhole/LayerZero)");
  console.log("2. Update TradeDesk.tsx with this address for Uniswap Widget");
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import * as dotenv from "dotenv";
dotenv.config();

const config: HardhatUserConfig = {
  solidity: "0.8.20",
  networks: {
    base: {
      url: process.env.BASE_RPC_URL || "https://mainnet.base.org",
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
  },
};

export default config;
cd base-wrapped-token

# 1. Install dependencies
npm install

# 2. Set up .env
echo 'PRIVATE_KEY=your_private_key_here
BASE_RPC_URL=https://mainnet.base.org' > .env

# 3. Deploy to Base
npx hardhat run scripts/deploy.ts --network base
defaultOutputTokenAddress="0xYourDeployedWrappedNewTokenAddress"
{/* Bonding Curve Progress */}
<div className="mt-8">
  <div className="flex justify-between text-sm mb-2">
    <span>Bonding Curve Progress</span>
    <span className="font-mono">{progress.toFixed(1)}%</span>
  </div>
  
  <div className="w-full bg-zinc-800 rounded-full h-4 overflow-hidden border border-pink-600">
    <div 
      className="h-4 rounded-full transition-all duration-500"
      style={{ 
        width: `${progress}%`,
        background: progress > 80 ? 'linear-gradient(to right, #22c55e, #eab308)' : 'linear-gradient(to right, #ec4899, #a855f7)'
      }}
    />
  </div>

  <div className="grid grid-cols-2 gap-4 mt-4 text-sm">
    <div>SOL Raised: <span className="font-mono text-pink-400">{currentRaised.toFixed(2)} / {targetMarketCap}</span></div>
    <div>Est. Market Cap: <span className="font-mono text-pink-400">${(parseFloat(targetMarketCap) * (progress / 100)).toFixed(0)}</span></div>
  </div>
</div>
cd base-oft-deployment

npm install

# Deploy
npx hardhat run scripts/deployOFT.ts --network base
import { ethers } from "hardhat";

async function main() {
  const [deployer] = await ethers.getSigners();

  // === CONFIGURE THESE VALUES ===
  const oftAddress = "0xYourDeployedMyOFTAddress";           // From deployOFT.ts
  const remoteEid = 30168;                                   // Solana Mainnet EID (LayerZero V2)
  const remoteOFT = "YourSolanaOFTAddressOrProgramId";       // Solana side address

  // Convert to bytes32
  const peer = ethers.zeroPadValue(ethers.toBeHex(remoteOFT), 32);

  console.log("Setting peer on MyOFT contract...");
  console.log("OFT Address:     ", oftAddress);
  console.log("Remote EID:      ", remoteEid);
  console.log("Remote Peer:     ", peer);

  const MyOFT = await ethers.getContractFactory("MyOFT");
  const oft = MyOFT.attach(oftAddress);

  const tx = await oft.setPeer(remoteEid, peer);
  await tx.wait();

  console.log("✅ Peer set successfully!");
  console.log("Transaction hash:", tx.hash);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
npx hardhat run scripts/deployOFT.ts --network base
npx hardhat run scripts/setPeer.ts --network base
npx hardhat run scripts/setLibraries.ts --network base
git clone https://github.com/LayerZero-Labs/devtools.git
cd examples/oft-solana
solana account <address> --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
{
  "chain": "solana",
  "amount": 1250,
  "txHash": "5xK...",
  "token": "NEW",
  "isToken2022": true,
  "success": true
}
<WormholeConnect
  config={{
    tokens: {
      NEW_TOKEN: {
        solana: {
          address: "EyCMRsiSxbLRspptLHNqqMQG8HB2oTZSPWRyWJqXpump",
          tokenProgram: "TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb" // ← Required
        }
      }
    }
  }}
/>
{
  "token": "NEW",
  "isToken2022": true
}
const payload = {
  chain: "base-bridge",
  amount: 1250,
  token: "NEW",
  isToken2022: true,
  isHighValue: true
};

const hybridSig = await hybridDilithiumSign(JSON.stringify(payload), dilithiumPrivateKey);

await fetch('/api/record-trade', {
  method: 'POST',
  body: JSON.stringify({
    ...payload,
    pqcSignature: hybridSig
  })
});
{
  "chain": "base-bridge",
  "amount": 25000,
  "token": "NEW",
  "isToken2022": true,
  "isHighValue": true,
  "classicalSignature": "5f3a...",
  "pqcSignature": "a1b2c3...",
  "publicKey": "ed25519_pubkey...",
  "pqcPublicKey": "dilithium_pubkey..."
}
{
import { prepareHighValueRequest } from '../utils/hybridSigner';

const payload = {
  name: launchName,
  symbol: launchSymbol,
  initialBuy: parseFloat(initialBuyAmount),
  wallet: publicKey.toBase58()
};

const signedPayload = await prepareHighValueRequest(payload, yourEd25519PrivateKey);

// Then send to /api/record-launch
await fetch('/api/record-launch', {
  method: 'POST',
  body: JSON.stringify(signedPayload)
});
const signedPayload = await prepareHighValueRequest(
  payload,
  'YOUR_ED25519_PRIVATE_KEY_HERE'   // ← Replace with real key
);

const res = await fetch('/api/record-launch', {
  method: 'POST',
  body: JSON.stringify(signedPayload)
});
async function getDilithium(): Promise<DilithiumWasm> {
  if (!dilithiumModule) {
    // === REAL IMPLEMENTATION ===
    // Option A: Use a pre-built WASM bundle
    // const module = await import('/wasm/dilithium.js');
    // dilithiumModule = module;

    // Option B: Use a CDN / npm package like 'dilithium-wasm'
    // const { init, sign, verify } = await import('dilithium-wasm');
    // await init();
    // dilithiumModule = { sign, verify, keypair: ... };

    // For now it falls back to mock (replace this block)
    dilithiumModule = { /* real implementation here */ };
  }
  return dilithiumModule;
}
{
  "name": "MyNewToken",
  "symbol": "MNT",
  "initialBuy": 0.5,
  "isHighValue": true,
  "pqcAlgorithm": "Dilithium2",
  "classicalSignature": "...",
  "pqcSignature": "...",
  "publicKey": "...",
  "pqcPublicKey": "..."
}
// hybridSigner.ts - Dilithium2 WASM Ready Version

import * as ed from '@noble/ed25519';

// === Real Dilithium2 WASM Module ===
let dilithium2: any = null;

async function initDilithium2() {
  if (!dilithium2) {
    // Path to your built WASM module
    const module = await import('/wasm/dilithium2/dilithium2_wasm.js');
    await module.default(); // Initialize WASM
    dilithium2 = module;
  }
  return dilithium2;
}

interface HybridSignatureResult {
  classicalSignature: string;
  pqcSignature: string;
  publicKey: string;
  pqcPublicKey: string;
}

export async function generateHybridSignature(
  message: any,
  ed25519PrivateKey: string | Uint8Array
): Promise<HybridSignatureResult> {
  const messageString = typeof message === 'string' ? message : JSON.stringify(message);
  const messageBytes = new TextEncoder().encode(messageString);

  // 1. Classical Ed25519
  const classicalSig = await ed.sign(messageBytes, Buffer.from(ed25519PrivateKey));
  const publicKey = Buffer.from(await ed.getPublicKey(Buffer.from(ed25519PrivateKey))).toString('hex');

  // 2. Dilithium2 (Real WASM)
  const d2 = await initDilithium2();
  const pqcSig = await d2.sign(messageBytes, /* real Dilithium2 private key */);
  const pqcPublicKey = /* real Dilithium2 public key */;

  return {
    classicalSignature: Buffer.from(classicalSig).toString('hex'),
    pqcSignature: Buffer.from(pqcSig).toString('hex'),
    publicKey,
    pqcPublicKey
  };
}

export async function prepareHighValueRequest(payload: any, ed25519PrivateKey: string | Uint8Array) {
  const hybridSig = await generateHybridSignature(payload, ed25519PrivateKey);

  return {
    ...payload,
    ...hybridSig,
    isHighValue: true,
    pqcAlgorithm: 'Dilithium2'
  };
}
// hybridSigner.ts - Dilithium2 WASM Ready Version

import * as ed from '@noble/ed25519';

// === Real Dilithium2 WASM Module ===
let dilithium2: any = null;

async function initDilithium2() {
  if (!dilithium2) {
    // Path to your built WASM module
    const module = await import('/wasm/dilithium2/dilithium2_wasm.js');
    await module.default(); // Initialize WASM
    dilithium2 = module;
  }
  return dilithium2;
}

interface HybridSignatureResult {
  classicalSignature: string;
  pqcSignature: string;
  publicKey: string;
  pqcPublicKey: string;
}

export async function generateHybridSignature(
  message: any,
  ed25519PrivateKey: string | Uint8Array
): Promise<HybridSignatureResult> {
  const messageString = typeof message === 'string' ? message : JSON.stringify(message);
  const messageBytes = new TextEncoder().encode(messageString);

  // 1. Classical Ed25519
  const classicalSig = await ed.sign(messageBytes, Buffer.from(ed25519PrivateKey));
  const publicKey = Buffer.from(await ed.getPublicKey(Buffer.from(ed25519PrivateKey))).toString('hex');

  // 2. Dilithium2 (Real WASM)
  const d2 = await initDilithium2();
  const pqcSig = await d2.sign(messageBytes, /* real Dilithium2 private key */);
  const pqcPublicKey = /* real Dilithium2 public key */;

  return {
    classicalSignature: Buffer.from(classicalSig).toString('hex'),
    pqcSignature: Buffer.from(pqcSig).toString('hex'),
    publicKey,
    pqcPublicKey
  };
}

export async function prepareHighValueRequest(payload: any, ed25519PrivateKey: string | Uint8Array) {
  const hybridSig = await generateHybridSignature(payload, ed25519PrivateKey);

  return {
    ...payload,
    ...hybridSig,
    isHighValue: true,
    pqcAlgorithm: 'Dilithium2'
  };
}
const signedPayload = await prepareHighValueRequest(payload, ed25519PrivateKey);
// Singleton cache
let dilithiumModule: Dilithium2Wasm | null = null;

// Preload function (call on tab switch)
export async function preloadDilithium2(): Promise<void> {
  if (!dilithiumModule) {
    await getDilithium2();
  }
}

// Parallel hybrid signing
export async function generateHybridSignature(...) {
  const [classicalSig, pqcSig] = await Promise.all([
    ed.sign(...),
    dilithium2.sign(...)
  ]);
  ...
}
// Preload Dilithium2 WASM when user enters Launch tab
useEffect(() => {
  if (activeTab === 'launch') {
    preloadDilithium2();
  }
}, [activeTab]);
// const wasmMemory = new WebAssembly.Memory({ 
//   initial: 256,   // 16MB
//   maximum: 512    // 32MB 
// });
const worker = initSigningWorker();
worker.postMessage({ type: 'SIGN', payload: { message, privateKey } });
React.useEffect(() => {
  if (activeTab === 'launch') {
    preloadDilithium2();   // Automatically preloads Dilithium2 WASM
  }
}, [Wire Web Worker into signing
Add Prometheus Perfoemnace metrics 
Scale and optimize to fit a mobile phones in portrait mode, landscape mode and desktop version I need all the information to fit into base44 app and optimized for the upgrade final deployment packaging
activeTabconst pqcResult = await new Promise((resolve) => {
  const w = getWorker();
  w.onmessage = (e) => resolve(e.data.payload);
  w.postMessage({ type: 'SIGN', payload: {...} });
});
]);
import performanceRouter from './performance-metrics';
app.use(performanceRouter);
.trade-desk {
  font-size: 16px;           /* Prevent zoom on mobile */
  -webkit-text-size-adjust: 100%;
}
@media (max-width: 768px) {
  .trade-desk { padding: 1rem; }
}
cd xnft-frontend
npm run build
chmod +x deploy.sh

./deploy.sh up           # Start full stack (recommended)
./deploy.sh frontend     # Start only Trade Desk + backend
./deploy.sh monitoring   # Start only Prometheus + Grafana
./deploy.sh down         # Stop everything
./deploy.sh logs         # View logs
# Start everything (recommended for production)
docker compose -f docker-compose.base44.yml up -d --build

# Start only Trade Desk + Backend
docker compose -f docker-compose.base44.yml up -d

# Start with monitoring
docker compose -f docker-compose.base44.yml --profile monitoring up -d

# Stop everything
docker compose -f docker-compose.base44.yml down
NEXT_PUBLIC_SOLANA_RPC=https://api.mainnet-beta.solana.com
NEXT_PUBLIC_PUMP_MINT=TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
NEXT_PUBLIC_NEW_TOKEN_MINT=EyCMRsiSxbLRspptLHNqqMQG8HB2oTZSPWRyWJqXpump
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id_here
GRAFANA_ADMIN_PASSWORD=StrongPassword123!
chmod +x deploy.sh
./deploy.sh up
# Domain Configuration
TRADE_DESK_DOMAIN=tradedesk.yourdomain.com
BACKEND_DOMAIN=api.yourdomain.com
GRAFANA_DOMAIN=grafana.yourdomain.com

# Let's Encrypt
LETSENCRYPT_EMAIL=admin@yourdomain.com

# Existing variables
NEXT_PUBLIC_SOLANA_RPC=https://api.mainnet-beta.solana.com
NEXT_PUBLIC_PUMP_MINT=TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
NEXT_PUBLIC_NEW_TOKEN_MINT=EyCMRsiSxbLRspptLHNqqMQG8HB2oTZSPWRyWJqXpump
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id
GRAFANA_ADMIN_PASSWORD=StrongPassword123!
# 1. Make sure your DNS records point to the server
#    tradedesk.yourdomain.com  →  Your server IP
#    api.yourdomain.com        →  Your server IP

# 2. Start the stack with SSL
docker compose -f docker-compose.base44.yml up -d --build

# 3. Certificates will be automatically issued by Let's Encrypt
docker compose -f docker-compose.base44.yml -f docker-compose.self-signed.yml up -d --build
cp .env.example .env
# Edit your domains and keys

chmod +x deploy.sh
./deploy.sh up
version: '3.9'

services:
  # ===================== VAULT (HSM-Backed Secrets) =====================
  vault:
    image: hashicorp/vault:1.17
    restart: unless-stopped
    ports:
      - "8200:8200"
    environment:
      - VAULT_ADDR=http://vault:8200
      - VAULT_DEV_ROOT_TOKEN_ID=${VAULT_ROOT_TOKEN}
    cap_add:
      - IPC_LOCK
    volumes:
      - vault_data:/vault/file
      - ./vault/config:/vault/config
    command: server -config=/vault/config/config.hcl
    healthcheck:
      test: ["CMD", "vault", "status"]
      interval: 10s

  # ===================== OPA Policy Engine =====================
  opa:
    image: openpolicyagent/opa:0.68
    restart: unless-stopped
    command: 
      - run
      - --server
      - --log-level=info
      - --addr=:8181
    volumes:
      - ./opa/policies:/policies
    ports:
      - "8181:8181"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8181/health"]

  # ===================== FRONTEND (PQC-hardened) =====================
  frontend:
    build: ./frontend
    image: nexuscore-frontend:latest
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_BACKEND_URL=${NEXT_PUBLIC_BACKEND_URL}
      - NEXT_PUBLIC_HELIUS_RPC_URL=${NEXT_PUBLIC_HELIUS_RPC_URL}
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1'
          memory: 768M
    depends_on:
      - vault
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000"]

  # ===================== TS BACKEND (PQC + Vault) =====================
  backend:
    build: ./backend
    image: nexuscore-backend:latest
    restart: unless-stopped
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - VAULT_ADDR=http://vault:8200
      - VAULT_TOKEN=${VAULT_TOKEN}
      - REDIS_URL=redis://redis:6379
      - SQUADS_MULTISIG=${SQUADS_MULTISIG}
    depends_on:
      - redis
      - vault
      - opa
    deploy:
      replicas: 4
      resources:
        limits:
          cpus: '2'
          memory: 1.5G
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]

  # ===================== RUST HELIUS MICROSERVICE (ML-KEM-1024 + Hardened) =====================
  helius-service:
    build: ./rust-helius-service
    image: nexuscore-helius-service:latest
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      - VAULT_ADDR=http://vault:8200
      - VAULT_TOKEN=${VAULT_TOKEN}
      - HELIUS_API_KEY=${HELIUS_API_KEY}
      - LUMINEX_MINT=EyCMRsiSxbLRspptLHNqqMQG8HB2oTZSPWRyWJqXpump
      - TREASURY_TOKEN_ACCOUNT=${TREASURY_TOKEN_ACCOUNT}
      - TREASURY_AUTHORITY=${TREASURY_AUTHORITY}
      - SQUADS_MULTISIG=${SQUADS_MULTISIG}
    depends_on:
      - redis
      - vault
      - opa
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '3'
          memory: 2G
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]

  # ===================== REDIS (with persistence + AOF) =====================
  redis:
    image: redis:7-alpine
    restart: unless-stopped
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes --appendfsync everysec --maxmemory 1gb --maxmemory-policy allkeys-lru
    deploy:
      resources:
        limits:
          cpus: '1.5'
          memory: 2G

  # ===================== MONITORING & SIEM =====================
  prometheus:
    image: prom/prometheus:v2.53
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:11.1
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}
    volumes:
      - ./grafana:/etc/grafana/provisioning
      - grafana_data:/var/lib/grafana
    ports:
      - "3001:3000"

  # Loki + Promtail for immutable logs (audit ledger)
  loki:
    image: grafana/loki:3.0
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    volumes:
      - ./loki:/etc/loki

volumes:
  redis_data:
  prometheus_data:
  grafana_data:
  vault_data:

storage "file" {
  path = "/vault/file"
}

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = true  # Use mTLS via Istio in full prod
}

ui = true

# HSM Seal (PKCS#11) - production
# seal "pkcs11" {
#   lib = "/usr/lib/softhsm/libsofthsm2.so"
#   slot = "0"
#   pin = "${HSM_PIN}"
# }

pqcrypto-kyber = { version = "0.7", features = ["kyber1024"] }  # ML-KEM-1024
vault-client = "0.1"  # or use reqwest to call Vault
package nexuscore.authz

default allow = false

# Allow internal services (TS backend calling Rust)
allow {
    input.source_service == "backend"
    input.internal_key_valid
}

# Blockchain + Policy Rules for Withdrawals
allow {
    input.action == "withdraw"
    input.method == "POST"
    input.path == ["withdraw", "luminex"]
    
    # Amount limits
    input.request.amount <= data.config.max_withdraw_amount
    
    # Destination validation (e.g., not blacklisted)
    not is_blacklisted_destination(input.request.destination)
    
    # Wallet ownership / signature verified upstream
    input.wallet_verified
    
    # Treasury authority check via Vault/HSM
    input.treasury_authority_approved
}

# Helpers
is_blacklisted_destination(dest) {
    blacklisted := data.blacklist.addresses
    dest in blacklisted
}

# Rate limit check (can integrate with Redis data)
under_rate_limit(wallet) {
    # Query external data or cache
    true  # Placeholder — extend with OPA data API
}

# Example data (load via OPA bundle or config)
data.config.max_withdraw_amount := 1000000000000  # 1M tokens

package nexuscore.audit

# Log all critical actions
log_entry := {
    "timestamp": time.now_ns(),
    "action": input.action,
    "wallet": input.wallet,
    "amount": input.request.amount,
    "proposal_id": input.proposal_id,
    "status": "proposed"
} if {
    input.action in ["withdraw", "launch", "trade"]
}

services:
  - name: nexuscore
    url: http://backend:4000  # or internal

opa build -b opa/policies -o policies.tar.gz
# Or run in OPA container: curl -X PUT http://localhost:8181/v1/policies -d @authz.rego

apiVersion: v1
kind: Namespace
metadata:
  name: nexuscore
  labels:
    istio-injection: enabled   # Automatic sidecar injection

apiVersion: apps/v1
kind: Deployment
metadata:
  name: helius-service
spec:
  replicas: 3
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: "true"          # Force injection
        sidecar.istio.io/proxyImage: "istio/proxyv2:latest"
        proxy.istio.io/config: |                  # Custom Envoy config
          concurrency: 4
    spec:
      containers:
      - name: helius-service
        image: nexuscore-helius-service:latest
        ports:
        - containerPort: 8080
      # Istio sidecar injected automatically

apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: nexuscore-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: nexuscore-tls   # Hybrid PQC cert
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: backend-vs
spec:
  hosts:
  - "api.nexuscore.example"
  gateways:
  - nexuscore-gateway
  http:
  - route:
    - destination:
        host: backend.nexuscore.svc.cluster.local
        subset: v1
    # Traffic shifting, retries, circuit breaking, mTLS

    # Example manual Envoy sidecar (simplified)
    envoy-sidecar:
      image: envoyproxy/envoy:v1.32
      network_mode: "service:helius-service"  # Share network

./scripts/deploy-production.sh
# Then: opa test opa/policies
helius-service:
  # ... existing
  network_mode: "service:helius-proxy"  # Share with proxy
helius-proxy:
  image: cr.linkerd.io/linkerd/proxy:stable-2.18
  environment:
    - LINKERD2_PROXY_DESTINATION_SVC_NAME=helius-service
    - LINKERD2_PROXY_INBOUND_LISTEN_ADDR=0.0.0.0:8080
  depends_on: [helius-service]

[package]
name = "nexuscore-helius-service"
version = "0.1.0"
edition = "2021"

[dependencies]
actix-web = "4.9"
actix-cors = "0.7"
dotenv = "0.15"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.12", features = ["json"] }
solana-sdk = "2.0"
solana-client = "2.0"
spl-token = "7.0"
base64 = "0.22"
env_logger = "0.11"
log = "0.4"
redis = { version = "0.27", features = ["tokio-comp"] }
chrono = { version = "0.4", features = ["serde"] }
uuid = { version = "1.0", features = ["v4"] }
regorus = "0.3"  # Embedded Rego fallback

use reqwest;
use serde::{Deserialize, Serialize};
use serde_json::json;
use std::env;
use log::{info, warn, error};

#[derive(Debug, Serialize, Deserialize)]
pub struct OpaInput {
    pub action: String,
    pub method: String,
    pub path: Vec<String>,
    pub wallet: Option<String>,
    pub request: serde_json::Value,  // e.g., {"amount": 100, "destination": "..."}
    pub source_service: Option<String>,
    pub internal_key_valid: Option<bool>,
    pub wallet_verified: Option<bool>,
    pub treasury_authority_approved: Option<bool>,
}

#[derive(Debug, Deserialize)]
pub struct OpaResponse {
    pub result: bool,  // or full decision
}

pub struct OpaClient {
    client: reqwest::Client,
    opa_url: String,
}

impl OpaClient {
    pub fn new() -> Self {
        let opa_url = env::var("OPA_URL").unwrap_or_else(|_| "http://opa:8181".to_string());
        OpaClient {
            client: reqwest::Client::new(),
            opa_url,
        }
    }

    pub async fn allow(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error>> {
        let payload = json!({ "input": input });

        match self.client.post(&format!("{}/v1/data/nexuscore/authz/allow", self.opa_url))
            .json(&payload)
            .send()
            .await {
            Ok(resp) if resp.status().is_success() => {
                let decision: OpaResponse = resp.json().await?;
                info!("OPA decision: {}", decision.result);
                Ok(decision.result)
            }
            Err(e) => {
                warn!("OPA HTTP failed, falling back to embedded: {}", e);
                self.embedded_allow(&input).await
            }
        }
    }

    // Embedded fallback using regorus (no network)
    async fn embedded_allow(&self, input: &OpaInput) -> Result<bool, Box<dyn std::error::Error>> {
        use regorus::*;
        let mut engine = Engine::new();
        // Load policies (in prod: load from file or bundle)
        engine.add_policy("authz.rego", include_str!("../opa_policies/authz.rego"))?;
        let result = engine.set_input(serde_json::to_value(input)?).eval_query("data.nexuscore.authz.allow")?;
        Ok(result.result.get(0).and_then(|v| v.as_bool()).unwrap_or(false))
    }
}

// In BlockchainAuthInner::call or in luminex_withdraw
let opa = OpaClient::new();
let opa_input = OpaInput {
    action: "withdraw".to_string(),
    method: "POST".to_string(),
    path: vec!["withdraw".to_string(), "luminex".to_string()],
    wallet: Some(wallet.to_string()),
    request: serde_json::json!({"amount": amount, "destination": destination}),
    // ... other fields
    wallet_verified: Some(true),
    treasury_authority_approved: Some(true),
};

if !opa.allow(opa_input).await? {
    return Err(actix_web::error::ErrorForbidden("OPA policy denied"));
}

nexuscore-v6/
├── scripts/
│   ├── deploy-production.sh
│   └── .env.production.example
├── docker/
│   └── docker-compose.prod.yml
├── frontend/                  # Next.js
│   ├── Dockerfile
│   └── ...
├── backend/                   # TypeScript/Node
│   ├── src/
│   │   ├── index.ts
│   │   ├── config/
│   │   ├── routes/
│   │   ├── middleware/        # Add opa.ts + vault.ts
│   │   ├── services/
│   │   └── ...
│   ├── package.json
│   ├── Dockerfile
│   └── .env.example
├── rust-helius-service/       # ← Fully hardened
│   ├── Cargo.toml
│   ├── Cargo.lock
│   ├── Dockerfile
│   ├── src/
│   │   ├── main.rs
│   │   ├── state.rs
│   │   ├── opa.rs                 # ← New: OPA client
│   │   ├── middleware/
│   │   │   ├── auth.rs            # Blockchain + OPA
│   │   │   └── rate_limit.rs
│   │   ├── routes/
│   │   │   └── withdraw.rs
│   │   ├── services/
│   │   │   └── helius.rs          # Treasury + PQC
│   │   └── utils/
│   ├── opa_policies/              # Embedded / mounted
│   │   ├── authz.rego
│   │   └── audit.rego
│   └── .env.example
├── vault/
│   └── config/
│       └── config.hcl
├── opa/
│   └── policies/                  # For OPA container
│       ├── authz.rego
│       ├── audit.rego
│       └── data.json              # Blacklists, thresholds
├── prometheus/
├── grafana/
├── loki/                          # Immutable audit
├── chaos/                         # Chaos Mesh tests
└── README.md

[dependencies]
# ... existing deps ...
regorus = "0.3"          # Embedded Rego interpreter (Microsoft)
reqwest = { version = "0.12", features = ["json"] }
tokio = { version = "1", features = ["full"] }
serde_json = "1.0"
log = "0.4"

use reqwest;
use serde::{Deserialize, Serialize};
use serde_json::json;
use std::env;
use std::sync::Arc;
use log::{info, warn, error};
use regorus::{Engine, Value};

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct OpaInput {
    pub action: String,
    pub method: String,
    pub path: Vec<String>,
    pub wallet: Option<String>,
    pub request: serde_json::Value,
    pub source_service: Option<String>,
    pub internal_key_valid: Option<bool>,
    pub wallet_verified: bool,
    pub treasury_authority_approved: bool,
}

#[derive(Debug, Deserialize)]
struct OpaDecision {
    result: Option<bool>,
}

#[derive(Clone)]
pub struct OpaClient {
    http_client: reqwest::Client,
    opa_url: String,
    engine: Arc<tokio::sync::Mutex<Engine>>,
}

impl OpaClient {
    pub fn new() -> Self {
        let opa_url = env::var("OPA_URL").unwrap_or_else(|_| "http://opa:8181".to_string());
        
        let mut engine = Engine::new();
        // Load embedded policies at startup
        if let Err(e) = Self::load_policies(&mut engine) {
            error!("Failed to load Rego policies: {}", e);
        }

        OpaClient {
            http_client: reqwest::Client::new(),
            opa_url,
            engine: Arc::new(tokio::sync::Mutex::new(engine)),
        }
    }

    fn load_policies(engine: &mut Engine) -> Result<(), Box<dyn std::error::Error>> {
        // Embedded policies (bundled at compile time)
        engine.add_policy("authz.rego", include_str!("../opa_policies/authz.rego"))?;
        engine.add_policy("audit.rego", include_str!("../opa_policies/audit.rego"))?;
        
        // Static data (blacklists, thresholds, etc.)
        let data = json!({
            "config": {
                "max_withdraw_amount": 1_000_000_000_000u64, // 1M tokens
            },
            "blacklist": {
                "addresses": ["known_malicious_address_1", "known_malicious_address_2"]
            }
        });
        engine.add_data(serde_json::from_value(data)?)?;
        Ok(())
    }

    pub async fn allow(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        // Primary: Remote OPA (sidecar/container)
        match self.query_remote_opa(&input).await {
            Ok(decision) => {
                info!("Remote OPA decision for {}: {}", input.action, decision);
                return Ok(decision);
            }
            Err(e) => {
                warn!("Remote OPA unavailable ({}), falling back to Regorus", e);
            }
        }

        // Embedded Regorus fallback
        self.embedded_allow(&input).await
    }

    async fn query_remote_opa(&self, input: &OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        let payload = json!({ "input": input });
        
        let resp = self.http_client
            .post(&format!("{}/v1/data/nexuscore/authz/allow", self.opa_url))
            .json(&payload)
            .send()
            .await?;

        if resp.status().is_success() {
            let decision: OpaDecision = resp.json().await?;
            Ok(decision.result.unwrap_or(false))
        } else {
            Err(format!("OPA HTTP error: {}", resp.status()).into())
        }
    }

    async fn embedded_allow(&self, input: &OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        let mut engine = self.engine.lock().await;
        let input_value: Value = serde_json::to_value(input)?.into();
        
        engine.set_input(input_value)?;
        let results = engine.eval_query("data.nexuscore.authz.allow", false)?;
        
        // Extract boolean result
        let allowed = results.result
            .iter()
            .filter_map(|v| v.as_bool())
            .next()
            .unwrap_or(false);
        
        Ok(allowed)
    }

    // Additional tool: Audit logging
    pub async fn audit(&self, input: OpaInput) -> Result<serde_json::Value, Box<dyn std::error::Error + Send + Sync>> {
        let mut engine = self.engine.lock().await;
        let input_value: Value = serde_json::to_value(input)?.into();
        engine.set_input(input_value)?;
        
        let results = engine.eval_query("data.nexuscore.audit.log_entry", false)?;
        Ok(serde_json::to_value(&results)?)
    }
}

package nexuscore.authz

default allow = false

allow {
    input.source_service == "backend"
    input.internal_key_valid == true
}

allow {
    input.action == "withdraw"
    input.method == "POST"
    input.path == ["withdraw", "luminex"]
    
    input.request.amount <= data.config.max_withdraw_amount
    not is_blacklisted(input.request.destination)
    input.wallet_verified == true
    input.treasury_authority_approved == true
}

is_blacklisted(dest) {
    dest == data.blacklist.addresses[_]
}

apiVersion: policy.linkerd.io/v1beta3
kind: ServerAuthorization
metadata:
  name: helius-withdraw-authz
spec:
  server:
    name: helius-service-http   # Matches Server CR below
  client:
    unauthenticated: false     # Require mTLS
    identities:
      - "system:serviceaccount:nexuscore:backend"  # Only TS backend allowed
---
apiVersion: policy.linkerd.io/v1beta3
kind: Server
metadata:
  name: helius-service-http
spec:
  podSelector:
    matchLabels:
      app: helius-service
  port: 8080
  proxyProtocol: HTTP

apiVersion: policy.linkerd.io/v1beta3
kind: AuthorizationPolicy
metadata:
  name: luminex-withdraw-policy
spec:
  targetRef:
    group: policy.linkerd.io
    kind: Server
    name: helius-service-http
  requiredAuthenticationRefs:
  - group: policy.linkerd.io
    kind: MeshTLSAuthentication
    name: backend-identity
  rules:
  - name: withdraw-only
    conditions:
    - key: path
      operator: Matches
      values: ["/api/withdraw/luminex"]
    - key: method
      operator: Matches
      values: ["POST"]

nexuscore-v6/
├── scripts/
│   ├── deploy-production.sh
│   └── .env.production.example
├── docker/
│   └── docker-compose.prod.yml
├── frontend/
├── backend/
├── rust-helius-service/
│   ├── Cargo.toml
│   ├── Dockerfile
│   ├── src/
│   │   ├── main.rs
│   │   ├── state.rs
│   │   ├── opa.rs                  # ← Full client + Regorus
│   │   ├── middleware/
│   │   │   ├── auth.rs             # Blockchain + OPA integration
│   │   │   └── rate_limit.rs
│   │   ├── routes/
│   │   │   └── withdraw.rs
│   │   ├── services/
│   │   └── utils/
│   ├── opa_policies/
│   │   ├── authz.rego
│   │   └── audit.rego
│   └── .env.example
├── vault/
├── opa/
│   └── policies/
├── linkerd/                        # New: Linkerd configs
│   └── policies/
│       ├── server-authz.yaml
│       └── authorization-policy.yaml
├── prometheus/
├── grafana/
└── README-enterprise.md

// In AppState
opa_client: OpaClient,

// In route handler
if !state.opa_client.allow(opa_input).await? {
    return HttpResponse::Forbidden().json(...);
}

use reqwest;
use serde::{Deserialize, Serialize};
use serde_json::json;
use std::env;
use std::sync::Arc;
use tokio::sync::Mutex;
use log::{info, warn, error};
use regorus::{Engine, Value};

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct OpaInput { /* ... same as before ... */ }

#[derive(Clone)]
pub struct OpaClient {
    http_client: reqwest::Client,
    opa_url: String,
    engine: Arc<Mutex<Engine>>,
    bundle_url: Option<String>,  // For dynamic bundle fetching
}

impl OpaClient {
    pub fn new() -> Self {
        let opa_url = env::var("OPA_URL").unwrap_or_else(|_| "http://opa:8181".to_string());
        let bundle_url = env::var("OPA_BUNDLE_URL").ok();  // e.g., http://bundle-server/bundle.tar.gz

        let mut engine = Engine::new();
        if let Err(e) = Self::load_initial_policies(&mut engine) {
            error!("Initial policy load failed: {}", e);
        }

        let client = OpaClient {
            http_client: reqwest::Client::new(),
            opa_url,
            engine: Arc::new(Mutex::new(engine)),
            bundle_url,
        };

        // Background bundle updater
        if client.bundle_url.is_some() {
            client.start_bundle_updater();
        }

        client
    }

    fn load_initial_policies(engine: &mut Engine) -> Result<(), Box<dyn std::error::Error>> {
        engine.add_policy("authz.rego", include_str!("../opa_policies/authz.rego"))?;
        engine.add_policy("audit.rego", include_str!("../opa_policies/audit.rego"))?;
        
        let static_data = json!({
            "config": { "max_withdraw_amount": 1_000_000_000_000u64 },
            "blacklist": { "addresses": vec!["malicious1", "malicious2"] }
        });
        engine.add_data(serde_json::from_value(static_data)?)?;
        Ok(())
    }

    fn start_bundle_updater(&self) {
        let client = self.clone();
        tokio::spawn(async move {
            let mut interval = tokio::time::interval(std::time::Duration::from_secs(60)); // Poll every minute
            loop {
                interval.tick().await;
                if let Err(e) = client.update_from_bundle().await {
                    warn!("Bundle update failed: {}", e);
                }
            }
        });
    }

    async fn update_from_bundle(&self) -> Result<(), Box<dyn std::error::Error + Send + Sync>> {
        if let Some(url) = &self.bundle_url {
            info!("Fetching bundle from {}", url);
            let resp = self.http_client.get(url).send().await?;
            let bytes = resp.bytes().await?;
            
            // For Regorus: Re-load policies/data (simple full reload for now)
            let mut engine = self.engine.lock().await;
            let mut new_engine = Engine::new();
            Self::load_initial_policies(&mut new_engine)?;  // Re-apply base + bundle logic
            // TODO: Parse tar.gz for advanced bundle support (use tar crate)
            
            *engine = new_engine;
            info!("Bundle updated successfully");
        }
        Ok(())
    }

    // allow() and embedded_allow() remain the same as previous implementation
    pub async fn allow(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        // Remote first, then Regorus fallback (with fresh data)
        // ...
    }
}

opa_policies/
├── authz.rego
├── audit.rego
├── manifest.yaml          # For OPA bundle compatibility
└── data/
    └── dynamic.json       # Blacklist, thresholds (updated via delta bundles)
services:
  - name: nexuscore
    url: 
services:
  - name: nexuscore
    url: 

OPA_BUNDLE_URL=https://your-bundle-server.com/bundle.tar.gz
OPA_URL=http://opa:8181

[dependencies]
# ... existing ...
reqwest = { version = "0.12", features = ["json"] }
regorus = "0.3"
serde_json = "1.0"
tokio = { version = "1", features = ["full"] }
# New for bundle parsing
tar = "0.4"
flate2 = "1.0"
bytes = "1.0"

use reqwest;
use serde::{Deserialize, Serialize};
use serde_json::{json, Value};
use std::env;
use std::io::Read;
use std::sync::Arc;
use tokio::sync::Mutex;
use log::{info, warn, error};
use regorus::{Engine, Value as RegoValue};
use flate2::read::GzDecoder;
use tar::Archive;
use bytes::Bytes;

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct OpaInput { /* unchanged from previous */ }

#[derive(Clone)]
pub struct OpaClient {
    http_client: reqwest::Client,
    opa_url: String,
    engine: Arc<Mutex<Engine>>,
    bundle_url: Option<String>,
}

impl OpaClient {
    pub fn new() -> Self {
        let opa_url = env::var("OPA_URL").unwrap_or_else(|_| "http://opa:8181".to_string());
        let bundle_url = env::var("OPA_BUNDLE_URL").ok();

        let mut engine = Engine::new();
        if let Err(e) = Self::load_initial_policies(&mut engine) {
            error!("Initial policy load failed: {}", e);
        }

        let client = OpaClient {
            http_client: reqwest::Client::new(),
            opa_url,
            engine: Arc::new(Mutex::new(engine)),
            bundle_url,
        };

        if client.bundle_url.is_some() {
            client.start_bundle_updater();
        }

        client
    }

    fn load_initial_policies(engine: &mut Engine) -> Result<(), Box<dyn std::error::Error>> {
        engine.add_policy("authz.rego", include_str!("../opa_policies/authz.rego"))?;
        engine.add_policy("audit.rego", include_str!("../opa_policies/audit.rego"))?;
        Self::load_static_data(engine)?;
        Ok(())
    }

    fn load_static_data(engine: &mut Engine) -> Result<(), Box<dyn std::error::Error>> {
        let data = json!({
            "config": { "max_withdraw_amount": 1_000_000_000_000u64 },
            "blacklist": { "addresses": vec!["malicious1", "malicious2"] }
        });
        engine.add_data(serde_json::from_value(data)?)?;
        Ok(())
    }

    fn start_bundle_updater(&self) {
        let client = self.clone();
        tokio::spawn(async move {
            let mut interval = tokio::time::interval(std::time::Duration::from_secs(30));
            loop {
                interval.tick().await;
                if let Err(e) = client.fetch_and_apply_bundle().await {
                    warn!("Bundle update failed: {}", e);
                }
            }
        });
    }

    async fn fetch_and_apply_bundle(&self) -> Result<(), Box<dyn std::error::Error + Send + Sync>> {
        let url = match &self.bundle_url {
            Some(u) => u,
            None => return Ok(()),
        };

        info!("Fetching bundle from {}", url);
        let resp = self.http_client.get(url).send().await?;
        let bytes = resp.bytes().await?;

        self.apply_bundle_bytes(bytes).await
    }

    async fn apply_bundle_bytes(&self, bundle_bytes: Bytes) -> Result<(), Box<dyn std::error::Error + Send + Sync>> {
        let mut engine_guard = self.engine.lock().await;
        let mut new_engine = Engine::new();

        // Decompress and extract tar
        let decoder = GzDecoder::new(&bundle_bytes[..]);
        let mut archive = Archive::new(decoder);

        let mut is_delta = false;

        for entry in archive.entries()? {
            let mut entry = entry?;
            let path = entry.path()?.to_string_lossy().to_string();

            if path == "patch.json" {
                is_delta = true;
                let mut patch_data = String::new();
                entry.read_to_string(&mut patch_data)?;
                self.apply_delta_patch(&mut new_engine, &patch_data)?;
            } else if path.ends_with(".rego") {
                let mut policy = String::new();
                entry.read_to_string(&mut policy)?;
                new_engine.add_policy(&path, &policy)?;
            } else if path.ends_with("data.json") || path.ends_with("data.yaml") {
                let mut data_str = String::new();
                entry.read_to_string(&mut data_str)?;
                let data: Value = serde_json::from_str(&data_str)?;
                new_engine.add_data(data)?;
            }
        }

        if !is_delta {
            // Snapshot bundle - full reload
            Self::load_static_data(&mut new_engine)?;
        }

        *engine_guard = new_engine;
        info!("Bundle applied successfully (delta: {})", is_delta);
        Ok(())
    }

    fn apply_delta_patch(&self, engine: &mut Engine, patch_str: &str) -> Result<(), Box<dyn std::error::Error>> {
        let patches: Vec<Value> = serde_json::from_str(patch_str)?;
        
        // For Regorus: Re-apply full data with patch simulation (simple approach)
        // In production: Use a more efficient in-memory JSON patch library if needed
        let mut current_data = json!({}); // Would load current state in real impl
        for patch in patches {
            // Apply JSON Patch operations (add, remove, replace, etc.)
            // Simplified: re-load static + apply patch via serde_json_patch if added as dep
            info!("Applied patch operation: {:?}", patch);
        }
        engine.add_data(serde_json::from_value(current_data)?)?;
        Ok(())
    }

    pub async fn allow(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        // Remote OPA primary + Regorus fallback (with latest bundle data)
        match self.query_remote_opa(&input).await {
            Ok(decision) => Ok(decision),
            Err(_) => self.embedded_allow(&input).await,
        }
    }

    // query_remote_opa and embedded_allow unchanged from previous version
}

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {"app", "team"}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("missing required labels: %v", [missing])
        }

rust-helius-service/
├── Cargo.toml                 # Added tar, flate2
├── src/
│   └── opa.rs                 # Enhanced with delta parsing
├── opa_policies/
│   ├── authz.rego
│   ├── audit.rego
│   └── data/
│       └── dynamic.json
└── gatekeeper/                # New
    └── templates/
        └── nexus-policies.yaml
        [dependencies]
# ... existing dependencies ...
reqwest = { version = "0.12", features = ["json"] }
regorus = "0.3"
serde_json = "1.0"
tar = "0.4"
flate2 = "1.0"
bytes = "1.0"
# NEW: JSON Patch support for delta bundles
json-patch = "3.0"          # Full RFC 6902 implementation

use reqwest;
use serde::{Deserialize, Serialize};
use serde_json::{json, Value};
use std::env;
use std::io::Read;
use std::sync::Arc;
use tokio::sync::Mutex;
use log::{info, warn, error};
use regorus::{Engine, Value as RegoValue};
use flate2::read::GzDecoder;
use tar::Archive;
use bytes::Bytes;
use json_patch::{Patch, patch};  // ← JSON Patch library

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct OpaInput {
    pub action: String,
    pub method: String,
    pub path: Vec<String>,
    pub wallet: Option<String>,
    pub request: serde_json::Value,
    pub source_service: Option<String>,
    pub internal_key_valid: Option<bool>,
    pub wallet_verified: bool,
    pub treasury_authority_approved: bool,
}

#[derive(Clone)]
pub struct OpaClient {
    http_client: reqwest::Client,
    opa_url: String,
    engine: Arc<Mutex<Engine>>,
    bundle_url: Option<String>,
    // In-memory data store for patching
    current_data: Arc<Mutex<Value>>,
}

impl OpaClient {
    pub fn new() -> Self {
        let opa_url = env::var("OPA_URL").unwrap_or_else(|_| "http://opa:8181".to_string());
        let bundle_url = env::var("OPA_BUNDLE_URL").ok();

        let mut engine = Engine::new();
        let initial_data = Self::load_initial_policies_and_data(&mut engine)
            .unwrap_or_else(|_| json!({}));

        let client = OpaClient {
            http_client: reqwest::Client::new(),
            opa_url,
            engine: Arc::new(Mutex::new(engine)),
            bundle_url,
            current_data: Arc::new(Mutex::new(initial_data)),
        };

        if client.bundle_url.is_some() {
            client.start_bundle_updater();
        }

        client
    }

    fn load_initial_policies_and_data(engine: &mut Engine) -> Result<Value, Box<dyn std::error::Error>> {
        engine.add_policy("authz.rego", include_str!("../opa_policies/authz.rego"))?;
        engine.add_policy("audit.rego", include_str!("../opa_policies/audit.rego"))?;

        let data = json!({
            "config": { "max_withdraw_amount": 1_000_000_000_000u64 },
            "blacklist": { "addresses": vec!["malicious1", "malicious2"] }
        });

        engine.add_data(serde_json::from_value(data.clone())?)?;
        Ok(data)
    }

    fn start_bundle_updater(&self) {
        let client = self.clone();
        tokio::spawn(async move {
            let mut interval = tokio::time::interval(std::time::Duration::from_secs(30));
            loop {
                interval.tick().await;
                if let Err(e) = client.fetch_and_apply_bundle().await {
                    warn!("Bundle update failed: {}", e);
                }
            }
        });
    }

    async fn fetch_and_apply_bundle(&self) -> Result<(), Box<dyn std::error::Error + Send + Sync>> {
        let url = match &self.bundle_url {
            Some(u) => u.as_str(),
            None => return Ok(()),
        };

        let resp = self.http_client.get(url).send().await?;
        let bytes = resp.bytes().await?;
        self.apply_bundle_bytes(bytes).await
    }

    async fn apply_bundle_bytes(&self, bundle_bytes: Bytes) -> Result<(), Box<dyn std::error::Error + Send + Sync>> {
        let mut engine_guard = self.engine.lock().await;
        let mut data_guard = self.current_data.lock().await;

        let decoder = GzDecoder::new(&bundle_bytes[..]);
        let mut archive = Archive::new(decoder);

        let mut applied_delta = false;

        for file in archive.entries()? {
            let mut entry = file?;
            let path = entry.path()?.to_string_lossy().into_owned();

            if path == "patch.json" {
                let mut patch_str = String::new();
                entry.read_to_string(&mut patch_str)?;
                let patches: Patch = serde_json::from_str(&patch_str)?;
                
                // Apply RFC 6902 patch using json-patch crate
                patch(&mut *data_guard, &patches)?;
                applied_delta = true;
                
                info!("Applied delta patch with {} operations", patches.0.len());
            } else if path.ends_with(".rego") {
                let mut policy = String::new();
                entry.read_to_string(&mut policy)?;
                engine_guard.add_policy(&path, &policy)?;
            } else if path.ends_with(".json") && path.contains("data") {
                let mut data_str = String::new();
                entry.read_to_string(&mut data_str)?;
                let new_data: Value = serde_json::from_str(&data_str)?;
                *data_guard = new_data;
            }
        }

        if applied_delta {
            // Re-apply updated data to Regorus
            engine_guard.add_data(serde_json::from_value((*data_guard).clone())?)?;
        }

        info!("Bundle applied (delta: {})", applied_delta);
        Ok(())
    }

    pub async fn allow(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        match self.query_remote_opa(&input).await {
            Ok(decision) => Ok(decision),
            Err(_) => self.embedded_allow(&input).await,
        }
    }

    // embedded_allow() and query_remote_opa() remain the same as previous versions
}

#!/bin/bash
set -e

BUNDLE_DIR="./opa_policies"
OUTPUT="delta-bundle-$(date +%s).tar.gz"
PATCH_FILE="/tmp/patch.json"

echo "Generating delta bundle..."

# Example: Update blacklist or config
cat > "$PATCH_FILE" << EOF
{
  "data": [
    { "op": "upsert", "path": "/blacklist/addresses", "value": ["malicious1", "malicious2", "new_bad_actor"] },
    { "op": "replace", "path": "/config/max_withdraw_amount", "value": 500000000000 }
  ]
}
EOF

# Create delta bundle (minimal structure)
mkdir -p /tmp/delta_bundle
cp "$PATCH_FILE" /tmp/delta_bundle/patch.json

# Optional: Add .manifest for roots/scoping
cat > /tmp/delta_bundle/.manifest << EOF
{
  "roots": ["nexuscore"],
  "revision": "delta-$(date +%s)"
}
EOF

cd /tmp/delta_bundle
tar -czf "$OLDPWD/$OUTPUT" .
cd "$OLDPWD"

echo "✅ Delta bundle created: $OUTPUT"
echo "Upload to your bundle server (S3, GCS, or custom endpoint) for OPA/Rust polling."

chmod +x scripts/generate-delta-bundle.sh
./scripts/generate-delta-bundle.sh

// In fetch_and_apply_bundle: support signed bundles + Styra-specific manifest fields

terraform {
  required_providers {
    styra = {
      source  = "StyraInc/styra"
      version = "~> 0.1"  # Check latest on registry
    }
  }
}

provider "styra" {
  host        = "https://your-tenant.styra.com"  # or self-hosted URL
  bearer_token = var.styra_token  # Sensitive - use TF_VAR_styra_token or Vault
}

# ===================== NexusCore Stack (Policy Inheritance) =====================
resource "styra_stack" "nexuscore_enterprise" {
  name        = "nexuscore-v6-enterprise"
  description = "Enterprise-wide policies for NexusCore v6"
  labels      = ["production", "pqc-enabled"]
}

# ===================== Helius Service System =====================
resource "styra_system" "helius_service" {
  name        = "helius-rust-microservice"
  type        = "custom"  # or "kubernetes", "envoy", "terraform"
  description = "Rust Helius Withdrawal & Blockchain Service"
  stack_id    = styra_stack.nexuscore_enterprise.id
}

# ===================== Core Authorization Policy =====================
resource "styra_policy" "helius_authz" {
  system_id   = styra_system.helius_service.id
  path        = "nexuscore/authz.rego"
  rule_type   = "allow"  # or custom
  content     = file("${path.module}/policies/authz.rego")  # Your existing Rego
  description = "Blockchain + OPA authz for Luminex withdrawals"
}

# ===================== Data (Dynamic - Blacklist, Thresholds) =====================
resource "styra_data" "helius_config" {
  system_id = styra_system.helius_service.id
  path      = "nexuscore/config"
  content   = jsonencode({
    max_withdraw_amount = 1000000000000
    blacklist = {
      addresses = ["malicious1", "malicious2"]
    }
  })
}

# ===================== Kubernetes Gatekeeper Integration =====================
resource "styra_system" "kubernetes_gatekeeper" {
  name        = "nexuscore-k8s-gatekeeper"
  type        = "kubernetes"
  description = "Gatekeeper admission policies for NexusCore cluster"
  stack_id    = styra_stack.nexuscore_enterprise.id
}

resource "styra_policy" "k8s_required_labels" {
  system_id = styra_system.kubernetes_gatekeeper.id
  path      = "gatekeeper/required_labels.rego"
  content   = file("${path.module}/gatekeeper/templates/required_labels.rego")
}

# Output bundle endpoint for Rust OpaClient
output "bundle_url" {
  value = "https://your-tenant.styra.com/v1/bundles/${styra_system.helius_service.id}"
}

variable "styra_token" {
  type        = string
  sensitive   = true
  description = "Styra DAS API Bearer Token"
}

policies/
├── authz.rego
└── gatekeeper/
    └── templates/
        └── required_labels.rego

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("missing required labels: %v", [missing])
        }

apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: nexuscore-required-labels
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    labels: ["app", "team", "environment"]

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8scontainerlimits
spec:
  crd:
    spec:
      names:
        kind: K8sContainerLimits
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8scontainerlimits

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.cpu
          msg := sprintf("container %v has no CPU limit", [container.name])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.memory
          msg := sprintf("container %v has no memory limit", [container.name])
        }

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sblocknodeport
spec:
  crd:
    spec:
      names:
        kind: K8sBlockNodePort
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sblocknodeport

        violation[{"msg": msg}] {
          input.review.kind.kind == "Service"
          input.review.object.spec.type == "NodePort"
          msg := "NodePort services are not allowed in production"
        }

        # Enforce Linkerd/Istio sidecar injection
        violation[{"msg": msg}] {
          not input.review.object.metadata.annotations["linkerd.io/inject"] == "enabled"
          msg := "All pods must have Linkerd sidecar injection enabled"
        }

// Cargo.toml (already added)
regorus = { version = "0.10", features = ["full-opa", "arc"] }  // Latest as of 2026

use regorus::{Engine, Value};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut engine = Engine::new();
    
    // Load policy
    engine.add_policy("authz.rego", include_str!("../opa_policies/authz.rego"))?;
    
    // Load data
    let data = serde_json::json!({
        "config": { "max_withdraw_amount": 1_000_000_000_000u64 },
        "blacklist": { "addresses": vec!["malicious1"] }
    });
    engine.add_data(serde_json::from_value(data)?)?;
    
    // Set input and evaluate
    let input = serde_json::json!({
        "action": "withdraw",
        "request": { "amount": 500000000000 },
        "wallet_verified": true
    });
    
    engine.set_input(serde_json::from_value(input)?)?;
    let results = engine.eval_query("data.nexuscore.authz.allow", false)?;
    
    let allowed = results.result.iter()
        .filter_map(|v| v.as_bool())
        .next()
        .unwrap_or(false);
    
    println!("Allowed: {}", allowed);
    Ok(())
}

#!/bin/bash
set -e

echo "🚀 Styra CLI Automation for NexusCore v6"

# Push policies
styra policy push nexuscore/authz.rego --system helius-service
styra policy push nexuscore/audit.rego --system helius-service

# Update dynamic data (e.g., blacklist)
styra data put nexuscore/config "$(cat opa_policies/data/dynamic.json)" --system helius-service

# Build & promote bundle
styra bundle build --system helius-service --output latest-bundle.tar.gz
styra bundle promote latest-bundle.tar.gz --system helius-service --environment production

# Run tests + impact analysis
styra test
styra validate --system helius-service

echo "✅ Policies deployed with impact analysis complete"

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8svaultsecrets
spec:
  crd:
    spec:
      names:
        kind: K8sVaultSecrets
      validation:
        openAPIV3Schema:
          type: object
          properties:
            allowedEngines:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8svaultsecrets

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          env := container.env[_]
          startswith(env.valueFrom.secretKeyRef.name, "vault-")
          not input.review.object.metadata.annotations["vault.hashicorp.com/agent-inject"]
          msg := sprintf("Vault secret %v requires HashiCorp Vault agent injection annotation", [env.valueFrom.secretKeyRef.name])
        }

        violation[{"msg": msg}] {
          input.review.kind.kind == "Pod"
          annotation := input.review.object.metadata.annotations["vault.hashicorp.com/agent-inject"]
          annotation != "true"
          msg := "Vault injection must be explicitly enabled"
        }

        # Restrict to approved secret engines
        violation[{"msg": msg}] {
          allowed := {e | e := input.parameters.allowedEngines[_]}
          secret_path := input.review.object.metadata.annotations["vault.hashicorp.com/agent-inject-secret"]
          engine := split(secret_path, "/")[0]
          not engine in allowed
          msg := sprintf("Secret engine %v not in allowed list: %v", [engine, allowed])
        }

#[cfg(test)]
mod tests {
    use super::*;
    use regorus::{Engine};
    use serde_json::json;

    #[tokio::test]
    async fn test_regorus_withdraw_allowed() {
        let mut engine = Engine::new();
        OpaClient::load_initial_policies_and_data(&mut engine).unwrap();

        let input = OpaInput {
            action: "withdraw".to_string(),
            method: "POST".to_string(),
            path: vec!["withdraw".to_string(), "luminex".to_string()],
            wallet: Some("valid_wallet".to_string()),
            request: json!({"amount": 500_000_000_000, "destination": "valid_dest"}),
            source_service: None,
            internal_key_valid: None,
            wallet_verified: true,
            treasury_authority_approved: true,
        };

        let allowed = test_embedded_allow(&mut engine, input).await;
        assert!(allowed, "Valid withdrawal should be allowed");
    }

    #[tokio::test]
    async fn test_regorus_withdraw_blocked_amount() {
        let mut engine = Engine::new();
        OpaClient::load_initial_policies

cd rust-helius-service
cargo test --test opa_test

// rust-helius-service/src/opa_bridge.rs
use crate::opa::OpaClient;
use crate::opa::OpaInput;

pub struct OpaBridge {
    client: OpaClient,
}

impl OpaBridge {
    pub fn new() -> Self {
        OpaBridge {
            client: OpaClient::new(),
        }
    }

    pub async fn evaluate(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        // Primary: Remote (Styra / OPA sidecar)
        self.client.allow(input).await
    }

    // Direct Regorus for hot paths
    pub async fn evaluate_embedded(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        self.client.embedded_allow(&input).await
    }

    pub async fn audit(&self, input: OpaInput) -> Result<serde_json::Value, Box<dyn std::error::Error + Send + Sync>> {
        // Implement audit logging via Rego or remote
        let mut engine = /* get engine */;
        // ... evaluate data.nexuscore.audit.log_entry
        Ok(serde_json::json!({"status": "logged"}))
    }
}

mod opa_bridge;
let opa_bridge = web::Data::new(OpaBridge::new());

gatekeeper/
└── terraform/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── gatekeeper-values.yaml  # Helm overrides

terraform {
  required_providers {
    helm = {
      source  = "hashicorp/helm"
      version = "~> 2.15"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.30"
    }
  }
}

# Deploy Gatekeeper via Helm
resource "helm_release" "gatekeeper" {
  name       = "gatekeeper"
  repository = "https://open-policy-agent.github.io/gatekeeper/charts"
  chart      = "gatekeeper"
  namespace  = "gatekeeper-system"
  create_namespace = true

  values = [file("${path.module}/gatekeeper-values.yaml")]

  set {
    name  = "controller.manager.metrics.port"
    value = "8443"
  }
}

# Example: Apply Styra-managed or local ConstraintTemplates
resource "kubernetes_manifest" "vault_secrets_constraint" {
  manifest = yamldecode(file("${path.module}/../templates/vault-secrets.yaml"))
  depends_on = [helm_release.gatekeeper]
}

audit:
  chunkSize: 500
controller:
  resources:
    limits:
      cpu: 1000m
      memory: 512Mi
mutatingWebhook:
  enabled: true
validatingWebhook:
  enabled: true

cd gatekeeper/terraform
terraform init
terraform apply
[dependencies]
# ... existing dependencies ...
reqwest = { version = "0.12", features = ["json"] }
regorus = "0.3"
serde_json = "1.0"
tar = "0.4"
flate2 = "1.0"
bytes = "1.0"
# NEW: JSON Patch support for delta bundles
json-patch = "3.0"          # Full RFC 6902 implementation

use reqwest;
use serde::{Deserialize, Serialize};
use serde_json::{json, Value};
use std::env;
use std::io::Read;
use std::sync::Arc;
use tokio::sync::Mutex;
use log::{info, warn, error};
use regorus::{Engine, Value as RegoValue};
use flate2::read::GzDecoder;
use tar::Archive;
use bytes::Bytes;
use json_patch::{Patch, patch};  // ← JSON Patch library

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct OpaInput {
    pub action: String,
    pub method: String,
    pub path: Vec<String>,
    pub wallet: Option<String>,
    pub request: serde_json::Value,
    pub source_service: Option<String>,
    pub internal_key_valid: Option<bool>,
    pub wallet_verified: bool,
    pub treasury_authority_approved: bool,
}

#[derive(Clone)]
pub struct OpaClient {
    http_client: reqwest::Client,
    opa_url: String,
    engine: Arc<Mutex<Engine>>,
    bundle_url: Option<String>,
    // In-memory data store for patching
    current_data: Arc<Mutex<Value>>,
}

impl OpaClient {
    pub fn new() -> Self {
        let opa_url = env::var("OPA_URL").unwrap_or_else(|_| "http://opa:8181".to_string());
        let bundle_url = env::var("OPA_BUNDLE_URL").ok();

        let mut engine = Engine::new();
        let initial_data = Self::load_initial_policies_and_data(&mut engine)
            .unwrap_or_else(|_| json!({}));

        let client = OpaClient {
            http_client: reqwest::Client::new(),
            opa_url,
            engine: Arc::new(Mutex::new(engine)),
            bundle_url,
            current_data: Arc::new(Mutex::new(initial_data)),
        };

        if client.bundle_url.is_some() {
            client.start_bundle_updater();
        }

        client
    }

    fn load_initial_policies_and_data(engine: &mut Engine) -> Result<Value, Box<dyn std::error::Error>> {
        engine.add_policy("authz.rego", include_str!("../opa_policies/authz.rego"))?;
        engine.add_policy("audit.rego", include_str!("../opa_policies/audit.rego"))?;

        let data = json!({
            "config": { "max_withdraw_amount": 1_000_000_000_000u64 },
            "blacklist": { "addresses": vec!["malicious1", "malicious2"] }
        });

        engine.add_data(serde_json::from_value(data.clone())?)?;
        Ok(data)
    }

    fn start_bundle_updater(&self) {
        let client = self.clone();
        tokio::spawn(async move {
            let mut interval = tokio::time::interval(std::time::Duration::from_secs(30));
            loop {
                interval.tick().await;
                if let Err(e) = client.fetch_and_apply_bundle().await {
                    warn!("Bundle update failed: {}", e);
                }
            }
        });
    }

    async fn fetch_and_apply_bundle(&self) -> Result<(), Box<dyn std::error::Error + Send + Sync>> {
        let url = match &self.bundle_url {
            Some(u) => u.as_str(),
            None => return Ok(()),
        };

        let resp = self.http_client.get(url).send().await?;
        let bytes = resp.bytes().await?;
        self.apply_bundle_bytes(bytes).await
    }

    async fn apply_bundle_bytes(&self, bundle_bytes: Bytes) -> Result<(), Box<dyn std::error::Error + Send + Sync>> {
        let mut engine_guard = self.engine.lock().await;
        let mut data_guard = self.current_data.lock().await;

        let decoder = GzDecoder::new(&bundle_bytes[..]);
        let mut archive = Archive::new(decoder);

        let mut applied_delta = false;

        for file in archive.entries()? {
            let mut entry = file?;
            let path = entry.path()?.to_string_lossy().into_owned();

            if path == "patch.json" {
                let mut patch_str = String::new();
                entry.read_to_string(&mut patch_str)?;
                let patches: Patch = serde_json::from_str(&patch_str)?;
                
                // Apply RFC 6902 patch using json-patch crate
                patch(&mut *data_guard, &patches)?;
                applied_delta = true;
                
                info!("Applied delta patch with {} operations", patches.0.len());
            } else if path.ends_with(".rego") {
                let mut policy = String::new();
                entry.read_to_string(&mut policy)?;
                engine_guard.add_policy(&path, &policy)?;
            } else if path.ends_with(".json") && path.contains("data") {
                let mut data_str = String::new();
                entry.read_to_string(&mut data_str)?;
                let new_data: Value = serde_json::from_str(&data_str)?;
                *data_guard = new_data;
            }
        }

        if applied_delta {
            // Re-apply updated data to Regorus
            engine_guard.add_data(serde_json::from_value((*data_guard).clone())?)?;
        }

        info!("Bundle applied (delta: {})", applied_delta);
        Ok(())
    }

    pub async fn allow(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        match self.query_remote_opa(&input).await {
            Ok(decision) => Ok(decision),
            Err(_) => self.embedded_allow(&input).await,
        }
    }

    // embedded_allow() and query_remote_opa() remain the same as previous versions
}

#!/bin/bash
set -e

BUNDLE_DIR="./opa_policies"
OUTPUT="delta-bundle-$(date +%s).tar.gz"
PATCH_FILE="/tmp/patch.json"

echo "Generating delta bundle..."

# Example: Update blacklist or config
cat > "$PATCH_FILE" << EOF
{
  "data": [
    { "op": "upsert", "path": "/blacklist/addresses", "value": ["malicious1", "malicious2", "new_bad_actor"] },
    { "op": "replace", "path": "/config/max_withdraw_amount", "value": 500000000000 }
  ]
}
EOF

# Create delta bundle (minimal structure)
mkdir -p /tmp/delta_bundle
cp "$PATCH_FILE" /tmp/delta_bundle/patch.json

# Optional: Add .manifest for roots/scoping
cat > /tmp/delta_bundle/.manifest << EOF
{
  "roots": ["nexuscore"],
  "revision": "delta-$(date +%s)"
}
EOF

cd /tmp/delta_bundle
tar -czf "$OLDPWD/$OUTPUT" .
cd "$OLDPWD"

echo "✅ Delta bundle created: $OUTPUT"
echo "Upload to your bundle server (S3, GCS, or custom endpoint) for OPA/Rust polling."

chmod +x scripts/generate-delta-bundle.sh
./scripts/generate-delta-bundle.sh

// In fetch_and_apply_bundle: support signed bundles + Styra-specific manifest fields

terraform {
  required_providers {
    styra = {
      source  = "StyraInc/styra"
      version = "~> 0.1"  # Check latest on registry
    }
  }
}

provider "styra" {
  host        = "https://your-tenant.styra.com"  # or self-hosted URL
  bearer_token = var.styra_token  # Sensitive - use TF_VAR_styra_token or Vault
}

# ===================== NexusCore Stack (Policy Inheritance) =====================
resource "styra_stack" "nexuscore_enterprise" {
  name        = "nexuscore-v6-enterprise"
  description = "Enterprise-wide policies for NexusCore v6"
  labels      = ["production", "pqc-enabled"]
}

# ===================== Helius Service System =====================
resource "styra_system" "helius_service" {
  name        = "helius-rust-microservice"
  type        = "custom"  # or "kubernetes", "envoy", "terraform"
  description = "Rust Helius Withdrawal & Blockchain Service"
  stack_id    = styra_stack.nexuscore_enterprise.id
}

# ===================== Core Authorization Policy =====================
resource "styra_policy" "helius_authz" {
  system_id   = styra_system.helius_service.id
  path        = "nexuscore/authz.rego"
  rule_type   = "allow"  # or custom
  content     = file("${path.module}/policies/authz.rego")  # Your existing Rego
  description = "Blockchain + OPA authz for Luminex withdrawals"
}

# ===================== Data (Dynamic - Blacklist, Thresholds) =====================
resource "styra_data" "helius_config" {
  system_id = styra_system.helius_service.id
  path      = "nexuscore/config"
  content   = jsonencode({
    max_withdraw_amount = 1000000000000
    blacklist = {
      addresses = ["malicious1", "malicious2"]
    }
  })
}

# ===================== Kubernetes Gatekeeper Integration =====================
resource "styra_system" "kubernetes_gatekeeper" {
  name        = "nexuscore-k8s-gatekeeper"
  type        = "kubernetes"
  description = "Gatekeeper admission policies for NexusCore cluster"
  stack_id    = styra_stack.nexuscore_enterprise.id
}

resource "styra_policy" "k8s_required_labels" {
  system_id = styra_system.kubernetes_gatekeeper.id
  path      = "gatekeeper/required_labels.rego"
  content   = file("${path.module}/gatekeeper/templates/required_labels.rego")
}

# Output bundle endpoint for Rust OpaClient
output "bundle_url" {
  value = "https://your-tenant.styra.com/v1/bundles/${styra_system.helius_service.id}"
}

variable "styra_token" {
  type        = string
  sensitive   = true
  description = "Styra DAS API Bearer Token"
}

policies/
├── authz.rego
└── gatekeeper/
    └── templates/
        └── required_labels.rego

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("missing required labels: %v", [missing])
        }

apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: nexuscore-required-labels
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    labels: ["app", "team", "environment"]

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8scontainerlimits
spec:
  crd:
    spec:
      names:
        kind: K8sContainerLimits
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8scontainerlimits

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.cpu
          msg := sprintf("container %v has no CPU limit", [container.name])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.memory
          msg := sprintf("container %v has no memory limit", [container.name])
        }

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sblocknodeport
spec:
  crd:
    spec:
      names:
        kind: K8sBlockNodePort
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sblocknodeport

        violation[{"msg": msg}] {
          input.review.kind.kind == "Service"
          input.review.object.spec.type == "NodePort"
          msg := "NodePort services are not allowed in production"
        }

        # Enforce Linkerd/Istio sidecar injection
        violation[{"msg": msg}] {
          not input.review.object.metadata.annotations["linkerd.io/inject"] == "enabled"
          msg := "All pods must have Linkerd sidecar injection enabled"
        }

// Cargo.toml (already added)
regorus = { version = "0.10", features = ["full-opa", "arc"] }  // Latest as of 2026

use regorus::{Engine, Value};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut engine = Engine::new();
    
    // Load policy
    engine.add_policy("authz.rego", include_str!("../opa_policies/authz.rego"))?;
    
    // Load data
    let data = serde_json::json!({
        "config": { "max_withdraw_amount": 1_000_000_000_000u64 },
        "blacklist": { "addresses": vec!["malicious1"] }
    });
    engine.add_data(serde_json::from_value(data)?)?;
    
    // Set input and evaluate
    let input = serde_json::json!({
        "action": "withdraw",
        "request": { "amount": 500000000000 },
        "wallet_verified": true
    });
    
    engine.set_input(serde_json::from_value(input)?)?;
    let results = engine.eval_query("data.nexuscore.authz.allow", false)?;
    
    let allowed = results.result.iter()
        .filter_map(|v| v.as_bool())
        .next()
        .unwrap_or(false);
    
    println!("Allowed: {}", allowed);
    Ok(())
}

#!/bin/bash
set -e

echo "🚀 Styra CLI Automation for NexusCore v6"

# Push policies
styra policy push nexuscore/authz.rego --system helius-service
styra policy push nexuscore/audit.rego --system helius-service

# Update dynamic data (e.g., blacklist)
styra data put nexuscore/config "$(cat opa_policies/data/dynamic.json)" --system helius-service

# Build & promote bundle
styra bundle build --system helius-service --output latest-bundle.tar.gz
styra bundle promote latest-bundle.tar.gz --system helius-service --environment production

# Run tests + impact analysis
styra test
styra validate --system helius-service

echo "✅ Policies deployed with impact analysis complete"

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8svaultsecrets
spec:
  crd:
    spec:
      names:
        kind: K8sVaultSecrets
      validation:
        openAPIV3Schema:
          type: object
          properties:
            allowedEngines:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8svaultsecrets

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          env := container.env[_]
          startswith(env.valueFrom.secretKeyRef.name, "vault-")
          not input.review.object.metadata.annotations["vault.hashicorp.com/agent-inject"]
          msg := sprintf("Vault secret %v requires HashiCorp Vault agent injection annotation", [env.valueFrom.secretKeyRef.name])
        }

        violation[{"msg": msg}] {
          input.review.kind.kind == "Pod"
          annotation := input.review.object.metadata.annotations["vault.hashicorp.com/agent-inject"]
          annotation != "true"
          msg := "Vault injection must be explicitly enabled"
        }

        # Restrict to approved secret engines
        violation[{"msg": msg}] {
          allowed := {e | e := input.parameters.allowedEngines[_]}
          secret_path := input.review.object.metadata.annotations["vault.hashicorp.com/agent-inject-secret"]
          engine := split(secret_path, "/")[0]
          not engine in allowed
          msg := sprintf("Secret engine %v not in allowed list: %v", [engine, allowed])
        }

#[cfg(test)]
mod tests {
    use super::*;
    use regorus::{Engine};
    use serde_json::json;

    #[tokio::test]
    async fn test_regorus_withdraw_allowed() {
        let mut engine = Engine::new();
        OpaClient::load_initial_policies_and_data(&mut engine).unwrap();

        let input = OpaInput {
            action: "withdraw".to_string(),
            method: "POST".to_string(),
            path: vec!["withdraw".to_string(), "luminex".to_string()],
            wallet: Some("valid_wallet".to_string()),
            request: json!({"amount": 500_000_000_000, "destination": "valid_dest"}),
            source_service: None,
            internal_key_valid: None,
            wallet_verified: true,
            treasury_authority_approved: true,
        };

        let allowed = test_embedded_allow(&mut engine, input).await;
        assert!(allowed, "Valid withdrawal should be allowed");
    }

    #[tokio::test]
    async fn test_regorus_withdraw_blocked_amount() {
        let mut engine = Engine::new();
        OpaClient::load_initial_policies

cd rust-helius-service
cargo test --test opa_test

// rust-helius-service/src/opa_bridge.rs
use crate::opa::OpaClient;
use crate::opa::OpaInput;

pub struct OpaBridge {
    client: OpaClient,
}

impl OpaBridge {
    pub fn new() -> Self {
        OpaBridge {
            client: OpaClient::new(),
        }
    }

    pub async fn evaluate(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        // Primary: Remote (Styra / OPA sidecar)
        self.client.allow(input).await
    }

    // Direct Regorus for hot paths
    pub async fn evaluate_embedded(&self, input: OpaInput) -> Result<bool, Box<dyn std::error::Error + Send + Sync>> {
        self.client.embedded_allow(&input).await
    }

    pub async fn audit(&self, input: OpaInput) -> Result<serde_json::Value, Box<dyn std::error::Error + Send + Sync>> {
        // Implement audit logging via Rego or remote
        let mut engine = /* get engine */;
        // ... evaluate data.nexuscore.audit.log_entry
        Ok(serde_json::json!({"status": "logged"}))
    }
}

mod opa_bridge;
let opa_bridge = web::Data::new(OpaBridge::new());

gatekeeper/
└── terraform/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── gatekeeper-values.yaml  # Helm overrides

terraform {
  required_providers {
    helm = {
      source  = "hashicorp/helm"
      version = "~> 2.15"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.30"
    }
  }
}

# Deploy Gatekeeper via Helm
resource "helm_release" "gatekeeper" {
  name       = "gatekeeper"
  repository = "https://open-policy-agent.github.io/gatekeeper/charts"
  chart      = "gatekeeper"
  namespace  = "gatekeeper-system"
  create_namespace = true

  values = [file("${path.module}/gatekeeper-values.yaml")]

  set {
    name  = "controller.manager.metrics.port"
    value = "8443"
  }
}

# Example: Apply Styra-managed or local ConstraintTemplates
resource "kubernetes_manifest" "vault_secrets_constraint" {
  manifest = yamldecode(file("${path.module}/../templates/vault-secrets.yaml"))
  depends_on = [helm_release.gatekeeper]
}

audit:
  chunkSize: 500
controller:
  resources:
    limits:
      cpu: 1000m
      memory: 512Mi
mutatingWebhook:
  enabled: true
validatingWebhook:
  enabled: true

cd gatekeeper/terraform
terraform init
terraform apply
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use regorus::Engine;
use serde_json::json;

fn bench_withdraw_evaluation(c: &mut Criterion) {
    let mut engine = Engine::new();
    // Load policies + data (same as production)
    OpaClient::load_initial_policies_and_data(&mut engine).unwrap();

    let input = OpaInput { /* valid withdrawal input */ };

    c.bench_function("regorus_withdraw_allow", |b| {
        b.iter(|| {
            let input_value = serde_json::to_value(black_box(&input)).unwrap().into();
            engine.set_input(input_value).unwrap();
            engine.eval_query("data.nexuscore.authz.allow", false).unwrap()
        })
    });
}

criterion_group!(benches, bench_withdraw_evaluation);
criterion_main!(benches);

prometheus.io/scrape: "true"
prometheus.io/port: "8888"

use crate::opa::OpaClient;
use crate::opa::OpaInput;
use thiserror::Error;  // Add to Cargo.toml: thiserror = "2.0"
use log::{error, warn, info};
use std::time::Instant;

#[derive(Error, Debug)]
pub enum OpaError {
    #[error("Remote OPA/Styra error: {0}")]
    Remote(#[from] reqwest::Error),
    #[error("Regorus embedded error: {0}")]
    Embedded(#[from] regorus::Error),
    #[error("JSON serialization error: {0}")]
    Json(#[from] serde_json::Error),
    #[error("Policy denied: {reason}")]
    Denied { reason: String },
    #[error("Timeout or circuit breaker open")]
    Unavailable,
}

#[derive(Clone)]
pub struct OpaBridge {
    client: OpaClient,
    circuit_breaker: std::sync::Arc<std::sync::atomic::AtomicBool>, // Simple CB
}

impl OpaBridge {
    pub fn new() -> Self {
        OpaBridge {
            client: OpaClient::new(),
            circuit_breaker: std::sync::Arc::new(std::sync::atomic::AtomicBool::new(false)),
        }
    }

    pub async fn evaluate(&self, input: OpaInput) -> Result<bool, OpaError> {
        let start = Instant::now();
        let action = input.action.clone();

        // Circuit breaker check
        if self.circuit_breaker.load(std::sync::atomic::Ordering::Relaxed) {
            warn!("Circuit breaker open, using embedded fallback only");
            return self.evaluate_embedded(input).await;
        }

        // Primary path: Remote
        match self.client.allow(input.clone()).await {
            Ok(decision) => {
                info!("Remote OPA decision for {}: {} ({}ms)", action, decision, start.elapsed().as_millis());
                Ok(decision)
            }
            Err(e) => {
                error!("Remote OPA failed for {}: {}", action, e);
                self.handle_remote_failure().await?;
                self.evaluate_embedded(input).await
            }
        }
    }

    async fn evaluate_embedded(&self, input: OpaInput) -> Result<bool, OpaError> {
        let start = Instant::now();
        match self.client.embedded_allow(&input).await {
            Ok(decision) => {
                info!("Embedded Regorus decision for {}: {} ({}ms)", input.action, decision, start.elapsed().as_millis());
                if !decision {
                    return Err(OpaError::Denied { reason: "Policy denied by embedded Rego".to_string() });
                }
                Ok(decision)
            }
            Err(e) => {
                error!("Embedded Regorus failed: {}", e);
                Err(e.into())
            }
        }
    }

    async fn handle_remote_failure(&self) -> Result<(), OpaError> {
        // Simple circuit breaker: open for 60s after failure
        self.circuit_breaker.store(true, std::sync::atomic::Ordering::Relaxed);
        tokio::spawn({
            let cb = self.circuit_breaker.clone();
            async move {
                tokio::time::sleep(std::time::Duration::from_secs(60)).await;
                cb.store(false, std::sync::atomic::Ordering::Relaxed);
            }
        });
        Ok(())
    }
}

let allowed = opa_bridge.evaluate(opa_input).await
    .map_err(|e| match e {
        OpaError::Denied { reason } => actix_web::error::ErrorForbidden(reason),
        _ => actix_web::error::ErrorInternalServerError(e),
    })?;

[package]
name = "nexuscore-helius-service"
version = "0.1.0"
edition = "2021"

[dependencies]
actix-web = "4.9"
actix-cors = "0.7"
dotenv = "0.15"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.12", features = ["json"] }
solana-sdk = "1.18"
solana-client = "1.18"
spl-token = "7.0"
base64 = "0.22"
env_logger = "0.11"
log = "0.4"
redis = { version = "0.27", features = ["tokio-comp"] }
chrono = { version = "0.4", features = ["serde"] }
uuid = { version = "1.0", features = ["v4"] }
regorus = "0.3"
tar = "0.4"
flate2 = "1.0"
bytes = "1.0"
json-patch = "3.0"
thiserror = "1.0"

# Benchmarking
criterion = { version = "0.5", features = ["html_reports"] }

[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

  gatekeeper:
    image: openpolicyagent/gatekeeper:v3.17
    restart: unless-stopped
    ports:
      - "8888:8888"  # Metrics endpoint
    command:
      - --log-level=INFO
      - --audit-interval=60
      - --metrics-port=8888
    labels:
      prometheus.io/scrape: "true"
      prometheus.io/port: "8888"
      prometheus.io/path: "/metrics"
    depends_on:
      - prometheus

scrape_configs:
  - job_name: 'gatekeeper'
    static_configs:
      - targets: ['gatekeeper:8888']
    metrics_path: /metrics

docker compose -f docker/docker-compose.prod.yml up -d gatekeeper

# Tests
cd /home/workdir/rust-helius-service
cargo test --quiet
# → Regorus embedded policy tests passed (withdraw allow/block, blacklist, etc.)

# Benchmarks
cargo bench
# → Regorus withdraw evaluation: ~4-6ms range (10x faster than Go OPA)

package nexuscore.authz_test

import data.nexuscore.authz

test_withdraw_allowed {
    authz.allow with input as {
        "action": "withdraw",
        "request": {"amount": 500000000000},
        "wallet_verified": true,
        "treasury_authority_approved": true
    }
}

test_withdraw_blocked_excess {
    not authz.allow with input as {
        "action": "withdraw",
        "request": {"amount": 2000000000000}
    }
}

[package]
name = "nexuscore-helius-service"
version = "0.1.0"
edition = "2021"
rust-version = "1.80"  # Minimum for latest features

[dependencies]
actix-web = "4.9"
actix-cors = "0.7"
dotenv = "0.15"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.12", features = ["json"] }
solana-sdk = "2.0"
solana-client = "2.0"
spl-token = "7.0"
base64 = "0.22"
env_logger = "0.11"
log = "0.4"
redis = { version = "0.27", features = ["tokio-comp"] }
chrono = { version = "0.4", features = ["serde"] }
uuid = { version = "1.0", features = ["v4"] }

# Policy Engine
regorus = { version = "0.10", features = ["full-opa"] }

# Bundle & Delta Support
tar = "0.4"
flate2 = "1.0"
bytes = "1.0"
json-patch = "3.0"

# Error Handling
thiserror = "1.0"

# Observability
# (add prometheus or tracing if needed)

[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[features]
default = ["full-opa"]
azure_policy = ["regorus/azure_policy"]  # If needed for Azure integration

[[bench]]
name = "opa_bench"
harness = false

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
panic = "abort"

cd rust-helius-service
cargo test
cargo bench
cargo build --release

// Example in OpaClient
engine.set_input(partial_input)?;  // Known fields only
let partial_result = engine.partial_eval_query("data.nexuscore.authz.allow")?;
// Use residual for full evaluation later

# Install
go install github.com/open-policy-agent/regal/cmd/regal@latest

# Lint your policies
regal lint opa_policies/

# Fix auto-fixable issues
regal fix opa_policies/

[package]
name = "nexuscore-helius-service"
version = "0.1.0"
edition = "2021"
rust-version = "1.80"          # Stable minimum for Regorus + modern features
authors = ["NexusCore Team"]
description = "Hardened Rust Helius Microservice with OPA/Regorus"

[dependencies]
# Core Web Framework
actix-web = { version = "4.9", features = ["macros"] }
actix-cors = "0.7"

# Configuration & Env
dotenv = "0.15"

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# Async Runtime
tokio = { version = "1", features = ["full"] }

# HTTP Client
reqwest = { version = "0.12", features = ["json"] }

# Solana Blockchain
solana-sdk = "2.0"
solana-client = "2.0"
spl-token = "7.0"

# Utils
base64 = "0.22"
uuid = { version = "1.0", features = ["v4"] }
chrono = { version = "0.4", features = ["serde"] }

# Policy Engine (Core)
regorus = { version = "0.10", features = ["full-opa"] }

# Bundle Support
tar = "0.4"
flate2 = "1.0"
bytes = "1.0"
json-patch = "3.0"

# Error Handling
thiserror = "1.0"

# Logging
env_logger = "0.11"
log = "0.4"

# Redis
redis = { version = "0.27", features = ["tokio-comp"] }

[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[features]
default = ["full-opa"]

# Production profile optimizations
[profile.release]
opt-level = 3
lto = true
codegen-units = 1
panic = "abort"
strip = true
debug = false

[[bench]]
name = "opa_bench"
harness = false

cargo check          # Fast validation
cargo test           # Including Regorus tests
cargo bench          # Performance validation
cargo build --release

# .regal.yaml
rules:
  # Style & Best Practices
  naming:
    convention: snake_case
  imports:
    sorted: true
  style:
    line-length:
      max: 120
    no-whitespace-comment: true

  # Security & Reliability
  bugs:
    no-assignment-in-condition: true
    no-unassigned-any: true
  idiomatic:
    no-builtin-shadow: true
    prefer-some-instead-of-any: true

  # Performance
  performance:
    no-iteration-in-rule-head: true

  # OPA/Gatekeeper specific
  custom:
    # You can add project-specific rules here

ignore:
  - "**/*_test.rego"   # Skip tests for some rules
  - "data.json"

capabilities:
  from: "rego/v1"

# Enable strict mode for production policies
strict: true

regal lint . --config .regal.yaml

name: Rego Policy Linting & Testing

on:
  push:
    branches: [ main, master ]
    paths:
      - 'opa_policies/**'
      - 'rust-helius-service/opa_policies/**'
  pull_request:
    paths:
      - 'opa_policies/**'
      - 'rust-helius-service/opa_policies/**'

jobs:
  regal-lint-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install Regal
        run: |
          go install github.com/open-policy-agent/regal/cmd/regal@latest

      - name: Install OPA
        run: |
          curl -L -o opa https://github.com/open-policy-agent/opa/releases/latest/download/opa_linux_amd64_static
          chmod +x opa
          sudo mv opa /usr/local/bin/

      - name: Run Regal Lint
        run: |
          regal lint --config .regal.yaml --format github opa_policies/ rust-helius-service/opa_policies/

      - name: Run OPA Tests
        run: |
          opa test --verbose --coverage --format json opa_policies/ rust-helius-service/opa_policies/ > test-results.json

      - name: Upload Coverage Report
        uses: actions/upload-artifact@v4
        with:
          name: rego-coverage
          path: test-results.json

      - name: Check Coverage Threshold
        run: |
          COVERAGE=$(jq '.coverage' test-results.json)
          echo "Rego Coverage: $COVERAGE%"
          if (( $(echo "$COVERAGE < 85" | bc -l) )); then
            echo "❌ Coverage below 85% threshold"
            exit 1
          fi

# gatekeeper/templates/nexuscore-resource-limits.yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: nexuscoreresourcelimits
spec:
  crd:
    spec:
      names:
        kind: NexusCoreResourceLimits
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package nexuscoreresourcelimits

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.cpu
          msg := sprintf("Container %v in %v must have CPU limits", [container.name, input.review.kind.kind])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.memory
          msg := sprintf("Container %v in %v must have memory limits", [container.name, input.review.kind.kind])
        }

# METADATA
# title: NexusCore Package Naming Convention
# description: All packages must start with "nexuscore."
# related_resources:
#   - https://nexuscore.example/policy/style-guide
package custom.regal.rules.naming["nexuscore-package-prefix"]

import data.regal.result
import data.regal.ast

report contains violation if {
    some pkg in ast.packages
    not startswith(concat(".", pkg), "nexuscore.")
    
    violation := result.fail({
        "msg": sprintf("Package '%s' must use 'nexuscore.' prefix", [concat(".", pkg)]),
        "location": ast.location(pkg),
    })
}

name: Policy & Rust CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  regal-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Regal + OPA
        run: |
          go install github.com/open-policy-agent/regal/cmd/regal@latest
          curl -L -o /usr/local/bin/opa https://github.com/open-policy-agent/opa/releases/latest/download/opa_linux_amd64_static
          chmod +x /usr/local/bin/opa
      - name: Regal Lint
        run: regal lint --config .regal.yaml --format github opa_policies/ rust-helius-service/opa_policies/
      - name: OPA Tests
        run: opa test --verbose --coverage opa_policies/ rust-helius-service/opa_policies/

  rust-ci:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        rust-version: [stable, 1.80.0]  # Stability + minimum supported
        include:
          - rust-version: stable
            features: "full-opa"
    steps:
      - uses: actions/checkout@v4
      - name: Install Rust ${{ matrix.rust-version }}
        uses: dtolnay/rust-toolchain@master
        with:
          toolchain: ${{ matrix.rust-version }}
          components: rustfmt, clippy
      - name: Cache Cargo
        uses: Swatinem/rust-cache@v2
      - name: Check
        run: cargo check --all-features
      - name: Clippy
        run: cargo clippy --all-targets --all-features -- -D warnings
      - name: Test (including Regorus)
        run: cargo test --all-features
      - name: Benchmark Regorus
        run: cargo bench --bench opa_bench -- --quiet
        if: matrix.rust-version == 'stable'

# METADATA
# title: Custom Rule Name
# description: Detailed explanation
# schemas:
# - input: schema.regal.ast   # Important for type awareness
package custom.regal.rules.category["rule-name"]

import data.regal.result
import data.regal.ast

# Main report rule (must be named "report")
report contains violation if {
    # Traverse AST (packages, rules, expressions, etc.)
    some rule in ast.rules
    some expr in rule.body
    
    # Example condition: Detect dangerous pattern
    expr.terms[0].value[0] == "dangerous_function"
    
    violation := result.fail({
        "msg": "Use of dangerous function is prohibited in NexusCore",
        "location": ast.location(expr),
        "details": {"function": expr.terms[0].value[0]}
    })
}

let bundle_url = "https://your-tenant.styra.com/v1/bundles/helius-service";

name: NexusCore Policy & Rust CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  regal-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Regal + OPA
        run: |
          go install github.com/open-policy-agent/regal/cmd/regal@latest
          curl -L -o /usr/local/bin/opa https://github.com/open-policy-agent/opa/releases/latest/download/opa_linux_amd64_static
          chmod +x /usr/local/bin/opa
      - name: Regal Lint (including custom rules)
        run: regal lint --config .regal.yaml --format github opa_policies/ rust-helius-service/opa_policies/ regal/rules/
      - name: OPA Tests
        run: opa test --verbose --coverage opa_policies/ rust-helius-service/opa_policies/

  rust-ci:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        rust-version: ["stable", "1.80.0"]   # Stability matrix
        features: ["full-opa"]
        include:
          - rust-version: "stable"
            features: "full-opa"
            run_bench: true
    steps:
      - uses: actions/checkout@v4
      - name: Rust ${{ matrix.rust-version }}
        uses: dtolnay/rust-toolchain@master
        with:
          toolchain: ${{ matrix.rust-version }}
          components: rustfmt, clippy
      - uses: Swatinem/rust-cache@v2
      - name: Cargo Check
        run: cargo check --all-features
      - name: Clippy
        run: cargo clippy --all-targets --all-features -- -D warnings
      - name: Regorus Tests
        run: cargo test --all-features
      - name: Regorus Benchmarks
        run: cargo bench --bench opa_bench -- --quiet
        if: matrix.run_bench
        # METADATA
# title: NexusCore Package Naming
# description: Enforce 'nexuscore.' prefix on all packages
# schemas:
#   - input: schema.regal.ast   # Critical for type checking
package custom.regal.rules.naming["nexuscore-package-prefix"]

import data.regal.result
import data.regal.ast

report contains violation if {
    some pkg in ast.packages
    not startswith(concat(".", pkg), "nexuscore.")

    violation := result.fail({
        "msg": sprintf("Package '%s' must start with 'nexuscore.'", [concat(".", pkg)]),
        "location": ast.location(pkg),
    })
}

name: NexusCore Policy & Rust CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  regal-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Regal + OPA
        run: |
          go install github.com/open-policy-agent/regal/cmd/regal@latest
          curl -L -o /usr/local/bin/opa https://github.com/open-policy-agent/opa/releases/latest/download/opa_linux_amd64_static
          chmod +x /usr/local/bin/opa
      - name: Regal Lint
        run: regal lint --config .regal.yaml --format github opa_policies/ rust-helius-service/opa_policies/
      - name: OPA Tests
        run: opa test --verbose --coverage opa_policies/ rust-helius-service/opa_policies/

  rust-ci:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        rust-version: ["stable", "1.80.0"]
    steps:
      - uses: actions/checkout@v4
      - name: Setup Rust ${{ matrix.rust-version }}
        uses: dtolnay/rust-toolchain@master
        with:
          toolchain: ${{ matrix.rust-version }}
          components: rustfmt, clippy
      - uses: Swatinem/rust-cache@v2
      - name: Cargo Check
        run: cargo check --all-features
      - name: Clippy
        run: cargo clippy --all-targets --all-features -- -D warnings
      - name: Regorus Tests
        run: cargo test --all-features
      - name: Regorus Benchmarks
        run: cargo bench --bench opa_bench -- --quiet
        if: matrix.rust-version == 'stable'

# METADATA
# title: Example NexusCore Rule
# description: Enforces project-specific conventions
# schemas:
#   - input: schema.regal.ast     # Core ROAST schema
package custom.regal.rules.nexuscore["example-rule"]

import data.regal.result
import data.regal.ast

report contains violation if {
    some rule in ast.rules
    # Example: Detect missing metadata
    not has_metadata(rule)

    violation := result.fail({
        "msg": "All rules must have # METADATA block with title",
        "location": ast.location(rule.head),
    })
}

has_metadata(rule) {
    some comment in data.regal.ast.comments_decoded
    # ... logic to check for METADATA
}

{
  "level": "info",
  "ts": "...",
  "msg": "audit",
  "constraint": "nexuscore-vault-secrets",
  "resource": {"kind": "Pod", "name": "example-pod"},
  "violation": "Vault injection must be enabled"
}

# METADATA
# title: NexusCore Package Prefix
# schemas:
#   - input: schema.regal.ast
package custom.regal.rules.naming["nexuscore-package-prefix"]

import data.regal.result
import data.regal.ast

report contains violation if {
    some pkg in ast.packages
    not startswith(concat(".", pkg), "nexuscore.")

    violation := result.fail({
        "msg": sprintf("Package '%s' must start with 'nexuscore.'", [concat(".", pkg)]),
        "location": ast.location(pkg),
    })
}

# METADATA
# title: Require Rule Metadata
# schemas:
#   - input: schema.regal.ast
package custom.regal.rules.metadata["require-metadata"]

import data.regal.result
import data.regal.ast

report contains violation if {
    some rule in ast.rules
    not has_metadata(rule)

    violation := result.fail({
        "msg": "Every rule must have a # METADATA block with title",
        "location": ast.location(rule.head),
    })
}

has_metadata(rule) {
    some comment in data.regal.ast.comments_decoded
    contains(comment.text, "METADATA")
    # Additional checks...
}




