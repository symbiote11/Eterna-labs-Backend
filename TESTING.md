# 🧪 Manual Testing Guide

## Test Results Summary

### ✅ Unit Tests Completed (44 tests)

**Helper Functions (17 tests)** - ✅ ALL PASSED
- generateOrderId: unique IDs, prefix validation, format
- generateMockTxHash: 64-char hex, uniqueness
- calculateBackoff: exponential calculation, max cap
- formatPrice: decimal formatting
- calculatePercentageDiff: percentage calculations
- sleep: async delay functionality

**DEX Router (12 tests)** - ✅ ALL PASSED
- getRaydiumQuote: price variance, network delay, scaling
- getMeteoraQuote: fee structure, variance range
- getBestQuote: parallel fetching, best price selection
- executeSwap: execution delay, tx hash, slippage

**Market Order Executor (15 tests)** - ✅ ALL PASSED
- validate: 8 validation edge cases
- execute: status updates, DEX routing, error handling

**Total: 44/44 tests passed ✅**

---

## Manual Testing Checklist (Without Docker)

Since Docker is not available in your environment, here's how to test the system:

### Option 1: Use External Database Services (Recommended for Testing)

#### Step 1: Setup Free Cloud Services

**PostgreSQL - ElephantSQL (Free Tier)**
1. Go to: https://www.elephantsql.com/
2. Sign up and create a free "Tiny Turtle" instance
3. Copy the connection URL
4. Update `.env`:
   ```env
   DATABASE_URL=postgresql://user:pass@server.db.elephantsql.com/dbname
   ```

**Redis - Upstash (Free Tier)**
1. Go to: https://upstash.com/
2. Sign up and create a free Redis database
3. Copy the connection URL
4. Update `.env`:
   ```env
   REDIS_URL=redis://user:pass@server.upstash.io:6379
   ```

#### Step 2: Run Migrations
```bash
npx prisma generate
npx prisma migrate dev
```

#### Step 3: Start Server
```bash
npm run dev
```

### Option 2: Install PostgreSQL & Redis Locally

**PostgreSQL 16:**
- Download: https://www.postgresql.org/download/windows/
- Install and create database: `order_engine`
- Update DATABASE_URL in `.env`

**Redis 7:**
- Download: https://github.com/microsoftarchive/redis/releases
- Or use Memurai: https://www.memurai.com/get-memurai
- Update REDIS_URL in `.env`

### Option 3: Use WSL with Docker (If Available)

From WSL terminal:
```bash
cd "/mnt/c/Users/Invincible/Desktop/VS_Code folders/Eterna/Backend_T2"
docker compose up -d
```

---

## Manual API Testing (Once Server is Running)

### Test 1: Health Check
```bash
curl http://localhost:3000/health
```

**Expected Response:**
```json
{
  "status": "ok",
  "timestamp": "2025-11-08T...",
  "uptime": 12.345
}
```

### Test 2: Submit Single Order
```bash
curl -X POST http://localhost:3000/api/orders/execute \
  -H "Content-Type: application/json" \
  -d "{\"tokenIn\":\"SOL\",\"tokenOut\":\"USDC\",\"amount\":1.5,\"slippage\":0.01}"
```

**Expected Response (201):**
```json
{
  "orderId": "ord_1699451234567_abc123",
  "status": "pending",
  "timestamp": "2025-11-08T10:30:00.000Z",
  "message": "Order created. WebSocket connection available for real-time updates."
}
```

### Test 3: Get Queue Stats
```bash
curl http://localhost:3000/api/orders/stats
```

**Expected Response:**
```json
{
  "queue": {
    "waiting": 0,
    "active": 0,
    "completed": 1,
    "failed": 0
  },
  "websocket": {
    "connections": 0
  }
}
```

### Test 4: Validation Error Test
```bash
curl -X POST http://localhost:3000/api/orders/execute \
  -H "Content-Type: application/json" \
  -d "{\"tokenIn\":\"SOL\",\"amount\":-1}"
```

**Expected Response (400):**
```json
{
  "error": "Bad Request",
  "message": "tokenOut: Token Out is required; amount: Amount must be positive"
}
```

---

## Postman Testing

### Import Collection
1. Open Postman
2. Import → Upload Files
3. Select `postman/collection.json`
4. Set environment variable: `baseUrl = http://localhost:3000`

### Run Automated Tests
1. Click "..." next to collection name
2. Select "Run collection"
3. Click "Run Order Execution Engine API"
4. View automated test results

**Expected Results:**
- ✅ Health Check: Status code 200
- ✅ Execute Order: Status code 201, has orderId
- ✅ Queue Stats: Has queue and websocket objects
- ✅ Invalid Orders: Status code 400, has error message

---

## Load Testing (Advanced)

### Prerequisites
- Server must be running
- Database and Redis connected

### Run Load Test
```bash
# In one terminal: start server
npm run dev

# In another terminal: run load test
npm run load:test
```

### Expected Output
```
🧪 ORDER EXECUTION ENGINE - LOAD TEST
=====================================

📋 TEST 1: 10 Concurrent Orders
------------------------------------------------------------
✅ Order 1/10 - 45ms
✅ Order 2/10 - 52ms
...
✅ Order 10/10 - 61ms

📊 LOAD TEST RESULTS
============================================================
Total Orders:       10
✅ Successful:      10
❌ Failed:          0
Success Rate:       100.0%
------------------------------------------------------------
Avg Response Time:  54ms
Min Response Time:  45ms
Max Response Time:  61ms
------------------------------------------------------------
Throughput:         120 orders/minute
============================================================
```

---

## WebSocket Testing

### Using JavaScript (Browser Console)
```javascript
// Submit order first via API or Postman to get orderId
const orderId = "ord_1699451234567_abc123";

// Connect to WebSocket (connection is auto-established after POST)
// For manual testing, you can connect directly:
const ws = new WebSocket('ws://localhost:3000/api/orders/execute');

ws.onopen = () => console.log('✅ Connected');

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('📡 Status Update:', data);
};

ws.onerror = (error) => console.error('❌ Error:', error);

ws.onclose = () => console.log('👋 Connection closed');
```

### Expected WebSocket Messages
```javascript
// 1. Connection established
{ event: "connection:established", orderId: "...", timestamp: "..." }

// 2. Pending
{ orderId: "...", status: "pending", timestamp: "..." }

// 3. Routing
{ 
  orderId: "...", 
  status: "routing", 
  timestamp: "...",
  data: { 
    selectedDex: "Meteora",
    reason: "Better price: 99.8 vs 99.5 (0.30% better)",
    quotes: [...]
  }
}

// 4. Building
{ orderId: "...", status: "building", timestamp: "..." }

// 5. Submitted
{ orderId: "...", status: "submitted", timestamp: "..." }

// 6. Confirmed
{ 
  orderId: "...", 
  status: "confirmed", 
  timestamp: "...",
  data: {
    txHash: "abc123...",
    executedPrice: 99.75,
    selectedDex: "Meteora"
  }
}
```

---

## Database Verification (If Using Prisma Studio)

```bash
# Open Prisma Studio
npx prisma studio

# Opens browser at http://localhost:5555
```

### Check Order Records
1. Click on "Order" model
2. View order records with:
   - orderId
   - status (should be "confirmed" or "failed")
   - txHash (mock transaction hash)
   - executedPrice
   - selectedDex
   - createdAt, updatedAt

---

## Troubleshooting Common Issues

### Server Won't Start
**Error:** Cannot connect to database
**Solution:** Check DATABASE_URL in `.env` and ensure database is accessible

**Error:** Cannot connect to Redis
**Solution:** Check REDIS_URL in `.env` and ensure Redis is running

**Error:** Port 3000 already in use
**Solution:** Change PORT in `.env` or kill process using port 3000

### Tests Failing
**Error:** Module not found
**Solution:** Run `npm install` and `npx prisma generate`

**Error:** Timeout errors
**Solution:** Increase Jest timeout in `jest.config.js`

### WebSocket Not Connecting
**Error:** Connection refused
**Solution:** Ensure server is running and check WebSocket URL

---

## Test Coverage Report

To generate HTML coverage report:

```bash
npm run test:coverage

# Open coverage report
# Windows:
start coverage/lcov-report/index.html

# Mac/Linux:
open coverage/lcov-report/index.html
```

---

## Next Steps After Manual Testing

1. ✅ Verify all unit tests pass (`npm test`)
2. ✅ Test API endpoints with Postman
3. ✅ Submit concurrent orders (5-10 simultaneously)
4. ✅ Verify WebSocket status updates
5. ✅ Check database for order records
6. ✅ Run load test script
7. ✅ Review logs for DEX routing decisions
8. 📹 Record demo video showing all above

---


## Summary

✅ **44 Unit Tests** - All passing
✅ **API Endpoints** - Ready for testing
✅ **Postman Collection** - 8 requests with automated tests
✅ **Load Test Script** - Performance testing ready
✅ **Documentation** - Complete guides available


For deployment, see `SETUP.md` for cloud service integration.
