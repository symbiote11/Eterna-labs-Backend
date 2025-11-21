# Postman Collection Testing Guide -


### Step 1: Import Collection

1. Open Postman
2. Click **Import** button (top left)
3. Choose **File** tab
4. Navigate to: `c:\Users\Invincible\Desktop\VS_Code folders\Eterna\Backend_T2\postman\collection.json`
5. Click **Import**

### Step 2: Set Base URL (Optional)

The collection uses `{{baseUrl}}` variable. To set it:

1. Click on **Collections** → **Order Execution Engine API**
2. Go to **Variables** tab
3. Set `baseUrl` to `http://localhost:3000`
4. Click **Save**

(Or just replace `{{baseUrl}}` with `http://localhost:3000` in each request)

### Step 3: Test Requests (in order)

#### 1. Health Check
- **Method**: GET
- **URL**: `http://localhost:3000/health`
- **Expected Response**:
  ```json
  {
    "status": "ok",
    "timestamp": "2025-11-08T14:00:00.000Z",
    "uptime": 123.456
  }
  ```

#### 2. Get API Info
- **Method**: GET
- **URL**: `http://localhost:3000/`
- **Expected Response**:
  ```json
  {
    "name": "Order Execution Engine",
    "version": "1.0.0",
    "endpoints": {
      "health": "/health",
      "executeOrder": "POST /api/orders/execute"
    }
  }
  ```

#### 3. Execute Market Order - SOL to USDC
- **Method**: POST
- **URL**: `http://localhost:3000/api/orders/execute`
- **Body**:
  ```json
  {
    "tokenIn": "SOL",
    "tokenOut": "USDC",
    "amount": 1.5,
    "slippage": 0.01
  }
  ```
- **Expected Response**:
  ```json
  {
    "orderId": "ord_1762613166643_l59weki",
    "status": "pending",
    "timestamp": "2025-11-08T14:00:00.000Z",
    "message": "Order created successfully...",
    "websocket": "ws://localhost:3000/api/orders/ord_xxx/stream"
  }
  ```

#### 4. Get Order Stats
- **Method**: GET
- **URL**: `http://localhost:3000/api/orders/stats`
- **Expected Response**:
  ```json
  {
    "queue": {
      "waiting": 0,
      "active": 0,
      "completed": 5,
      "failed": 0,
      "delayed": 0,
      "total": 5
    },
    "websocket": {
      "connections": 0
    },
    "timestamp": "2025-11-08T14:00:00.000Z"
  }
  ```

#### 5-8. More Order Variations
Test different combinations:
- USDC to SOL
- Large amounts
- Different slippage
- Invalid inputs (should return errors)

### Step 4: Run Collection (Automated)

1. Click on **Order Execution Engine API** collection
2. Click **Run** button
3. Select requests to run
4. Click **Run Order Execution Engine API**
5. Watch automated tests pass! 

The collection includes automated tests that check:
- ✅ Status codes
- ✅ Response structure
- ✅ Order ID format
- ✅ Status values

### WebSocket Testing in Postman

Postman supports WebSocket connections:

1. Create new **WebSocket Request**
2. URL: `ws://localhost:3000/api/orders/{orderId}/stream`
3. Replace `{orderId}` with an actual order ID
4. Click **Connect**
5. Watch real-time messages!

---

## Quick Test

If you don't have Postman, you can use `curl` commands:

```bash
# Health Check
curl http://localhost:3000/health

# Submit Order
curl -X POST http://localhost:3000/api/orders/execute ^
  -H "Content-Type: application/json" ^
  -d "{\"tokenIn\":\"SOL\",\"tokenOut\":\"USDC\",\"amount\":1.5,\"slippage\":0.01}"

# Get Stats
curl http://localhost:3000/api/orders/stats
```

Or use the test scripts:
```bash
cmd /c npx tsx scripts/test-api.ts
```
