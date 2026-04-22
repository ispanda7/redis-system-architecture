# 🚀 คู่มือสร้างระบบ Redis ครบทุก Use Case ด้วย Cursor AI

คัดลอกคำสั่งเหล่านี้ไปวางใน Cursor AI Chat (Cmd+L หรือ Ctrl+L) ทีละส่วน:

---

## 📋 **Part 1: สร้างโครงสร้างโปรเจกต์**

```
สร้างโปรเจกต์ monorepo ชื่อ "redis-complete-system" ที่มีโครงสร้างดังนี้:
- docker-compose.yml
- backend/ (NestJS)
- frontend/ (React + TypeScript + Vite)
- README.md

โดยให้สร้าง folder structure แบบครบถ้วนพร้อม package.json สำหรับแต่ละส่วน
```

---

## 🐳 **Part 2: สร้าง Docker Compose**

```
สร้างไฟล์ docker-compose.yml ที่มี services ดังนี้:
1. redis: ใช้ image redis:7-alpine, expose port 6379, มี volume
2. redis-commander: สำหรับดูข้อมูล Redis ผ่าน UI, port 8081
3. postgres: ใช้ image postgres:15-alpine, user/password: postgres/postgres, database: redis_demo, port 5432
4. backend: build จาก ./backend, port 3000, depends_on redis และ postgres
5. frontend: build จาก ./frontend, port 5173, depends_on backend

เพิ่ม networks และ volumes ที่เหมาะสม
```

---

##  **Part 3: สร้าง Database Schema (PostgreSQL)**

```
ใน folder backend/prisma สร้างไฟล์ schema.prisma ที่มี models:
1. User: id, email, name, createdAt
2. Product: id, name, price, stock, category
3. Order: id, userId, totalAmount, status, createdAt
4. OrderItem: id, orderId, productId, quantity, price
5. PageView: id, pageName, userId, viewedAt
6. Activity: id, userId, action, timestamp

สร้าง seed.ts สำหรับสร้างข้อมูลตัวอย่างอย่างน้อย 10 users, 20 products
```

---

## 🎯 **Part 4: สร้าง Redis Service (NestJS)**

```
ใน backend/src สร้างไฟล์ redis/redis.service.ts ที่มี methods ครบทุก use case:

1. STRING USE CASES:
   - setSession(userId, data, ttl)
   - getSession(userId)
   - cacheData(key, data, ttl)
   - getCachedData(key)
   - acquireLock(lockName, ttl)
   - releaseLock(lockName)
   - incrementCounter(key)
   - getCounter(key)
   - checkRateLimit(key, maxRequests, windowSeconds)
   - generateGlobalId(sequenceName)

2. HASH USE CASES:
   - addToCart(userId, productId, quantity)
   - getCart(userId)
   - updateCartItem(userId, productId, quantity)
   - removeFromCart(userId, productId)
   - clearCart(userId)

3. BITMAP USE CASES:
   - markUserActive(date, userId)
   - getUserRetention(startDate, endDate)
   - countActiveUsers(date)
   - calculateRetentionRate(days)

4. LIST USE CASES:
   - addToQueue(queueName, data)
   - processQueue(queueName)
   - getQueueLength(queueName)
   - peekQueue(queueName, count)

5. SORTED SET USE CASES:
   - addToLeaderboard(category, userId, score)
   - getLeaderboard(category, start, end)
   - getUserRank(category, userId)
   - incrementScore(category, userId, points)

6. SET USE CASES (เพิ่มเติม):
   - addTag(itemId, tag)
   - getTags(itemId)
   - findCommonTags(itemId1, itemId2)

สร้าง redis.module.ts ที่ export RedisService
```

---

## 🔧 **Part 5: สร้าง NestJS Modules และ Controllers**

```
สร้าง NestJS modules และ controllers ดังนี้:

1. Users Module:
   - users.controller.ts: CRUD operations
   - ใช้ Redis cache สำหรับ GET /users/:id
   
2. Products Module:
   - products.controller.ts: CRUD operations
   - ใช้ Redis cache สำหรับ GET /products
   - ใช้ Redis Hash สำหรับ shopping cart
   
3. Orders Module:
   - orders.controller.ts: create order, get orders
   - ใช้ Redis Counter สำหรับ order ID
   - ใช้ Redis List สำหรับ order processing queue
   
4. Analytics Module:
   - analytics.controller.ts:
     * POST /analytics/pageview - track page view
     * GET /analytics/retention - get user retention
     * GET /analytics/popular-pages - get popular pages
   
5. Leaderboard Module:
   - leaderboard.controller.ts:
     * POST /leaderboard/score - update score
     * GET /leaderboard/:category - get rankings
     * GET /leaderboard/:category/user/:userId - get user rank
   
6. Rate Limiter Module:
   - rate-limiter.controller.ts:
     * GET /api/* - ใช้ rate limiter guard
   
7. Session Module:
   - session.controller.ts:
     * POST /session/login
     * GET /session/me
     * DELETE /session/logout

สร้าง DTOs สำหรับทุก endpoint
```

---

## 🛡️ **Part 6: สร้าง Guards และ Interceptors**

```
สร้าง files ดังนี้:

1. backend/src/guards/rate-limiter.guard.ts:
   - ใช้ Redis checkRateLimit
   - จำกัด 10 requests ต่อนาทีต่อ IP
   
2. backend/src/guards/session.guard.ts:
   - ตรวจสอบ session จาก Redis
   - Inject user info จาก session
   
3. backend/src/interceptors/cache.interceptor.ts:
   - cache GET requests
   - ตั้ง TTL 60 วินาที
   
4. backend/src/middleware/logging.middleware.ts:
   - log ทุก request
   - track response time
```

---

## 📡 **Part 7: สร้าง Frontend React Components**

```
สร้าง frontend/src/components ดังนี้:

1. UserManagement.tsx:
   - แสดงรายการ users
   - ฟอร์มเพิ่ม/แก้ไข user
   - แสดง cache hit/miss status
   
2. ProductList.tsx:
   - แสดง products จาก cache
   - ปุ่ม add to cart (ใช้ Redis Hash)
   - แสดง stock แบบ real-time
   
3. ShoppingCart.tsx:
   - แสดงตะกร้าสินค้าจาก Redis
   - ปรับ quantity
   - checkout (สร้าง order)
   
4. Leaderboard.tsx:
   - แสดงอันดับแบบ real-time
   - กรองตาม category
   - แสดง rank ของ user ปัจจุบัน
   
5. Analytics.tsx:
   - แสดงกราฟ user retention (Bitmap)
   - แสดง page views
   - แสดง active users
   
6. RateLimitDemo.tsx:
   - ปุ่มกดทดสอบ rate limiting
   - แสดงจำนวน request ที่เหลือ
   - แสดง countdown
   
7. MessageQueue.tsx:
   - แสดง queue length
   - ปุ่ม add to queue
   - ปุ่ม process queue
   - แสดง logs
   
8. SessionDemo.tsx:
   - ฟอร์ม login
   - แสดง session info
   - ปุ่ม logout

สร้าง services/api.ts สำหรับเรียก backend APIs
```

---

## 🎨 **Part 8: สร้าง Frontend Pages และ Routing**

```
สร้าง frontend/src/pages:

1. HomePage.tsx:
   - dashboard แสดงภาพรวม
   - quick links ไปทุก use case
   
2. UsersPage.tsx:
   - ใช้ UserManagement component
   
3. ProductsPage.tsx:
   - ใช้ ProductList และ ShoppingCart
   
4. LeaderboardPage.tsx:
   - ใช้ Leaderboard component
   
5. AnalyticsPage.tsx:
   - ใช้ Analytics component
   
6. DemoPage.tsx:
   - รวม demo ทุก use case
   - RateLimitDemo
   - MessageQueue
   - SessionDemo

สร้าง App.tsx พร้อม routing ด้วย React Router
สร้าง navigation menu ที่ครบทุกหน้า
```

---

## 🔌 **Part 9: สร้าง Backend Main Configuration**

```
อัพเดทไฟล์ backend/src/main.ts:
- เปิด CORS สำหรับ frontend
- ใช้ global prefix 'api'
- ใช้ ValidationPipe
- เชื่อมต่อ Prisma
- เชื่อมต่อ Redis
- ตั้งค่า swagger documentation
- enable shutdown hooks

สร้าง backend/src/config:
- redis.config.ts
- database.config.ts
- app.config.ts

สร้าง backend/.env.example ที่มี:
REDIS_HOST, REDIS_PORT, POSTGRES_URL, JWT_SECRET, PORT
```

---

## 📝 **Part 10: สร้าง API Documentation**

```
สร้างไฟล์ backend/src/redis/redis.documentation.ts
อธิบายทุก method พร้อม example:

1. Session Management
   Example: login -> create session -> get session -> logout

2. Caching Strategy
   Example: get products -> cache -> invalidate on update

3. Shopping Cart Flow
   Example: add item -> update qty -> checkout -> clear cart

4. Rate Limiting Flow
   Example: check limit -> allow/deny -> reset window

5. Leaderboard Flow
   Example: add score -> get ranking -> get top 10

6. User Retention Flow
   Example: mark active -> calculate retention -> get stats

7. Message Queue Flow
   Example: enqueue -> process -> dequeue

สร้าง README.md ที่อธิบาย:
- วิธีติดตั้ง
- วิธีรัน
- Architecture diagram
- API endpoints ทั้งหมด
- Redis keys pattern ที่ใช้
```

---

## 🧪 **Part 11: สร้าง Test Files**

```
สร้าง backend/test/redis.e2e-spec.ts:
- ทดสอบทุก Redis use case
- ทดสอบ session flow
- ทดสอบ cart flow
- ทดสอบ leaderboard
- ทดสอบ rate limiting
- ทดสอบ caching

สร้าง frontend/src/test:
- ทดสอบ components
- ทดสอบ API calls
- ทดสอบ Redis interactions

สร้าง scripts/test-redis.sh สำหรับ test manual
```

---

## 🚀 **Part 12: สร้าง Scripts และ Dockerfiles**

```
สร้าง Dockerfile สำหรับ backend:
- Multi-stage build
- Node 18-alpine
- Install dependencies
- Build Prisma
- Expose port 3000

สร้าง Dockerfile สำหรับ frontend:
- Multi-stage build
- Node 18-alpine
- Build Vite
- Nginx serve
- Expose port 80

สร้าง docker-compose.prod.yml สำหรับ production

สร้าง scripts:
- start-dev.sh (รันทั้งหมดใน dev mode)
- start-prod.sh (รันด้วย Docker)
- reset-redis.sh (ล้าง Redis)
- seed-db.sh (seed ข้อมูล)
```

---

## 📊 **Part 13: สร้าง Monitoring Dashboard**

```
สร้าง backend/src/monitoring/monitoring.controller.ts:
- GET /monitoring/redis-info - ข้อมูล Redis
- GET /monitoring/memory - memory usage
- GET /monitoring/keys - จำนวน keys แต่ละ type
- GET /monitoring/slow-logs - slow queries
- GET /monitoring/stats - statistics

สร้าง frontend/src/components/MonitoringDashboard.tsx:
- แสดง Redis memory usage
- แสดงจำนวน keys
- แสดง connected clients
- แสดง ops/sec
- auto-refresh ทุก 5 วินาที
```

---

## 🎯 **Part 14: สร้าง Real-time Features**

```
ติดตั้ง @nestjs/websockets และ socket.io

สร้าง backend/src/events/events.gateway.ts:
- broadcast เมื่อมี order ใหม่
- broadcast เมื่อ leaderboard เปลี่ยน
- broadcast เมื่อ stock เปลี่ยน

สร้าง frontend/src/hooks/useWebSocket.ts:
- connect to WebSocket
- listen events
- update UI real-time

อัพเดท components:
- ShoppingCart: แสดง order ใหม่ real-time
- Leaderboard: อัพเดทอันดับ real-time
- ProductList: แสดง stock real-time
```

---

## 📚 **Part 15: สร้าง Documentation Site**

```
สร้าง docs/ folder:

1. docs/REDIS_USE_CASES.md:
   - อธิบายทุก use case พร้อม code example
   - อธิบาย data structures
   - best practices

2. docs/API_REFERENCE.md:
   - ทุก endpoints
   - request/response examples
   - error codes

3. docs/ARCHITECTURE.md:
   - system architecture
   - data flow diagrams
   - Redis keys naming convention

4. docs/PERFORMANCE.md:
   - benchmarking results
   - optimization tips
   - scaling strategies

สร้าง frontend/src/pages/Documentation.tsx:
- แสดง docs ทั้งหมดใน UI
- search functionality
- code syntax highlighting
```

---

## 🎉 **Part 16: สร้าง Demo Data และ Scenarios**

```
สร้าง backend/scripts/create-demo-data.ts:

1. สร้าง users 100 คน
2. สร้าง products 50 ชิ้น
3. สร้าง orders 200 orders
4. สร้าง page views 1000 records
5. สร้าง leaderboard data
6. สร้าง shopping carts
7. สร้าง sessions
8. สร้าง message queue items

สร้าง backend/scripts/demo-scenarios.ts:

Scenario 1: E-commerce Flow
- user login
- browse products
- add to cart
- checkout
- view order

Scenario 2: Gaming Leaderboard
- update scores
- get rankings
- track user progress

Scenario 3: Analytics Dashboard
- track page views
- calculate retention
- show active users

Scenario 4: Rate Limiting Demo
- make multiple requests
- show when limited
- show reset time

สร้าง frontend/src/pages/DemoScenarios.tsx:
- ปุ่มรันแต่ละ scenario
- แสดงผลลัพธ์ real-time
- step-by-step visualization
```

---

## 🏁 **Part 17: สร้าง Final Setup Instructions**

```
สร้างไฟล์ INSTALLATION.md:

## Prerequisites
- Docker & Docker Compose
- Node.js 18+
- Git

## Quick Start
1. git clone <repo>
2. cd redis-complete-system
3. cp backend/.env.example backend/.env
4. docker-compose up -d
5. npm run seed (ใน backend)
6. เปิด http://localhost:5173

## Development
- Backend: http://localhost:3000
- Frontend: http://localhost:5173
- Redis Commander: http://localhost:8081
- Swagger: http://localhost:3000/api/docs
- PostgreSQL: localhost:5432

## Testing
- npm run test
- npm run test:e2e

## Cleanup
- docker-compose down -v

สร้างไฟล์ TROUBLESHOOTING.md:
- Common issues และ solutions
- Redis connection problems
- Database migration issues
- Port conflicts
```

---

## 🎊 **คำสั่งสุดท้าย: Generate ทั้งหมด**

```
ตอนนี้ให้สร้างไฟล์ทั้งหมดตามที่ระบุไว้ข้างต้น:
1. เริ่มจาก docker-compose.yml
2. สร้าง backend NestJS structure ครบถ้วน
3. สร้าง frontend React structure ครบถ้วน
4. สร้าง Prisma schema และ seed data
5. สร้าง Redis service ครบทุก use case
6. สร้าง controllers, services, modules ทั้งหมด
7. สร้าง React components ครบทุกหน้า
8. สร้าง documentation
9. สร้าง test files
10. สร้าง scripts และ Dockerfiles

ทำให้สามารถรันคำสั่ง:
docker-compose up -d
และระบบทำงานได้ทันที

พร้อม demo data และ UI ที่สวยงาม ใช้งานง่าย
```

---

## 📌 **วิธีใช้งาน**

1. เปิด Cursor AI
2. กด `Cmd+L` (Mac) หรือ `Ctrl+L` (Windows)
3. คัดลอกคำสั่งแต่ละ Part ไปวางทีละส่วน
4. รอให้ Cursor สร้างไฟล์
5. เมื่อเสร็จทั้งหมด ให้รัน:

```bash
# สร้าง environment
cd backend
cp .env.example .env

# รันระบบ
docker-compose up -d

# Seed ข้อมูล
cd backend
npx prisma migrate dev
npx prisma db seed

# เปิด browser
# Frontend: http://localhost:5173
# Backend API: http://localhost:3000
# Redis Commander: http://localhost:8081
# Swagger Docs: http://localhost:3000/api/docs
```

ระบบนี้จะทำให้คุณเรียนรู้ Redis ครบทุก Use Case จากการลงมือทำจริง! 🚀
