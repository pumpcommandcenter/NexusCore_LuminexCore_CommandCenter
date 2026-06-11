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
# Dashboard + Metrics: http://localhost:3000
# Grafana: http://localhost:3001
# Prometheus: http://localhost:9090

