# redis-system-architecture
ระบบนี้คือ Redis Playground ที่สมบูรณ์ สำหรับเรียนรู้และทดลองใช้งาน Redis ทุก Use Case ในโลกจริง!

# 📋 อธิบายระบบ Redis Complete System

## 🏗️ **ภาพรวมสถาปัตยกรรม (System Architecture)**

```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend (React)                     │
│                    http://localhost:5173                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │   Users  │ │ Products │ │   Cart   │ │Leaderboard│     │
│  └──────────┘ └────────── └────────── └──────────      │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP/REST + WebSocket
┌────────────────────────▼────────────────────────────────────┐
│                    Backend (NestJS)                          │
│                   http://localhost:3000                      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Redis Service (Core)                     │  │
│  │  ┌──────── ┌───────┐ ┌────────┐ ┌─────────────┐  │  │
│  │  │ String │ │ Hash  │ │  List  │ │ Sorted Set  │  │  │
│  │  └────────┘ └───────┘ └────────┘ └─────────────┘  │  │
│  │  ┌────────┐ ───────┐ ────────┐ ┌─────────────┐  │  │
│  │  │  Set   │ │Bitmap │ │  Stream│ │HyperLogLog  │  │  │
│  │  └────────┘ └───────┘ └────────┘ └─────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │    Users     │  │   Products   │  │    Orders    │    │
│  │   Module     │  │    Module    │  │    Module    │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  Analytics   │  │  Leaderboard │  │ Rate Limiter │    │
│  │   Module     │  │    Module    │  │    Module    │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────┬──────────────────┬──────────────────┬───────────┘
          │                  │                  │
┌─────────▼──────┐  ┌──────▼──────┐  ┌────────▼────────┐
│   PostgreSQL   │  │    Redis    │  │  Redis Commander│
│   (Port 5432)  │  │ (Port 6379) │  │   (Port 8081)   │
│                │  │             │  │                 │
│ • Users        │  │ • Sessions  │  │  Redis UI       │
│ • Products     │  │ • Cache     │  │  Monitor & Debug│
│ • Orders       │  │ • Queue     │  │                 │
│ • Activities   │  │ • Pub/Sub   │  │                 │
└────────────────┘  └─────────────┘  └─────────────────┘
```

---

## 🎯 **Redis Use Cases ทั้งหมดในระบบ**

### **1. STRING - Use Cases (6 รูปแบบ)**

#### **1.1 Session Management**
```
วัตถุประสงค์: เก็บ session ผู้ใช้ที่ login
Key Pattern: session:{userId}
TTL: 30 นาที

Flow:
1. User login → สร้าง session
2. เก็บ user data ใน Redis
3. ทุก request ตรวจสอบ session
4. Logout → ลบ session

ตัวอย่าง:
SET session:user123 '{"id":123,"email":"user@test.com"}' EX 1800
GET session:user123
DEL session:user123
```

#### **1.2 Caching**
```
วัตถุประสงค์: cache ข้อมูลจาก database
Key Pattern: cache:{entity}:{id}
TTL: 5-60 นาที

Flow:
1. รับ request ข้อมูล
2. ตรวจสอบ cache ใน Redis
3. ถ้ามี → return cache (cache hit)
4. ถ้าไม่มี → query DB → save cache → return

ตัวอย่าง:
GET cache:products:all
→ (miss) → SELECT * FROM products
→ SET cache:products:all "[...]" EX 300
```

#### **1.3 Distributed Lock**
```
วัตถุประสงค์: ควบคุม concurrent access
Key Pattern: lock:{resourceName}
TTL: 10-30 วินาที

Flow:
1. ต้องการ access resource
2. SET lock NX EX (สร้าง lock ถ้ายังไม่มี)
3. ทำงานกับ resource
4. DEL lock (ปล่อย lock)

ตัวอย่าง:
SET lock:order:123 unique_id NX EX 10
→ (ถ้าได้ lock) → process order
→ DEL lock:order:123
```

#### **1.4 Counter**
```
วัตถุประสงค์: นับจำนวนต่างๆ
Key Pattern: counter:{type}:{id}

Flow:
1. เกิด event
2. INCR counter
3. อ่านค่าเมื่อต้องการ

ตัวอย่าง:
INCR counter:page:home
INCR counter:product:456:views
GET counter:page:home
```

#### **1.5 Rate Limiter**
```
วัตถุประสงค์: จำกัดจำนวน request
Key Pattern: rate:{userId/IP}:{endpoint}
TTL: 1 นาที

Flow:
1. รับ request
2. INCR key
3. ถ้าเกิน limit → reject
4. SET EX ครั้งแรก

ตัวอย่าง:
INCR rate:user123:/api/data
→ ถ้า = 1 → EXPIRE rate:user123:/api/data 60
→ ถ้า > 10 → return 429 Too Many Requests
```

#### **1.6 Global ID Generator**
```
วัตถุประสงค์: สร้าง unique ID แบบ sequential
Key Pattern: id:{sequenceName}

Flow:
1. ต้องการ ID ใหม่
2. INCR sequence
3. ได้ ID แบบ unique, increasing

ตัวอย่าง:
INCR id:orders
→ ได้ 1001
INCR id:orders
→ ได้ 1002
```

---

### **2. HASH - Use Cases (Shopping Cart)**

```
วัตถุประสงค์: เก็บตะกร้าสินค้า
Key Pattern: cart:{userId}
Field: productId
Value: quantity

Flow:
1. User เลือกสินค้า
2. HSET cart userId productId quantity
3. อัพเดท quantity
4. Checkout → อ่านข้อมูล → ลบ cart

ตัวอย่าง:
HSET cart:user123 product456 2
HSET cart:user123 product789 1
HGETALL cart:user123
→ { "product456": "2", "product789": "1" }
HINCRBY cart:user123 product456 1
→ เพิ่ม quantity เป็น 3
```

---

### **3. BITMAP - Use Cases (User Retention)**

```
วัตถุประสงค์: ติดตามการใช้งานของผู้ใช้
Key Pattern: active:{date}
Bit Position: userId
Value: 0/1 (inactive/active)

Flow:
1. User เข้าใช้งาน
2. SETBIT active:2024-01-15 userId 1
3. นับจำนวน active users
4. คำนวณ retention rate

ตัวอย่าง:
SETBIT active:2024-01-15 123 1
SETBIT active:2024-01-16 123 1
BITCOUNT active:2024-01-15
→ นับจำนวน bit ที่ = 1
BITOP AND retention active:2024-01-15 active:2024-01-16
→ หา users ที่ active 2 วันติด
```

---

### **4. LIST - Use Cases (Message Queue)**

```
วัตถุประสงค์: ระบบ queue สำหรับประมวลผล
Key Pattern: queue:{queueName}

Flow:
1. มีงานใหม่ → LPUSH
2. Worker → RPOP
3. ประมวลผล
4. ทำซ้ำ

ตัวอย่าง:
LPUSH queue:orders '{"orderId":123}'
LPUSH queue:emails '{"to":"user@test.com"}'
RPOP queue:orders
→ ได้งานมาประมวลผล
LLEN queue:orders
→ ดูจำนวนงานคงค้าง
```

---

### **5. SORTED SET (ZSet) - Use Cases (Leaderboard)**

```
วัตถุประสงค์: ระบบอันดับ/คะแนน
Key Pattern: leaderboard:{category}
Member: userId
Score: points

Flow:
1. User ทำกิจกรรม → ได้คะแนน
2. ZADD leaderboard userId score
3. แสดงอันดับ
4. อัพเดทคะแนน

ตัวอย่าง:
ZADD leaderboard:game1 1500 user123
ZADD leaderboard:game1 2000 user456
ZREVRANGE leaderboard:game1 0 9 WITHSCORES
→ Top 10 พร้อมคะแนน
ZREVRANK leaderboard:game1 user123
→ อันดับของ user123
ZINCRBY leaderboard:game1 100 user123
→ เพิ่มคะแนน 100
```

---

### **6. SET - Use Cases (Tags & Unique Items)**

```
วัตถุประสงค์: เก็บ tags, หา common items
Key Pattern: tags:{itemId} หรือ following:{userId}

Flow:
1. เพิ่ม tag
2. หา tags ร่วมกัน
3. นับจำนวน unique

ตัวอย่าง:
SADD tags:post1 "redis" "database" "nosql"
SADD tags:post2 "redis" "cache" "performance"
SINTER tags:post1 tags:post2
→ ได้ "redis" (tags ร่วม)
SCARD tags:post1
→ จำนวน tags = 3
```

---

## 🔄 **Data Flow ในระบบ**

### **Flow 1: User Login & Session**
```
Frontend                Backend                 Redis              PostgreSQL
   │                       │                       │                     │
   │──POST /login────────>│                       │                     │
   │  {email, password}   │                       │                     │
   │                       │──SELECT user───────>│                     │
   │                       │<────────────────────│                     │
   │                       │                       │                     │
   │                       │──SET session───────>│                     │
   │                       │   EX 1800            │                     │
   │                       │                       │                     │
   │<──{token, user}──────│                       │                     │
   │                       │                       │                     │
   │──GET /profile───────>│                       │                     │
   │  (with token)         │                       │                     │
   │                       │──GET session───────>│                     │
   │                       │<────────────────────│                     │
   │<──{user data}────────│                       │                     │
```

### **Flow 2: Shopping Cart**
```
Frontend                Backend                 Redis
   │                       │                       │
   │──POST /cart/add─────>│                       │
   │  {productId, qty}    │                       │
   │                       │                       │
   │                       │──HSET cart──────────>│
   │                       │   userId:productId    │
   │                       │                       │
   │<──{cart}─────────────│                       │
   │                       │                       │
   │──GET /cart──────────>│                       │
   │                       │                       │
   │                       │──HGETALL cart───────>│
   │                       │<─────────────────────│
   │                       │                       │
   │<──{items}────────────│                       │
   │                       │                       │
   │──POST /checkout─────>│                       │
   │                       │                       │
   │                       │──HGETALL cart───────>│
   │                       │──Create Order───────>│ PostgreSQL
   │                       │──DEL cart───────────>│
   │                       │                       │
   │<──{order}────────────│                       │
```

### **Flow 3: Product Caching**
```
Frontend                Backend                 Redis              PostgreSQL
   │                       │                       │                     │
   │──GET /products──────>│                       │                     │
   │                       │                       │                     │
   │                       │──GET cache:products─>│                     │
   │                       │<─────(miss)──────────│                     │
   │                       │                       │                     │
   │                       │──SELECT * FROM─────>│                     │
   │                       │   products            │                     │
   │                       │<─────────────────────│                     │
   │                       │                       │                     │
   │                       │──SET cache──────────>│                     │
   │                       │   EX 300              │                     │
   │                       │                       │                     │
   │<──{products}─────────│                       │                     │
   │                       │                       │                     │
   │  (ครั้งที่ 2)          │                       │                     │
   │                       │                       │                     │
   │──GET /products──────>│                       │                     │
   │                       │                       │                     │
   │                       │──GET cache:products─>│                     │
   │                       │<─────(hit)───────────│                     │
   │                       │                       │                     │
   │<──{products}─────────│ (เร็วมาก!)             │                     │
```

---

##  **โครงสร้างโปรเจกต์**

```
redis-complete-system/
├── docker-compose.yml           # Orchestrate ทุก services
├── backend/                     # NestJS Application
│   ├── src/
│   │   ├── main.ts             # Entry point
│   │   ├── app.module.ts       # Main module
│   │   │
│   │   ├── redis/              # Redis Core
│   │   │   ├── redis.service.ts    # ทุก Redis operations
│   │   │   ├── redis.module.ts
│   │   │   └── redis.config.ts
│   │   │
│   │   ├── users/              # Users Module
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   └── dto/
│   │   │
│   │   ├── products/           # Products Module
│   │   │   ├── products.controller.ts
│   │   │   ├── products.service.ts
│   │   │   └── dto/
│   │   │
│   │   ├── orders/             # Orders Module
│   │   │   ├── orders.controller.ts
│   │   │   ├── orders.service.ts
│   │   │   └── dto/
│   │   │
│   │   ├── analytics/          # Analytics Module
│   │   │   ├── analytics.controller.ts
│   │   │   └── analytics.service.ts
│   │   │
│   │   ├── leaderboard/        # Leaderboard Module
│   │   │   ├── leaderboard.controller.ts
│   │   │   └── leaderboard.service.ts
│   │   │
│   │   ├── rate-limiter/       # Rate Limiter Module
│   │   │   ├── rate-limiter.guard.ts
│   │   │   └── rate-limiter.service.ts
│   │   │
│   │   ├── session/            # Session Module
│   │   │   ├── session.guard.ts
│   │   │   └── session.service.ts
│   │   │
│   │   ├── guards/             # Custom Guards
│   │   ├── interceptors/       # Cache Interceptor
│   │   ├── middleware/         # Logging Middleware
│   │   └── config/             # Configuration
│   │
│   ├── prisma/
│   │   ├── schema.prisma       # Database Schema
│   │   └── seed.ts             # Seed Data
│   │
│   ├── test/                   # E2E Tests
│   ├── .env.example
│   └── package.json
│
├── frontend/                   # React Application
│   ├── src/
│   │   ├── App.tsx            # Main App + Routing
│   │   ├── main.tsx
│   │   │
│   │   ├── pages/             # Pages
│   │   │   ├── HomePage.tsx
│   │   │   ├── UsersPage.tsx
│   │   │   ├── ProductsPage.tsx
│   │   │   ├── LeaderboardPage.tsx
│   │   │   ├── AnalyticsPage.tsx
│   │   │   └── DemoPage.tsx
│   │   │
│   │   ├── components/        # Reusable Components
│   │   │   ├── UserManagement.tsx
│   │   │   ├── ProductList.tsx
│   │   │   ├── ShoppingCart.tsx
│   │   │   ├── Leaderboard.tsx
│   │   │   ├── Analytics.tsx
│   │   │   ├── RateLimitDemo.tsx
│   │   │   ├── MessageQueue.tsx
│   │   │   └── SessionDemo.tsx
│   │   │
│   │   ├── services/          # API Services
│   │   │   └── api.ts
│   │   │
│   │   ├── hooks/             # Custom Hooks
│   │   │   └── useWebSocket.ts
│   │   │
│   │   └── types/             # TypeScript Types
│   │
│   ├── index.html
│   ├── vite.config.ts
│   └── package.json
│
├── docs/                      # Documentation
│   ├── REDIS_USE_CASES.md
│   ├── API_REFERENCE.md
│   ├── ARCHITECTURE.md
│   └── PERFORMANCE.md
│
└── scripts/                   # Utility Scripts
    ├── start-dev.sh
    ├── seed-db.sh
    └── reset-redis.sh
```

---

## 🛠️ **Technology Stack**

### **Backend**
- **NestJS** - Node.js Framework
- **Prisma** - ORM
- **ioredis** - Redis Client
- **PostgreSQL** - Relational Database
- **Redis 7** - In-memory Database
- **JWT** - Authentication
- **Class Validator** - DTO Validation
- **Swagger** - API Documentation

### **Frontend**
- **React 18** - UI Library
- **TypeScript** - Type Safety
- **Vite** - Build Tool
- **React Router** - Routing
- **Axios** - HTTP Client
- **Socket.io Client** - WebSocket
- **TailwindCSS** - Styling
- **Recharts** - Charts & Analytics

### **Infrastructure**
- **Docker** - Containerization
- **Docker Compose** - Orchestration
- **Redis Commander** - Redis UI
- **Nginx** - Web Server (Production)

---

## 🎮 **Features ที่สามารถทดสอบได้**

### **1. Session Management**
- ✅ Login/Logout
- ✅ Session persistence
- ✅ Auto-expire (30 นาที)
- ✅ Multiple devices

### **2. Caching System**
- ✅ Cache products
- ✅ Cache users
- ✅ Cache invalidation
- ✅ Cache hit/miss statistics

### **3. Shopping Cart**
- ✅ Add to cart
- ✅ Update quantity
- ✅ Remove items
- ✅ Checkout
- ✅ Cart persistence

### **4. Leaderboard**
- ✅ Real-time rankings
- ✅ Multiple categories
- ✅ Score updates
- ✅ User rank lookup

### **5. Analytics**
- ✅ Page view tracking
- ✅ User retention (Bitmap)
- ✅ Active users count
- ✅ Popular pages

### **6. Rate Limiting**
- ✅ 10 requests/minute
- ✅ Per-user limiting
- ✅ Custom error messages
- ✅ Countdown timer

### **7. Message Queue**
- ✅ Add to queue
- ✅ Process queue
- ✅ Queue length monitoring
- ✅ Real-time updates

### **8. Monitoring**
- ✅ Redis memory usage
- ✅ Keys count
- ✅ Connected clients
- ✅ Operations/sec

---

## 🚀 **การทำงานจริง**

### **Example: User ซื้อสินค้า**

```
1. User login
   → สร้าง session ใน Redis (STRING)
   
2. User browse products
   → ดึงจาก cache (STRING)
   → ถ้าไม่มี → query DB → save cache
   
3. User add to cart
   → บันทึกใน Redis Hash (HASH)
   
4. User checkout
   → อ่าน cart จาก Redis
   → สร้าง order ใน PostgreSQL
   → ใช้ Global ID (STRING - INCR)
   → เพิ่มใน queue (LIST)
   → ลบ cart
   
5. ระบบ process order
   → Worker ดึงจาก queue
   → ประมวลผล
   → อัพเดท stock
   
6. User ดู leaderboard
   → ดึงจาก Sorted Set (ZSET)
   
7. ระบบ track activity
   → บันทึก Bitmap (BITMAP)
   → นับ page view (STRING - INCR)
```

---

## 📊 **Redis Keys Pattern ที่ใช้**

```
Sessions:        session:{userId}
Cache:           cache:{entity}:{id}
Locks:           lock:{resource}:{id}
Counters:        counter:{type}:{id}
Rate Limits:     rate:{userId}:{endpoint}
Global IDs:      id:{sequenceName}
Carts:           cart:{userId}
Bitmaps:         active:{YYYY-MM-DD}
Queues:          queue:{queueName}
Leaderboards:    leaderboard:{category}
Tags:            tags:{itemId}
Page Views:      pageview:{pageName}
```

---

## 🎯 **สิ่งที่ได้เรียนรู้**

1. **Redis Data Types** - เข้าใจการใช้งานแต่ละ type
2. **Caching Strategies** - Cache-aside, Write-through
3. **Session Management** - Stateless authentication
4. **Rate Limiting** - ป้องกัน abuse
5. **Real-time Features** - WebSocket + Redis Pub/Sub
6. **Message Queues** - Async processing
7. **Analytics** - Bitmap, HyperLogLog
8. **Performance** - ลด load database
9. **Scalability** - Horizontal scaling
10. **Best Practices** - Key naming, TTL, Memory management

---

ระบบนี้คือ **Redis Playground** ที่สมบูรณ์ สำหรับเรียนรู้และทดลองใช้งาน Redis ทุก Use Case ในโลกจริง! 🎉
