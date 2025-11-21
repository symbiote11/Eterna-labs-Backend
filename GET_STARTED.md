Details with troubleshoot - 
---

## Steps

### 1. Setup & Run (First Time)

```bash
# Install dependencies
npm install

# Start infrastructure (PostgreSQL + Redis)
# Note: Requires Docker installed
docker compose up -d

# Run database migrations
npx prisma migrate dev

# Start development server
npm run dev
```


### 2. Verify Installation

```bash
# Health check
curl http://localhost:3000/health

# Expected response:
# {"status":"ok","timestamp":"...","uptime":...}
```

### 3. Test the System

#### Option A: Use Postman
1. Import `postman/collection.json`
2. Run "Execute Market Order - SOL to USDC"
3. Check WebSocket response (auto-upgrade after POST)

#### Option B: Use cURL
```bash
curl -X POST http://localhost:3000/api/orders/execute \
  -H "Content-Type: application/json" \
  -d '{
    "tokenIn": "SOL",
    "tokenOut": "USDC",
    "amount": 1.5,
    "slippage": 0.01
  }'
```

### 4. Run Tests

```bash
# Run all tests
npm test

# Run with coverage
npm run test:coverage

# Run load test (server must be running)
npm run load:test
```


## 📂 File Structure Overview

```
Backend_T2/
├── 📁 src/
│   ├── 📁 config/           → Env, DB, Redis setup
│   ├── 📁 controllers/      → HTTP handlers
│   ├── 📁 executors/        → Market order logic
│   ├── 📁 middleware/       → Validation & errors
│   ├── 📁 models/           → TypeScript types
│   ├── 📁 routes/           → API endpoints
│   ├── 📁 services/         → DEX router, queue, WebSocket
│   ├── 📁 utils/            → Helpers & constants
│   ├── 📁 workers/          → BullMQ job processor
│   ├── 📁 tests/            → Unit tests
│   ├── 📄 app.ts            → Fastify app
│   └── 📄 index.ts          → Server entry
├── 📁 prisma/               → Database schema & migrations
├── 📁 postman/              → API test collection
├── 📁 scripts/              → Load test script
├── 📄 Dockerfile            → Production container
├── 📄 docker-compose.yml    → Local infrastructure
├── 📄 README.md             → Full documentation
├── 📄 SETUP.md              → Quick start guide
└── 📄 package.json          → Dependencies & scripts
```

---

## 🎯 Key Features Explained

### 1. Order Lifecycle
```
Client POST → Validation → Queue → Worker → DEX Router → Swap → Confirmed
                ↓           ↓        ↓          ↓          ↓         ↓
            WebSocket   WebSocket WebSocket WebSocket WebSocket WebSocket
             "pending"  "routing" "building" "submitted" "confirmed"
```

### 2. DEX Routing Logic
```typescript
1. Fetch Raydium quote  → price: 99.5, fee: 0.3%
2. Fetch Meteora quote  → price: 99.8, fee: 0.2%
3. Compare effective prices
4. Select Meteora (better: 99.8 vs 99.5)
5. Log routing decision
6. Execute swap on Meteora
```

### 3. Queue Concurrency
```
┌─────────────┐
│   Request   │  ← 100 orders submitted
└──────┬──────┘
       ↓
┌──────────────┐
│  BullMQ      │  ← Queue (max 100/min)
│  Queue       │
└──────┬───────┘
       ↓
┌──────────────────────────────┐
│  10 Concurrent Workers       │  ← Processes 10 at a time
│  [W1] [W2] ... [W10]         │
└──────────────────────────────┘
```

### 4. Error Handling
```
Attempt 1: Fail → Wait 1s  → Retry
Attempt 2: Fail → Wait 2s  → Retry
Attempt 3: Fail → Wait 4s  → Mark as "failed"
                             Persist error to DB
                             Emit WebSocket update
```

---

## 🧪 Testing Overview

### Unit Tests (22 tests)
- DEX Router: Price comparison, delays, slippage
-  Market Order Executor: Validation, state machine, errors
-  Helper Functions: ID generation, backoff, formatting


### Load Testing
```bash
npm run load:test

# Tests:
# - 10 concurrent orders
# - 20 concurrent orders (queue limit test)
# - 100 orders/minute sustained (30s)
```

---

##  Performance Expectations

| Metric | Target | Notes |
|--------|--------|-------|
| API Response | <100ms | POST endpoint only |
| WebSocket Latency | <50ms | Per status update |
| Order Execution | 2-3s | Mock swap time |
| Concurrent Orders | 10 | Simultaneous processing |
| Throughput | 100/min | Sustained rate |

---


## 🔧 Troubleshooting

### Docker not starting?
```bash
# Check Docker is running
docker ps

# Start Docker Desktop (Windows/Mac)
# Or: sudo systemctl start docker (Linux)
```

### Port 3000 already in use?
```bash
# Change PORT in .env
PORT=3001

# Or kill process using port 3000
# Windows: netstat -ano | findstr :3000
# Mac/Linux: lsof -i :3000
```

### Database connection error?
```bash
# Ensure PostgreSQL container is running
docker compose ps

# Restart containers
docker compose restart

# Check logs
docker compose logs postgres
```

### Tests failing?
```bash
# Clean install
rm -rf node_modules package-lock.json
npm install

# Regenerate Prisma Client
npx prisma generate

# Run tests
npm test
```

---


## 🎓 Learning Outcomes

By building this project, I have learnt and implemented:

1. **Queue-Based Architecture** with BullMQ
2. **Real-Time Communication** with WebSocket
3. **Microservices Patterns** (separation of concerns)
4. **Event-Driven Design** (status updates via pub/sub)
5. **Factory Pattern** (extensible order types)
6. **Type-Safe Development** (TypeScript + Zod + Prisma)
7. **Production Best Practices** (logging, error handling, testing)

---

