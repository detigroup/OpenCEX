# OpenCEX - Tài liệu Đặc tả Toàn bộ Dự án

## 📋 Mục lục
1. [Tổng quan Dự án](#1-tổng-quan-dự-án)
2. [Kiến trúc Hệ thống](#2-kiến-trúc-hệ-thống)
3. [Backend Service](#3-backend-service)
4. [Frontend Application](#4-frontend-application)
5. [Nuxt SSR Static Pages](#5-nuxt-ssr-static-pages)
6. [Admin Dashboard](#6-admin-dashboard)
7. [Infrastructure & DevOps](#7-infrastructure--devops)
8. [Database & Storage](#8-database--storage)
9. [Blockchain Integration](#9-blockchain-integration)
10. [Security & Compliance](#10-security--compliance)
11. [Deployment Guide](#11-deployment-guide)
12. [API Documentation](#12-api-documentation)
13. [Development Workflow](#13-development-workflow)
14. [Monitoring & Logging](#14-monitoring--logging)
15. [Troubleshooting](#15-troubleshooting)

---

## 1. Tổng quan Dự án

### 1.1 Giới thiệu
**OpenCEX** là nền tảng sàn giao dịch tiền điện tử (Cryptocurrency Exchange) mã nguồn mở hoàn chỉnh, được phát triển bởi **Polygant**. Hệ thống cung cấp đầy đủ các tính năng của một sàn giao dịch chuyên nghiệp với kiến trúc microservices.

### 1.2 Thông tin cơ bản
- **License**: Apache License, Version 2.0
- **Developer**: Polygant (https://polygant.net)
- **Documentation**: https://polygant.notion.site/OpenCEX-Help-Center
- **GitHub**: https://github.com/Polygant/OpenCEX/
- **Community**: https://t.me/opencex

### 1.3 Công nghệ chính
```
Backend:     Python 3.8, Django 3.2.23, Django REST Framework
Frontend:    Vue 3, Vite, TypeScript, Tailwind CSS
Admin:       Vue 3, Vuetify 3, Pinia, TypeScript
Nuxt:        Nuxt.js 2, Vue 2 (SSR)
Database:    PostgreSQL 17
Cache:       Redis (latest)
Queue:       RabbitMQ 3.9.22
Proxy:       Caddy (Docker Proxy)
Blockchain:  Bitcoin Core, Web3, Tronpy
```

### 1.4 Kiến trúc Overview
```
┌─────────────────────────────────────────────────────────────┐
│                     Internet / Users                         │
└──────────────────┬──────────────────────────────────────────┘
                   │
        ┌──────────▼──────────┐
        │   Caddy Proxy       │
        │  (Reverse Proxy)    │
        └──────────┬──────────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
┌───▼────┐   ┌────▼────┐   ┌────▼────┐
│ Nuxt   │   │Frontend │   │ Admin   │
│ (SSR)  │   │(Vue3/SPA│   │(Vue3/SPA│
└────────┘   └─────┬───┘   └────┬────┘
                   │             │
            ┌──────▼─────────────▼─────┐
            │   Backend API (Django)    │
            │  - REST API               │
            │  - WebSocket (Daphne)     │
            └──────┬───────────────────┘
                   │
    ┌──────────────┼──────────────┬──────────────┐
    │              │              │              │
┌───▼────┐   ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
│Postgre │   │ Redis   │   │RabbitMQ │   │Blockchain│
│SQL     │   │         │   │         │   │ Nodes   │
└────────┘   └─────────┘   └─────────┘   └─────────┘
                   │
            ┌──────▼─────────────────────┐
            │   Celery Workers (12+)     │
            │  - General Tasks           │
            │  - Blockchain Scanners     │
            │  - Deposits/Withdrawals    │
            │  - Accumulations/Gas       │
            └────────────────────────────┘
```

---

## 2. Kiến trúc Hệ thống

### 2.1 Microservices Architecture

#### 2.1.1 Container Services
```yaml
Services:
  1. opencex              # Main Django Application (Gunicorn)
  2. opencex-wss          # WebSocket Server (Daphne)
  3. opencex-cel          # General Celery Worker
  4. opencex-stack        # Stack Manager
  5. opencex-btc          # Bitcoin Worker
  6. opencex-eth-blocks   # Ethereum Block Scanner
  7. opencex-bnb-blocks   # BNB Chain Block Scanner
  8. opencex-trx-blocks   # Tron Block Scanner
  9. opencex-matic-blocks # Polygon Block Scanner
  10. opencex-deposits    # Deposit Processor
  11. opencex-payouts     # Withdrawal Processor
  12. opencex-balances    # Balance Checker
  13. opencex-coin-accumulations    # Coin Accumulation
  14. opencex-token-accumulations   # Token Accumulation
  15. opencex-gas         # Gas Management
  16. frontend            # Vue 3 Frontend
  17. nuxt                # Nuxt.js SSR
  18. admin               # Admin Dashboard
  19. caddy               # Reverse Proxy
  20. postgresql          # Database
  21. redis               # Cache & Channel Layer
  22. rabbitmq            # Message Broker
  23. bitcoind            # Bitcoin Node
```

### 2.2 Communication Flow
```
User Browser
    ↓
Caddy Reverse Proxy
    ↓
┌─────────────┬─────────────┬─────────────┐
│   Nuxt      │  Frontend   │   Admin     │
│   (SSR)     │   (SPA)     │   (SPA)     │
└─────────────┴─────────────┴─────────────┘
    ↓             ↓             ↓
┌───────────────────────────────────────────┐
│         Django Backend API                 │
│  - HTTP REST API (Gunicorn)               │
│  - WebSocket (Daphne)                     │
└───────────────────────────────────────────┘
    ↓             ↓             ↓
┌─────────┐  ┌─────────┐  ┌─────────┐
│PostgreSQL│  │  Redis  │  │RabbitMQ │
└─────────┘  └─────────┘  └─────────┘
                              ↓
                    ┌─────────────────┐
                    │ Celery Workers  │
                    │  (Background)   │
                    └─────────────────┘
                              ↓
                    ┌─────────────────┐
                    │ Blockchain Nodes│
                    └─────────────────┘
```

### 2.3 Data Flow
```
User Action → Frontend/Admin
    ↓
API Request → Backend (Django)
    ↓
Business Logic → Models/Services
    ↓
┌─────────────┼─────────────┐
│             │             │
Database ← Task Queue → Cache
(PostgreSQL)  (RabbitMQ)  (Redis)
    ↓             ↓
Persistent   Celery Workers
Storage           ↓
            Blockchain Ops
```

---

## 3. Backend Service

### 3.1 Technology Stack
```python
Framework:          Django 3.2.23
API:                Django REST Framework 3.12.4
WebSocket:          Django Channels 4.0.0 + Daphne
Task Queue:         Celery 5.2.7
Database ORM:       Django ORM
Authentication:     JWT (SimpleJWT) + Django OTP
Documentation:      drf-spectacular (OpenAPI 3.0)
```

### 3.2 Project Structure
```
backend/
├── admin_panel/          # Django Admin customization
├── admin_rest/           # Admin REST API
├── bots/                 # Trading bots module
├── core/                 # Core business logic
│   ├── api_urls/        # API routing
│   ├── models/          # Data models
│   │   ├── facade/      # User, Profile, KYC
│   │   ├── inouts/      # Deposits, Withdrawals
│   │   ├── orders/      # Trading orders
│   │   ├── stats/       # Statistics
│   │   └── wallet_history/
│   ├── serializers/     # DRF serializers
│   ├── tasks/           # Celery tasks
│   ├── views/           # API views
│   └── websockets/      # WebSocket consumers
├── cryptocoins/         # Blockchain integration
│   ├── coins/           # Coin implementations
│   ├── evm/             # EVM chains handler
│   ├── models/          # Blockchain models
│   └── tasks/           # Blockchain tasks
├── dashboard_rest/      # Dashboard API
├── exchange/            # Main Django app
│   ├── settings/        # Settings modules
│   ├── wsgi.py
│   ├── asgi.py
│   └── urls.py
├── lib/                 # Shared libraries
├── notifications/       # Notification system
├── public_api/          # Public API
├── sci/                 # Payment gateway
├── seo/                 # SEO management
├── manage.py
├── requirements.txt
└── Dockerfile
```

### 3.3 Core Modules

#### 3.3.1 Core Models
```python
# User & Authentication
Profile                  # User profiles
UserKYC                  # KYC verification
LoginHistory            # Login tracking
TwoFactorSecretTokens   # 2FA management

# Wallet & Balance
UserWallet              # User cryptocurrency wallets
Balance                 # User balances
WalletHistoryItem       # Wallet history

# Trading
Order                   # Buy/Sell orders
Exchange                # Order matching
ExecutionResult         # Trade execution results
Pair                    # Trading pairs
PairSettings            # Trading pair configuration

# Transactions
Transaction             # Blockchain transactions
WithdrawalRequest       # Withdrawal requests
WalletTransactions      # Wallet transactions
PayGateTopup            # Payment gateway deposits

# Fees & Limits
FeesAndLimits          # Fee configuration
WithdrawalFee          # Withdrawal fees
UserFee                # User-specific fees
UserExchangeFee        # Exchange fees per user
```

#### 3.3.2 Cryptocurrency Models
```python
Keeper                      # Hot wallet management
GasKeeper                   # Gas keeper for EVM chains
LastProcessedBlock          # Block tracking
AccumulationTransaction     # Accumulation tracking
ScoringSettings            # Risk scoring config
TransactionInputScore      # Transaction risk scores
```

#### 3.3.3 Bot Models
```python
BotConfig              # Trading bot configuration
OrderMatch             # Bot order matching
OrderMatchStat         # Bot statistics
```

### 3.4 API Endpoints Structure
```
/api/
├── schema/                    # OpenAPI schema
├── schema/swagger-ui/         # Swagger documentation
├── schema/redoc/             # ReDoc documentation
├── v1/                       # API version 1
│   ├── auth/                 # Authentication
│   ├── profile/              # User profile
│   ├── wallets/              # Wallet management
│   ├── deposits/             # Deposit tracking
│   ├── withdrawals/          # Withdrawal requests
│   ├── orders/               # Order management
│   ├── pairs/                # Trading pairs
│   ├── orderbook/            # Order book data
│   ├── trades/               # Trade history
│   ├── ticker/               # Market tickers
│   └── stats/                # Statistics
├── public/v1/                # Public API
└── external/                 # External integrations
    ├── nomics/               # Nomics API format
    └── coingecko/            # CoinGecko API format
```

### 3.5 Celery Task Queues
```python
# General Queue
- Default tasks
- Scheduled tasks (Beat)
- General operations

# Blockchain Scanners (per chain)
- eth_new_blocks
- bnb_new_blocks
- trx_new_blocks
- matic_new_blocks

# Transaction Processing
- eth_deposits, bnb_deposits, trx_deposits, matic_deposits
- eth_payouts, bnb_payouts, trx_payouts, matic_payouts

# Maintenance Tasks
- eth_check_balances, bnb_check_balances, trx_check_balances
- eth_accumulations, bnb_accumulations, trx_accumulations
- eth_tokens_accumulations, bnb_tokens_accumulations, trx_tokens_accumulations
- eth_send_gas, bnb_send_gas, trx_send_gas, matic_send_gas

# Bitcoin Queue
- btc (dedicated worker)
```

### 3.6 WebSocket Endpoints
```
/ws/orderbook/{pair}/       # Real-time order book updates
/ws/ticker/{pair}/          # Price ticker updates
/ws/trades/{pair}/          # Live trade stream
/ws/user/balances/          # User balance updates
/ws/user/orders/            # User order updates
```

---

## 4. Frontend Application

### 4.1 Technology Stack
```javascript
Framework:       Vue 3.2.25
Build Tool:      Vite 2.9.16
Language:        TypeScript 4.5.4
State:           Vuex 4.0.2
Router:          Vue Router 4.0.13
UI:              Tailwind CSS 3.0.24
Charts:          Highcharts 10.0.0
i18n:            Vue I18n 9.1.9
HTTP:            Axios 0.26.1
WebSocket:       vue-native-websocket-vue3
```

### 4.2 Project Structure
```
frontend/
├── src/
│   ├── api/              # API service layer
│   ├── assets/           # Static assets (images, fonts)
│   ├── components/       # Reusable components
│   │   ├── common/       # Common components
│   │   ├── trading/      # Trading components
│   │   ├── wallet/       # Wallet components
│   │   └── ...
│   ├── directives/       # Vue directives
│   ├── helpers/          # Helper functions
│   ├── locales/          # i18n translations
│   ├── mixins/           # Vue mixins
│   ├── modules/          # Feature modules
│   ├── pages/            # Page components
│   │   ├── Account/      # User account pages
│   │   ├── Exchange/     # Trading pages
│   │   ├── Wallet/       # Wallet pages
│   │   └── ...
│   ├── plugins/          # Vue plugins
│   ├── router/           # Route configuration
│   │   └── index.ts
│   ├── static/           # Public static files
│   ├── store/            # Vuex store
│   │   ├── modules/      # Store modules
│   │   └── index.ts
│   ├── utilities/        # Utility functions
│   ├── App.vue           # Root component
│   └── main.ts           # Application entry
├── deploy/               # Deployment configs
├── package.json
├── vite.config.ts
└── tsconfig.json
```

### 4.3 Key Features

#### 4.3.1 Trading Interface
```
- Real-time order book
- TradingView charts integration
- Market/Limit order placement
- Order history
- Trade history
- Portfolio overview
```

#### 4.3.2 Wallet Management
```
- Multi-currency wallets
- Deposit addresses
- Withdrawal forms
- Transaction history
- QR code generation
```

#### 4.3.3 User Account
```
- Profile management
- KYC verification
- 2FA setup
- API key management
- Security settings
- Notification preferences
```

### 4.4 State Management (Vuex)
```javascript
store/
├── modules/
│   ├── auth.ts           # Authentication state
│   ├── user.ts           # User profile
│   ├── wallet.ts         # Wallet data
│   ├── trading.ts        # Trading state
│   ├── orderbook.ts      # Order book data
│   └── websocket.ts      # WebSocket connections
└── index.ts              # Root store
```

### 4.5 API Integration
```typescript
// API Service Structure
api/
├── auth.ts               # Authentication APIs
├── user.ts               # User APIs
├── wallet.ts             # Wallet APIs
├── trading.ts            # Trading APIs
├── orders.ts             # Order APIs
└── config.ts             # API configuration
```

### 4.6 Routing Structure
```javascript
routes:
  /                       # Home/Landing
  /login                  # Login page
  /register               # Registration
  /exchange/:pair         # Trading page
  /wallet                 # Wallet overview
  /wallet/deposit         # Deposit page
  /wallet/withdraw        # Withdrawal page
  /wallet/history         # Transaction history
  /account                # Account settings
  /account/security       # Security settings
  /account/kyc            # KYC verification
  /orders                 # Order history
  /trades                 # Trade history
```

---

## 5. Nuxt SSR Static Pages

### 5.1 Technology Stack
```javascript
Framework:       Nuxt.js 2.15.7
Vue:             Vue 2
Rendering:       Server-Side Rendering (SSR)
Cache:           nuxt-ssr-cache
HTTP:            @nuxtjs/axios
i18n:            @nuxtjs/i18n
Charts:          Highcharts 10.2.1
```

### 5.2 Project Structure
```
nuxt/
├── components/           # Vue components
├── layouts/              # Layout templates
│   ├── default.vue
│   └── error.vue
├── locales/              # i18n translations
│   └── en.js
├── mixins/               # Vue mixins
├── pages/                # File-based routing
│   ├── index.vue         # Homepage
│   ├── about.vue         # About page
│   ├── faq.vue           # FAQ page
│   └── ...
├── plugins/              # Nuxt plugins
│   ├── axios.js
│   └── highcharts.js
├── serverMiddleware/     # Server middleware
├── static/               # Static files
├── store/                # Vuex store
├── styles/               # Global styles
├── nuxt.config.js        # Nuxt configuration
└── package.json
```

### 5.3 Purpose
```
SEO-Optimized Pages:
  - Landing page
  - About us
  - FAQ
  - Terms of service
  - Privacy policy
  - Help center
  - Blog/News
  - Market overview
```

### 5.4 Configuration Highlights
```javascript
// nuxt.config.js
export default {
  target: 'server',           // SSR mode
  
  modules: [
    '@nuxtjs/axios',
    '@nuxtjs/i18n',
    'nuxt-ssr-cache'          // Page caching
  ],
  
  cache: {
    pages: ['/'],             // Cache homepage
    store: {
      type: 'memory',
      max: 100,
      ttl: 60                 // 60 seconds cache
    }
  }
}
```

### 5.5 SSR Benefits
```
✓ Better SEO
✓ Faster initial page load
✓ Social media previews
✓ Search engine indexing
✓ Reduced client-side JS
```

---

## 6. Admin Dashboard

### 6.1 Technology Stack
```javascript
Framework:       Vue 3.3.0
UI Library:      Vuetify 3.1.14
State:           Pinia 2.0.34
Router:          Vue Router 4.1.6
Build:           Vite 4.2.1
Language:        TypeScript 4.9.5
HTTP:            Axios 1.3.5 + vue-axios
Icons:           Material Design Icons (@mdi/js)
Utils:           Lodash, Moment.js
```

### 6.2 Project Structure
```
admin/
├── src/
│   ├── assets/           # Images, styles
│   ├── components/       # Reusable components
│   │   ├── charts/       # Chart components
│   │   ├── forms/        # Form components
│   │   ├── tables/       # Table components
│   │   └── ...
│   ├── pages/            # Admin pages
│   │   ├── Dashboard/    # Main dashboard
│   │   ├── Users/        # User management
│   │   ├── Orders/       # Order management
│   │   ├── Wallets/      # Wallet management
│   │   ├── Transactions/ # Transaction monitoring
│   │   ├── Settings/     # System settings
│   │   └── Reports/      # Reports & analytics
│   ├── plugins/          # Vue plugins
│   ├── router/           # Route configuration
│   │   └── index.ts
│   ├── stores/           # Pinia stores
│   │   ├── auth.ts
│   │   ├── users.ts
│   │   └── ...
│   ├── App.vue
│   ├── main.ts
│   └── local_config.ts   # Local configuration
├── deploy/
├── public/
├── package.json
├── vite.config.ts
└── tsconfig.json
```

### 6.3 Key Features

#### 6.3.1 Dashboard Overview
```
- Real-time metrics
- User statistics
- Trading volume charts
- Revenue analytics
- Active orders count
- System health status
```

#### 6.3.2 User Management
```
- User list with search/filter
- User details & profile
- KYC verification review
- Account status management
- Balance adjustment
- Fee customization
- Activity logs
```

#### 6.3.3 Order Management
```
- Active orders list
- Order history
- Order details
- Manual order cancellation
- Trade history
- Matched orders
```

#### 6.3.4 Wallet Management
```
- All wallets overview
- Deposit monitoring
- Withdrawal approval
- Balance auditing
- Address management
- Accumulation status
```

#### 6.3.5 Transaction Monitoring
```
- Blockchain transaction tracking
- Deposit confirmations
- Withdrawal status
- Failed transactions
- Transaction details
- Re-broadcasting
```

#### 6.3.6 System Settings
```
- Trading pair configuration
- Fee settings
- Limits configuration
- Coin enable/disable
- Maintenance mode
- Email templates
- Notification settings
```

#### 6.3.7 Reports & Analytics
```
- Daily/Monthly reports
- Revenue reports
- User growth analytics
- Trading volume analysis
- Fee collection reports
- Export to CSV/Excel
```

### 6.4 State Management (Pinia)
```typescript
stores/
├── auth.ts               # Admin authentication
├── users.ts              # User management
├── orders.ts             # Order data
├── wallets.ts            # Wallet data
├── transactions.ts       # Transaction data
└── settings.ts           # System settings
```

### 6.5 Routing Structure
```javascript
routes:
  /admin                    # Dashboard home
  /admin/users              # User list
  /admin/users/:id          # User details
  /admin/orders             # Orders list
  /admin/wallets            # Wallets overview
  /admin/deposits           # Deposits monitoring
  /admin/withdrawals        # Withdrawals approval
  /admin/transactions       # Transaction tracking
  /admin/pairs              # Trading pairs config
  /admin/fees               # Fee settings
  /admin/settings           # System settings
  /admin/reports            # Reports
```

---

## 7. Infrastructure & DevOps

### 7.1 Docker Compose Architecture

#### 7.1.1 Network Configuration
```yaml
networks:
  caddy:
    external: true          # External Caddy network
```

#### 7.1.2 Service Dependencies
```
Dependency Tree:
  caddy (base)
  ├── postgresql
  ├── redis
  ├── rabbitmq
  ├── bitcoind
  ├── frontend
  ├── nuxt
  └── admin
      └── opencex (main app)
          ├── opencex-wss (WebSocket)
          ├── opencex-cel (Celery general)
          ├── opencex-stack
          ├── opencex-btc
          ├── opencex-*-blocks (4 workers)
          ├── opencex-deposits
          ├── opencex-payouts
          ├── opencex-balances
          ├── opencex-coin-accumulations
          ├── opencex-token-accumulations
          └── opencex-gas
```

### 7.2 Container Details

#### 7.2.1 Backend Containers
```yaml
opencex:
  image: opencex:latest
  command: gunicorn exchange.wsgi:application -b 0.0.0.0:8080 -w 2
  volumes: /app/opencex/backend:/app
  
opencex-wss:
  command: daphne -b 0.0.0.0 exchange.asgi:application
  
opencex-cel:
  command: celery -A exchange worker -l info -n general -B
  
opencex-btc:
  command: python manage.py btcworker
  
opencex-eth-blocks:
  command: celery -A exchange worker -Q eth_new_blocks -c 1
```

#### 7.2.2 Frontend Containers
```yaml
frontend:
  image: frontend:latest
  port: 80
  
nuxt:
  image: nuxt:latest
  
admin:
  image: admin:latest
```

#### 7.2.3 Infrastructure Containers
```yaml
postgresql:
  image: postgres:17
  volumes: ./postgresql_data:/var/lib/postgresql/data
  environment:
    POSTGRES_USER: opencex
    POSTGRES_PASSWORD: ***
    POSTGRES_DB: opencex
    
redis:
  image: redis:latest
  volumes: ./redis_data:/data
  
rabbitmq:
  image: rabbitmq:3.9.22-management
  volumes:
    - ./rabbitmq_data:/var/lib/rabbitmq/
    - ./rabbitmq_logs:/var/log/rabbitmq/
  ports: 15672 (management UI)
  
bitcoind:
  image: lncm/bitcoind:v24.0.1
  volumes: ./bitcoind_data:/data/.bitcoin/
```

#### 7.2.4 Reverse Proxy
```yaml
caddy:
  image: lucaslorentz/caddy-docker-proxy:latest
  ports:
    - 80:80
    - 443:443
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - ./caddy_data:/data
  environment:
    CADDY_INGRESS_NETWORKS: caddy
```

### 7.3 Caddy Labels Configuration
```yaml
# Frontend with custom domain
frontend:
  labels:
    caddy: detidex.yeuthich.net
    caddy.reverse_proxy: "{{upstreams 80}}"

# RabbitMQ Management UI
rabbitmq:
  labels:
    caddy: 
    caddy.reverse_proxy: "{{upstreams http 15672}}"
```

### 7.4 Volume Management
```
Data Persistence:
  - postgresql_data/      # Database data
  - redis_data/          # Redis persistence
  - rabbitmq_data/       # RabbitMQ data
  - rabbitmq_logs/       # RabbitMQ logs
  - bitcoind_data/       # Bitcoin blockchain data
  - caddy_data/          # Caddy SSL certificates
  - /app/opencex/backend # Backend source code
```

### 7.5 Build Process

#### 7.5.1 Backend Build
```dockerfile
FROM python:3.8
ENV PYTHONUNBUFFERED 1
WORKDIR /app
COPY requirements.txt /tmp/
RUN pip install --upgrade pip
RUN pip install --no-cache-dir -r /tmp/requirements.txt
RUN apt-get update && apt-get install systemd gettext -y
```

#### 7.5.2 Frontend Build
```bash
# Build Vue 3 Frontend
cd frontend
npm install
npm run build
docker build -t frontend:latest .
```

#### 7.5.3 Admin Build
```bash
# Build Admin Dashboard
cd admin
npm install
npm run build
docker build -t admin:latest .
```

#### 7.5.4 Nuxt Build
```bash
# Build Nuxt SSR
cd nuxt
npm install
npm run build
docker build -t nuxt:latest .
```

### 7.6 Deployment Commands
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f [service_name]

# Restart service
docker-compose restart [service_name]

# Stop all services
docker-compose down

# Rebuild and restart
docker-compose up -d --build [service_name]

# Scale workers
docker-compose up -d --scale opencex-cel=3
```

---

## 8. Database & Storage

### 8.1 PostgreSQL Schema

#### 8.1.1 Core Tables
```sql
-- User & Authentication
auth_user
core_profile
core_userkyc
core_loginhistory
core_accesslog
core_twofactorsecrettokens

-- Wallet & Balance
core_userwallet
core_balance
core_wallethistoryitem

-- Trading
core_order
core_exchange
core_executionresult
core_pair
core_pairsettings

-- Transactions
core_transaction
core_withdrawalrequest
core_wallettransactions
core_paygatetopup

-- Fees
core_feesandlimits
core_withdrawalfee
core_userfee
core_userexchangefee

-- Blockchain
cryptocoins_keeper
cryptocoins_gaskeeper
cryptocoins_lastprocessedblock
cryptocoins_accumulationtransaction

-- Bots
bots_botconfig
bots_ordermatch
bots_ordermatchstat

-- Statistics
core_externalpriceshistory
core_tradesaggregatedstats
core_userpairdailystat
```

#### 8.1.2 Indexes
```sql
-- Performance indexes
CREATE INDEX idx_order_user_pair ON core_order(user_id, pair_id);
CREATE INDEX idx_transaction_status ON core_transaction(status);
CREATE INDEX idx_balance_user_currency ON core_balance(user_id, currency);
CREATE INDEX idx_withdrawal_status ON core_withdrawalrequest(status);
```

### 8.2 Redis Usage

#### 8.2.1 Cache Keys
```python
# User sessions
session:{session_key}

# API rate limiting
ratelimit:{user_id}:{endpoint}

# Order book cache
orderbook:{pair_id}

# Ticker cache
ticker:{pair_id}

# User balance cache
balance:{user_id}:{currency}
```

#### 8.2.2 Channels (for WebSocket)
```python
# Order book updates
orderbook.{pair_id}

# Ticker updates
ticker.{pair_id}

# Trade stream
trades.{pair_id}

# User notifications
user.{user_id}.notifications
```

### 8.3 RabbitMQ Queues

#### 8.3.1 Exchange & Queue Structure
```
Exchange: celery (default)
  ├── Queue: celery (general tasks)
  ├── Queue: btc (bitcoin tasks)
  ├── Queue: eth_new_blocks
  ├── Queue: bnb_new_blocks
  ├── Queue: trx_new_blocks
  ├── Queue: matic_new_blocks
  ├── Queue: eth_deposits
  ├── Queue: bnb_deposits
  ├── Queue: trx_deposits
  ├── Queue: matic_deposits
  ├── Queue: eth_payouts
  ├── Queue: bnb_payouts
  ├── Queue: trx_payouts
  ├── Queue: matic_payouts
  ├── Queue: eth_check_balances
  ├── Queue: bnb_check_balances
  ├── Queue: trx_check_balances
  ├── Queue: matic_check_balances
  ├── Queue: eth_accumulations
  ├── Queue: bnb_accumulations
  ├── Queue: trx_accumulations
  ├── Queue: matic_accumulations
  ├── Queue: eth_tokens_accumulations
  ├── Queue: bnb_tokens_accumulations
  ├── Queue: trx_tokens_accumulations
  ├── Queue: matic_tokens_accumulations
  ├── Queue: eth_send_gas
  ├── Queue: bnb_send_gas
  ├── Queue: trx_send_gas
  └── Queue: matic_send_gas
```

#### 8.3.2 Task Routing
```python
# Route tasks to specific queues
CELERY_TASK_ROUTES = {
    'cryptocoins.tasks.eth.*': {'queue': 'eth_new_blocks'},
    'cryptocoins.tasks.bnb.*': {'queue': 'bnb_new_blocks'},
    # ... etc
}
```

---

## 9. Blockchain Integration

### 9.1 Supported Blockchains

#### 9.1.1 Bitcoin (BTC)
```python
Node:         Bitcoin Core 24.0.1
Connection:   RPC (python-bitcoinrpc)
Features:
  - UTXO-based transactions
  - HD wallet generation
  - Multi-signature support
  - Transaction monitoring
  - Fee estimation
```

#### 9.1.2 Ethereum (ETH)
```python
Provider:     Infura / Custom RPC
Library:      Web3.py 6.2.0
Features:
  - Account creation
  - ERC20 token support
  - Gas management
  - Smart contract interaction
  - Event listening
  - Transaction tracking
```

#### 9.1.3 BNB Chain (BNB)
```python
Provider:     BSC RPC nodes
Library:      Web3.py (EVM compatible)
API:          BscScan API
Features:
  - BEP20 token support
  - Gas optimization
  - Fast block time (3s)
  - Low transaction fees
```

#### 9.1.4 Tron (TRX)
```python
Provider:     TronGrid
Library:      Tronpy 0.4.0
Features:
  - TRC20 token support
  - Energy/Bandwidth management
  - Fast confirmations
  - Fee-less transfers (with energy)
```

#### 9.1.5 Polygon (MATIC)
```python
Provider:     Polygon RPC
Library:      Web3.py
API:          PolygonScan API
Features:
  - EVM compatible
  - Low fees
  - Fast transactions
  - ERC20 token support
```

### 9.2 Blockchain Workers

#### 9.2.1 Block Scanner Workers
```python
# Scan new blocks for transactions
opencex-eth-blocks      # Ethereum block scanner
opencex-bnb-blocks      # BNB Chain block scanner
opencex-trx-blocks      # Tron block scanner
opencex-matic-blocks    # Polygon block scanner
opencex-btc             # Bitcoin worker (dedicated)

Tasks:
  - Monitor new blocks
  - Detect incoming transactions
  - Update last processed block
  - Trigger deposit processing
```

#### 9.2.2 Deposit Processor
```python
# Process detected deposits
opencex-deposits

Chains:
  - trx_deposits
  - bnb_deposits
  - eth_deposits
  - matic_deposits

Tasks:
  - Verify transaction confirmations
  - Credit user balances
  - Send notifications
  - Update transaction status
```

#### 9.2.3 Withdrawal Processor
```python
# Process withdrawal requests
opencex-payouts

Chains:
  - trx_payouts
  - eth_payouts
  - bnb_payouts
  - matic_payouts

Tasks:
  - Validate withdrawal requests
  - Check balances
  - Create blockchain transactions
  - Broadcast transactions
  - Track confirmations
```

#### 9.2.4 Balance Checker
```python
# Verify hot wallet balances
opencex-balances

Chains:
  - trx_check_balances
  - bnb_check_balances
  - eth_check_balances
  - matic_check_balances

Tasks:
  - Query wallet balances
  - Compare with database
  - Alert on discrepancies
  - Monitor gas balances
```

#### 9.2.5 Accumulation Workers
```python
# Accumulate funds to main wallet
opencex-coin-accumulations      # Native coins
opencex-token-accumulations     # Tokens

Tasks:
  - Scan user wallets
  - Accumulate to hot wallet
  - Optimize gas usage
  - Track accumulation txs
```

#### 9.2.6 Gas Manager
```python
# Distribute gas to user wallets
opencex-gas

Chains:
  - trx_send_gas
  - bnb_send_gas
  - eth_send_gas
  - matic_send_gas

Tasks:
  - Monitor wallet gas levels
  - Send gas when needed
  - Optimize gas distribution
```

### 9.3 Wallet Management

#### 9.3.1 Hot Wallet (Keeper)
```python
Purpose:   Operational wallets for deposits/withdrawals
Storage:   Encrypted in database
Type:      HD (Hierarchical Deterministic)
Security:  AES encryption with CRYPTO_KEY
```

#### 9.3.2 Cold Wallet
```python
Purpose:   Long-term storage
Storage:   Offline/Hardware wallets
Config:    SAFE_ADDR environment variables
Usage:     Manual accumulation target
```

#### 9.3.3 Gas Keeper (EVM)
```python
Purpose:   Provide gas for EVM transactions
Coins:     ETH, BNB, TRX, MATIC
Function:  Automatically fund user wallets for token transfers
```

#### 9.3.4 User Wallets
```python
Generation: HD wallet per user per currency
Address:    Unique deposit address per user
Type:       Watch-only addresses
Private Keys: Stored encrypted
```

### 9.4 Transaction Flow

#### 9.4.1 Deposit Flow
```
1. User sends crypto to deposit address
2. Block scanner detects transaction
3. Wait for confirmations (configurable)
4. Deposit processor validates transaction
5. Credit user balance
6. Send notification
7. Trigger accumulation (if threshold met)
```

#### 9.4.2 Withdrawal Flow
```
1. User creates withdrawal request
2. Validation (balance, limits, KYC)
3. Admin approval (if required)
4. Withdrawal processor picks up request
5. Create blockchain transaction
6. Broadcast to network
7. Monitor confirmations
8. Update status
9. Send notification
```

#### 9.4.3 Accumulation Flow
```
1. Periodic scan of user wallets
2. Check balance threshold
3. Send gas (for EVM tokens)
4. Wait for gas confirmation
5. Transfer funds to hot wallet
6. Record accumulation transaction
7. Update wallet balance
```

---

## 10. Security & Compliance

### 10.1 Authentication & Authorization

#### 10.1.1 Multi-layer Authentication
```python
# JWT Tokens
Access Token:  Short-lived (15-60 minutes)
Refresh Token: Long-lived (7-30 days)
Storage:       HttpOnly cookies / localStorage

# 2FA (Two-Factor Authentication)
Method:        TOTP (Time-based One-Time Password)
Library:       django-otp
Support:       Google Authenticator, Authy compatible

# Session Security
Django sessions with Redis backend
CSRF protection enabled
Secure cookies (HTTPS only)
```

#### 10.1.2 Password Security
```python
Hashing:       PBKDF2 (Django default)
Validators:
  - Minimum length
  - Complexity requirements
  - Common password check
  - User attribute similarity check
```

### 10.2 KYC/KYT Integration

#### 10.2.1 KYC (Know Your Customer)
```python
Provider:      SumSub
Integration:   REST API + Webhooks
Features:
  - Identity verification
  - Document verification
  - Liveness check
  - AML screening
  
Configuration:
  - SUMSUB_SECRET_KEY
  - SUMSUB_APP_TOKEN
  - SUMSUM_CALLBACK_VALIDATION_SECRET
```

#### 10.2.2 KYT (Know Your Transaction)
```python
Provider:      Scorechain
Chains:
  - Bitcoin
  - Ethereum
  - Tron
  - BNB Chain

Features:
  - Transaction risk scoring
  - Address screening
  - AML compliance
  - Suspicious activity detection

Configuration:
  - SCORECHAIN_BITCOIN_TOKEN
  - SCORECHAIN_ETHEREUM_TOKEN
  - SCORECHAIN_TRON_TOKEN
  - SCORECHAIN_BNB_TOKEN
```

### 10.3 Data Encryption

#### 10.3.1 At Rest
```python
# Sensitive Data Encryption
Algorithm:     AES-256
Library:       cryptography
Keys:
  - CRYPTO_KEY (current)
  - CRYPTO_KEY_OLD (for migration)

Encrypted Fields:
  - Private keys
  - API secrets
  - Bot API keys
  - Sensitive user data
```

#### 10.3.2 In Transit
```python
# HTTPS/TLS
Proxy:         Caddy (automatic HTTPS)
Certificates:  Let's Encrypt (auto-renewal)
Protocols:     TLS 1.2, TLS 1.3

# WebSocket Security
WSS:           Secure WebSocket
Auth:          JWT token validation
```

### 10.4 API Security

#### 10.4.1 Rate Limiting
```python
# Django REST Framework throttling
Anon users:    100/hour
Auth users:    1000/hour
API keys:      5000/hour (configurable)

# Redis-based rate limiting
Sliding window algorithm
Per-user, per-endpoint limits
```

#### 10.4.2 CORS Configuration
```python
# django-cors-headers
ALLOWED_ORIGINS:
  - Frontend domain
  - Admin domain
  - Mobile apps

ALLOWED_METHODS:
  - GET, POST, PUT, PATCH, DELETE, OPTIONS

ALLOW_CREDENTIALS: True
```

#### 10.4.3 Input Validation
```python
# DRF Serializers
- Field validation
- Custom validators
- Type checking
- Length limits
- Format validation

# XSS Protection
- HTML sanitization
- Content Security Policy
- X-Content-Type-Options
- X-Frame-Options
```

### 10.5 Withdrawal Security

#### 10.5.1 Multi-level Approval
```python
Levels:
  1. User 2FA verification
  2. Email confirmation
  3. SMS verification (optional)
  4. Admin approval (large amounts)
  5. Manual review (suspicious activity)
```

#### 10.5.2 Withdrawal Limits
```python
Limits based on:
  - KYC level
  - Account age
  - Trading volume
  - Risk score

Types:
  - Daily limit
  - Monthly limit
  - Per-transaction limit
```

#### 10.5.3 Whitelist Addresses
```python
# Optional feature
- Pre-approved withdrawal addresses
- Whitelist verification period
- Remove from whitelist confirmation
```

### 10.6 Monitoring & Alerts

#### 10.6.1 Real-time Monitoring
```python
Tools:
  - Sentry (error tracking)
  - Prometheus (metrics)
  - Flower (Celery monitoring)

Alerts:
  - Failed transactions
  - Large withdrawals
  - Unusual activity
  - System errors
  - Security events
```

#### 10.6.2 Audit Logs
```python
Tracked Events:
  - User logins
  - API access
  - Order placement
  - Withdrawals
  - Settings changes
  - Admin actions

Storage:
  - core_accesslog
  - core_loginhistory
  - Model-specific history tables
```

### 10.7 Compliance Features

#### 10.7.1 AML (Anti-Money Laundering)
```python
- Transaction monitoring
- Suspicious activity reporting
- Large transaction alerts
- Pattern detection
- Risk-based approach
```

#### 10.7.2 Regulatory Reporting
```python
- Transaction history export
- User activity reports
- Volume reports
- Tax reporting support
- Audit trail
```

---

## 11. Deployment Guide

### 11.1 Prerequisites
```bash
# System Requirements
OS:              Linux (Ubuntu 20.04+ recommended)
RAM:             16GB minimum, 32GB+ recommended
CPU:             4+ cores recommended
Disk:            500GB+ SSD for blockchain data
Docker:          20.10+
Docker Compose:  1.29+

# Network
- Public IP address
- Domain name
- Open ports: 80, 443
```

### 11.2 Initial Setup

#### 11.2.1 Clone Repository
```bash
git clone https://github.com/Polygant/OpenCEX.git
cd OpenCEX
```

#### 11.2.2 Environment Configuration
```bash
# Copy environment template
cd backend
cp .env.template .env

# Edit configuration
nano .env
```

#### 11.2.3 Key Configuration
```bash
# Required Settings
PROJECT_NAME="YourExchange"
DOMAIN="exchange.com"
INSTANCE_NAME="production"

# Database
DB_USER="opencex"
DB_NAME="opencex"
DB_PASS="strong_password"
DB_HOST="postgresql"
DB_PORT="5432"

# Redis
REDIS_HOST="redis"
REDIS_PORT="6379"

# RabbitMQ
AMQP_USER="opencex"
AMQP_PASS="strong_password"
AMQP_HOST="rabbitmq"

# Security
CRYPTO_KEY="generate_random_32_byte_key"
ADMIN_MASTERPASS="strong_admin_password"

# Blockchain Nodes
BTC_NODE_HOST="bitcoind"
BTC_NODE_PORT="8332"
BTC_NODE_USER="rpcuser"
BTC_NODE_PASS="rpcpassword"

INFURA_API_KEY="your_infura_key"
TRONGRID_API_KEY="your_trongrid_key"

# External Services
SUMSUB_APP_TOKEN="your_sumsub_token"
TWILIO_ACCOUNT_SID="your_twilio_sid"
TELEGRAM_BOT_TOKEN="your_telegram_token"

# Cold Wallets
BTC_SAFE_ADDR="your_btc_cold_wallet"
ETH_SAFE_ADDR="your_eth_cold_wallet"
```

### 11.3 Build Images

#### 11.3.1 Backend
```bash
cd backend
docker build -t opencex:latest .
```

#### 11.3.2 Frontend
```bash
cd frontend
npm install
npm run build
docker build -t frontend:latest .
```

#### 11.3.3 Admin
```bash
cd admin
npm install
npm run build
docker build -t admin:latest .
```

#### 11.3.4 Nuxt
```bash
cd nuxt
npm install
npm run build
docker build -t nuxt:latest .
```

### 11.4 Database Setup

#### 11.4.1 Create Network
```bash
docker network create caddy
```

#### 11.4.2 Start Infrastructure
```bash
docker-compose up -d postgresql redis rabbitmq bitcoind
```

#### 11.4.3 Run Migrations
```bash
docker-compose run opencex python manage.py migrate
```

#### 11.4.4 Create Superuser
```bash
docker-compose run opencex python manage.py createsuperuser
```

#### 11.4.5 Collect Static Files
```bash
docker-compose run opencex python manage.py collectstatic --noinput
```

### 11.5 Start Services

#### 11.5.1 Full Stack
```bash
docker-compose up -d
```

#### 11.5.2 Verify Services
```bash
docker-compose ps
docker-compose logs -f opencex
```

### 11.6 Post-Deployment

#### 11.6.1 Access Admin Panel
```
URL: https://yourdomain.com/admin/
Login with superuser credentials
```

#### 11.6.2 Initial Configuration
```
1. Configure trading pairs
2. Set up fees and limits
3. Enable cryptocurrencies
4. Configure withdrawal limits
5. Set up KYC levels
6. Configure email templates
7. Test deposit/withdrawal flow
```

### 11.7 SSL/TLS Configuration
```bash
# Caddy handles SSL automatically via Let's Encrypt
# Ensure domain DNS points to server IP
# Caddy will auto-request and renew certificates
```

### 11.8 Monitoring Setup

#### 11.8.1 RabbitMQ Management
```
URL: http://yourdomain.com:15672/
Default: guest/guest (change in production)
```

#### 11.8.2 Flower (Celery Monitoring)
```bash
# Optional: Add flower service to docker-compose
docker-compose run -d -p 5555:5555 \
  opencex celery -A exchange flower
```

### 11.9 Backup Strategy

#### 11.9.1 Database Backup
```bash
# Automated daily backup
docker exec postgresql pg_dump -U opencex opencex > backup_$(date +%Y%m%d).sql

# Restore
docker exec -i postgresql psql -U opencex opencex < backup.sql
```

#### 11.9.2 Volume Backup
```bash
# Backup volumes
tar -czf postgresql_data_backup.tar.gz postgresql_data/
tar -czf redis_data_backup.tar.gz redis_data/
```

### 11.10 Update Procedure

#### 11.10.1 Backend Update
```bash
# Pull latest code
git pull origin master

# Rebuild image
cd backend
docker build -t opencex:latest .

# Run migrations
docker-compose run opencex python manage.py migrate

# Restart services
docker-compose restart opencex opencex-wss opencex-cel
```

#### 11.10.2 Frontend Update
```bash
cd frontend
git pull
npm install
npm run build
docker build -t frontend:latest .
docker-compose restart frontend
```

---

## 12. API Documentation

### 12.1 API Overview
```
Base URL:        https://api.yourdomain.com/api/
Version:         v1
Format:          JSON
Authentication:  JWT Bearer Token
Documentation:   /api/schema/swagger-ui/
```

### 12.2 Authentication Endpoints

#### 12.2.1 Register
```http
POST /api/v1/auth/register/
Content-Type: application/json

Request:
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "password_confirm": "SecurePass123!",
  "username": "username"
}

Response (201):
{
  "user": {
    "id": 1,
    "email": "user@example.com",
    "username": "username"
  },
  "tokens": {
    "access": "eyJ0eXAiOiJKV1QiLCJh...",
    "refresh": "eyJ0eXAiOiJKV1QiLCJh..."
  }
}
```

#### 12.2.2 Login
```http
POST /api/v1/auth/login/
Content-Type: application/json

Request:
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}

Response (200):
{
  "access": "eyJ0eXAiOiJKV1QiLCJh...",
  "refresh": "eyJ0eXAiOiJKV1QiLCJh...",
  "user": {
    "id": 1,
    "email": "user@example.com"
  }
}
```

#### 12.2.3 Refresh Token
```http
POST /api/v1/auth/token/refresh/
Content-Type: application/json

Request:
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJh..."
}

Response (200):
{
  "access": "eyJ0eXAiOiJKV1QiLCJh..."
}
```

### 12.3 Trading Endpoints

#### 12.3.1 Get Trading Pairs
```http
GET /api/v1/pairs/
Authorization: Bearer {access_token}

Response (200):
[
  {
    "id": 1,
    "base": "BTC",
    "quote": "USDT",
    "price": "45000.00",
    "change_24h": "+2.5%",
    "volume_24h": "1250000.00"
  }
]
```

#### 12.3.2 Create Order
```http
POST /api/v1/orders/
Authorization: Bearer {access_token}
Content-Type: application/json

Request:
{
  "pair": "BTC-USDT",
  "side": "buy",
  "type": "limit",
  "quantity": "0.5",
  "price": "45000.00"
}

Response (201):
{
  "id": 12345,
  "pair": "BTC-USDT",
  "side": "buy",
  "type": "limit",
  "quantity": "0.5",
  "price": "45000.00",
  "status": "open",
  "created_at": "2024-01-01T12:00:00Z"
}
```

#### 12.3.3 Get Order Book
```http
GET /api/v1/orderbook/BTC-USDT/
Authorization: Bearer {access_token}

Response (200):
{
  "bids": [
    ["44990.00", "0.5"],
    ["44980.00", "1.2"]
  ],
  "asks": [
    ["45010.00", "0.8"],
    ["45020.00", "1.5"]
  ]
}
```

### 12.4 Wallet Endpoints

#### 12.4.1 Get Wallets
```http
GET /api/v1/wallets/
Authorization: Bearer {access_token}

Response (200):
[
  {
    "currency": "BTC",
    "balance": "1.5",
    "available": "1.2",
    "locked": "0.3"
  },
  {
    "currency": "ETH",
    "balance": "10.0",
    "available": "10.0",
    "locked": "0.0"
  }
]
```

#### 12.4.2 Get Deposit Address
```http
GET /api/v1/wallets/BTC/address/
Authorization: Bearer {access_token}

Response (200):
{
  "currency": "BTC",
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "tag": null
}
```

#### 12.4.3 Create Withdrawal
```http
POST /api/v1/withdrawals/
Authorization: Bearer {access_token}
Content-Type: application/json

Request:
{
  "currency": "BTC",
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "amount": "0.5",
  "otp_code": "123456"
}

Response (201):
{
  "id": 789,
  "currency": "BTC",
  "amount": "0.5",
  "fee": "0.0005",
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "status": "pending",
  "created_at": "2024-01-01T12:00:00Z"
}
```

### 12.5 WebSocket API

#### 12.5.1 Connection
```javascript
const ws = new WebSocket('wss://api.yourdomain.com/ws/orderbook/BTC-USDT/');

ws.onopen = () => {
  console.log('Connected');
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Order book update:', data);
};
```

#### 12.5.2 Message Format
```json
{
  "type": "orderbook_update",
  "pair": "BTC-USDT",
  "bids": [["44990.00", "0.5"]],
  "asks": [["45010.00", "0.8"]],
  "timestamp": "2024-01-01T12:00:00Z"
}
```

### 12.6 Public API (No Auth Required)

#### 12.6.1 Ticker
```http
GET /api/public/v1/ticker/BTC-USDT/

Response (200):
{
  "pair": "BTC-USDT",
  "last": "45000.00",
  "high_24h": "46000.00",
  "low_24h": "44000.00",
  "volume_24h": "1250000.00",
  "change_24h": "+2.5%"
}
```

#### 12.6.2 Trades
```http
GET /api/public/v1/trades/BTC-USDT/

Response (200):
[
  {
    "price": "45000.00",
    "quantity": "0.5",
    "side": "buy",
    "timestamp": "2024-01-01T12:00:00Z"
  }
]
```

---

## 13. Development Workflow

### 13.1 Local Development Setup

#### 13.1.1 Backend Development
```bash
# Create virtual environment
cd backend
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Copy environment
cp .env.template .env

# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Run development server
python manage.py runserver
```

#### 13.1.2 Frontend Development
```bash
cd frontend
npm install
npm run dev  # Starts Vite dev server on http://localhost:5173
```

#### 13.1.3 Admin Development
```bash
cd admin
npm install
npm run dev  # Starts on http://localhost:5174
```

#### 13.1.4 Nuxt Development
```bash
cd nuxt
npm install
npm run dev  # Starts on http://localhost:3000
```

### 13.2 Code Quality

#### 13.2.1 Backend Linting
```bash
cd backend
flake8 .
```

#### 13.2.2 Frontend Linting
```bash
cd frontend
npm run lint
npm run format  # Prettier
```

#### 13.2.3 Type Checking
```bash
# Frontend
cd frontend
npm run type-check  # or vue-tsc

# Admin
cd admin
npm run build  # Includes type checking
```

### 13.3 Testing

#### 13.3.1 Backend Tests
```bash
cd backend
pytest
pytest --cov  # With coverage
behave        # BDD tests
```

#### 13.3.2 Frontend Tests
```bash
cd frontend
npm run test
```

### 13.4 Git Workflow
```bash
# Feature branch
git checkout -b feature/new-feature

# Commit changes
git add .
git commit -m "feat: add new feature"

# Push to remote
git push origin feature/new-feature

# Create pull request on GitHub
```

### 13.5 Database Migrations
```bash
# Create migration
python manage.py makemigrations

# View SQL
python manage.py sqlmigrate core 0001

# Apply migrations
python manage.py migrate

# Rollback
python manage.py migrate core 0001
```

---

## 14. Monitoring & Logging

### 14.1 Application Logs
```bash
# View container logs
docker-compose logs -f opencex
docker-compose logs -f opencex-cel
docker-compose logs -f opencex-deposits

# Filter logs
docker-compose logs opencex | grep ERROR
```

### 14.2 Celery Monitoring

#### 14.2.1 Flower
```bash
# Access Flower UI
http://localhost:5555

# Monitor:
- Active workers
- Task success/failure rates
- Task execution times
- Queue lengths
```

### 14.3 Database Monitoring
```sql
-- Active connections
SELECT count(*) FROM pg_stat_activity;

-- Slow queries
SELECT pid, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY duration DESC;

-- Table sizes
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename))
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### 14.4 RabbitMQ Monitoring
```
Management UI: http://localhost:15672

Monitor:
- Queue lengths
- Message rates
- Consumer counts
- Memory usage
```

### 14.5 Sentry Integration
```python
# Configured in backend/exchange/settings/sentry.py
SENTRY_DSN = env('SENTRY_DSN')

Features:
- Real-time error tracking
- Performance monitoring
- Release tracking
- User feedback
```

---

## 15. Troubleshooting

### 15.1 Common Issues

#### 15.1.1 Database Connection Error
```bash
Problem: Cannot connect to PostgreSQL

Solution:
1. Check PostgreSQL is running:
   docker-compose ps postgresql

2. Check credentials in .env file

3. Restart PostgreSQL:
   docker-compose restart postgresql
```

#### 15.1.2 Celery Worker Not Processing
```bash
Problem: Tasks stuck in queue

Solution:
1. Check worker status:
   docker-compose ps | grep opencex-

2. View worker logs:
   docker-compose logs -f opencex-cel

3. Restart worker:
   docker-compose restart opencex-cel
```

#### 15.1.3 WebSocket Connection Failed
```bash
Problem: Cannot connect to WebSocket

Solution:
1. Check Daphne is running:
   docker-compose ps opencex-wss

2. Verify Redis connection:
   docker exec redis redis-cli ping

3. Check firewall/proxy settings
```

#### 15.1.4 Blockchain Node Sync
```bash
Problem: Bitcoin node not synced

Solution:
1. Check sync status:
   docker exec bitcoind bitcoin-cli getblockchaininfo

2. Wait for full sync (may take days)

3. Use external node (configure BTC_NODE_HOST)
```

### 15.2 Performance Tuning

#### 15.2.1 Database Optimization
```sql
-- Create indexes
CREATE INDEX CONCURRENTLY idx_custom ON table_name(column);

-- Vacuum database
VACUUM ANALYZE;

-- Update statistics
ANALYZE;
```

#### 15.2.2 Redis Memory
```bash
# Check memory usage
docker exec redis redis-cli info memory

# Set max memory
docker exec redis redis-cli config set maxmemory 2gb
docker exec redis redis-cli config set maxmemory-policy allkeys-lru
```

#### 15.2.3 Celery Workers
```bash
# Scale workers
docker-compose up -d --scale opencex-cel=3

# Adjust concurrency
command: celery -A exchange worker -c 4  # 4 concurrent tasks
```

### 15.3 Security Checklist
```
□ Change default passwords
□ Set strong CRYPTO_KEY
□ Enable HTTPS
□ Configure firewall
□ Enable 2FA for admin
□ Regular backups
□ Monitor logs for suspicious activity
□ Keep dependencies updated
□ Configure rate limiting
□ Set up intrusion detection
```

### 15.4 Support Resources
```
Documentation: https://polygant.notion.site/OpenCEX-Help-Center
GitHub Issues:  https://github.com/Polygant/OpenCEX/issues
Telegram:      https://t.me/opencex
Email:         hello@polygant.net
```

---

## Phụ lục: Tổng kết

### Các thành phần chính:
1. **Backend**: Django 3.2.23 + DRF + Celery + Channels
2. **Frontend**: Vue 3 + Vite + TypeScript
3. **Admin**: Vue 3 + Vuetify + Pinia
4. **Nuxt**: Nuxt.js 2 (SSR cho SEO)
5. **Database**: PostgreSQL 17
6. **Cache**: Redis
7. **Queue**: RabbitMQ
8. **Proxy**: Caddy
9. **Blockchain**: BTC, ETH, BNB, TRX, MATIC

### Đặc điểm nổi bật:
- ✅ Kiến trúc microservices với 23+ containers
- ✅ 12+ Celery workers chuyên biệt cho từng blockchain
- ✅ WebSocket real-time cho trading
- ✅ KYC/KYT integration đầy đủ
- ✅ Multi-layer security
- ✅ Docker compose deployment
- ✅ Auto SSL với Caddy
- ✅ Scalable architecture

### Liên hệ:
- **Developer**: Polygant (https://polygant.net)
- **Email**: hello@polygant.net
- **License**: Apache 2.0

---

**Tài liệu này được tạo dựa trên phân tích toàn bộ source code của dự án OpenCEX.**
**Cập nhật lần cuối: 2024**
