# 🚀 Quick Start Guide

## Prerequisites Installation

### 1. Install Node.js 20+
```bash
# Download from https://nodejs.org/
# Or use nvm:
nvm install 20
nvm use 20
```

### 2. Install Docker
- **Windows**: [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)
- **Mac**: [Docker Desktop for Mac](https://docs.docker.com/desktop/install/mac-install/)
- **Linux**: [Docker Engine](https://docs.docker.com/engine/install/)

### 3. Verify Installation
```bash
node --version  # Should show v20.x.x
npm --version   # Should show 10.x.x
docker --version # Should show Docker version 24.x.x
```

---

## Project Setup

### Step 1: Install Dependencies
```bash
npm install
```

### Step 2: Environment Configuration
```bash
# Copy environment template
cp .env.example .env

# Edit .env if needed (defaults should work for local development)
```

### Step 3: Start Infrastructure (PostgreSQL + Redis)

#### Option A: Using Docker (Recommended)
```bash
# Start containers
npm run docker:up

# Verify containers are running
docker ps

# Expected output:
# - order_engine_postgres (port 5432)
# - order_engine_redis (port 6379)
```

#### Option B: Using WSL (Windows)
```bash
# From PowerShell
wsl

# Inside WSL
cd /mnt/c/Users/YourUsername/Desktop/VS_Code\ folders/Eterna/Backend_T2
docker compose up -d
```

#### Option C: Manual Installation
- Install PostgreSQL 16: https://www.postgresql.org/download/
- Install Redis 7: https://redis.io/download
- Update DATABASE_URL and REDIS_URL in .env

### Step 4: Database Migration
```bash
# Generate Prisma Client
npm run prisma:generate

# Run migrations (creates tables)
npm run prisma:migrate

# Optional: Open Prisma Studio to view database
npm run prisma:studio
```

---

## Running the Application

### Development Mode (Hot Reload)
```bash
npm run dev

# Server starts on http://localhost:3000
```

### Production Build
```bash
# Build TypeScript
npm run build

# Start production server
npm start
```

---

## Testing

### Run All Tests
```bash
npm test
```

### Run with Coverage
```bash
npm run test:coverage

# Coverage report will be in coverage/lcov-report/index.html
```

### Run Specific Test Suites
```bash
# Unit tests only
npm run test:unit

# Integration tests only
npm run test:integration

# Watch mode (re-run on file changes)
npm run test:watch
```

### Load Testing
```bash
# Start server first
npm run dev

# In another terminal, run load test
npm run load:test
```

---

## API Usage

### Using Postman
1. Import `postman/collection.json` into Postman
2. Set environment variable `baseUrl` to `http://localhost:3000`
3. Run requests or use "Run Collection" for automated tests

### Using cURL

#### Submit Order
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

#### Get Statistics
```bash
curl http://localhost:3000/api/orders/stats
```

#### Health Check
```bash
curl http://localhost:3000/health
```

### WebSocket Connection (Example using wscat)
```bash
# Install wscat
npm install -g wscat

# First, submit an order to get orderId
# Then connect to WebSocket (connection auto-upgrades after POST)

# For testing, you can use JavaScript:
const ws = new WebSocket('ws://localhost:3000/api/orders/execute');
ws.onmessage = (event) => console.log('Status Update:', JSON.parse(event.data));
```

---

## Troubleshooting

### Port Already in Use
```bash
# Find process using port 3000
# Windows
netstat -ano | findstr :3000

# Mac/Linux
lsof -i :3000

# Kill the process or change PORT in .env
```

### Database Connection Issues
```bash
# Check if PostgreSQL container is running
docker ps

# View container logs
docker compose logs postgres

# Restart containers
docker compose restart
```

### Redis Connection Issues
```bash
# Check Redis container
docker compose logs redis

# Test Redis connection
docker exec -it order_engine_redis redis-cli ping
# Should return: PONG
```

### Prisma Client Not Generated
```bash
# Regenerate Prisma Client
npm run prisma:generate

# If schema changed, run migration
npm run prisma:migrate
```

### TypeScript Errors
```bash
# Clean build artifacts
npm run clean

# Rebuild
npm run build
```

---

## Development Workflow

### 1. Start Development Environment
```bash
# Terminal 1: Start infrastructure
npm run docker:up

# Terminal 2: Start dev server
npm run dev

# Terminal 3: Run tests in watch mode
npm run test:watch
```

### 2. Making Changes
```bash
# Format code
npm run format

# Lint code
npm run lint

# Run tests
npm test
```

### 3. Before Committing
```bash
# Run all checks
npm run lint && npm run format && npm run test:coverage
```

---

## Docker Commands

### View Logs
```bash
# All containers
docker compose logs -f

# Specific container
docker compose logs -f postgres
docker compose logs -f redis
```

### Stop Services
```bash
# Stop containers (keeps data)
docker compose stop

# Stop and remove containers (keeps data in volumes)
docker compose down

# Remove everything including volumes (⚠️ deletes data)
docker compose down -v
```

### Restart Services
```bash
docker compose restart
```

---

## Monitoring

### Check Queue Status
```bash
curl http://localhost:3000/api/orders/stats | jq
```

### View Database Records
```bash
# Open Prisma Studio
npm run prisma:studio

# Or connect directly
docker exec -it order_engine_postgres psql -U postgres -d order_engine
\dt  # List tables
SELECT * FROM orders LIMIT 10;
```

### View Redis Data
```bash
docker exec -it order_engine_redis redis-cli

# View all keys
KEYS *

# View active orders
HGETALL active:order:ord_123456

# Monitor real-time commands
MONITOR
```

---

## Production Deployment

### Build Docker Image
```bash
docker build -t order-execution-engine .
```

### Run Container
```bash
docker run -d \
  -p 3000:3000 \
  -e DATABASE_URL="postgresql://..." \
  -e REDIS_URL="redis://..." \
  --name order-engine \
  order-execution-engine
```

### Deploy to Cloud Platforms

#### Render.com
1. Push code to GitHub
2. Create new Web Service on Render
3. Connect GitHub repository
4. Add PostgreSQL database (free tier available)
5. Add environment variables
6. Deploy

#### Railway.app
1. Create new project from GitHub
2. Add PostgreSQL plugin
3. Add Redis plugin
4. Deploy automatically

---

## Next Steps

- ✅ Read [README.md](README.md) for architecture and design decisions
- ✅ Import Postman collection for API testing
- ✅ Run load tests to see concurrent processing
- ✅ Explore code structure in `src/` directory
- ✅ Check out tests in `src/tests/` for examples

---

## Support

If you encounter issues:
1. Check this guide's Troubleshooting section
2. Review logs: `npm run docker:logs`
3. Verify environment variables in `.env`
4. Ensure all prerequisites are installed

**Happy coding! 🚀**
