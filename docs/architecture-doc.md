# 🏗️ System Architecture

## Overview

ระบบนี้ออกแบบเป็น **Dual System Architecture** ที่ประกอบด้วย:
1. **Universal RAG Platform** - Multi-tenant platform สำหรับทุกองค์กร
2. **English Mania Education System** - Single-tenant education management

---

## 🎨 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      External Users                             │
│     (Web Browser, LINE OA, API, Claude Desktop MCP)            │
└──────────────────┬────────────────────────────────┬─────────────┘
                   │                                │
                   ↓                                ↓
    ┌──────────────────────────┐      ┌────────────────────────┐
    │  Universal RAG Platform  │      │  English Mania System  │
    │  (Multi-tenant)          │      │  (Single-tenant)       │
    └────────────┬─────────────┘      └───────────┬────────────┘
                 │                                 │
                 ↓                                 ↓
    ┌──────────────────────────┐      ┌────────────────────────┐
    │  Load Balancer           │      │  Cloudflare Worker     │
    │  (Nginx/Traefik)         │      │  (LINE Webhook)        │
    └────────────┬─────────────┘      └───────────┬────────────┘
                 │                                 │
                 ↓                                 ↓
    ┌──────────────────────────┐      ┌────────────────────────┐
    │  Next.js Frontend        │      │  FastAPI Backend       │
    │  + FastAPI Backend       │      │  (Education Logic)     │
    └────────────┬─────────────┘      └───────────┬────────────┘
                 │                                 │
                 └─────────────┬───────────────────┘
                               ↓
              ┌────────────────────────────────────┐
              │      Shared Infrastructure         │
              │  ┌──────────┬──────────┬────────┐ │
              │  │PostgreSQL│  Qdrant  │ Redis  │ │
              │  │(Main DB) │(Vectors) │(Cache) │ │
              │  └──────────┴──────────┴────────┘ │
              │  ┌──────────┬──────────┬────────┐ │
              │  │  MinIO   │ Gemini   │Customer│ │
              │  │(Storage) │   AI     │   DBs  │ │
              │  └──────────┴──────────┴────────┘ │
              └────────────────────────────────────┘
```

---

## 🔄 Request Flow

### 1. Universal RAG Platform Flow

```
User (Web) → Nginx → Next.js Frontend → FastAPI Backend
                                              ↓
                                        ┌─────┴─────┐
                                        │           │
                                        ↓           ↓
                                   Query Router  Document Search
                                        ↓           ↓
                                  SQL Agent    Qdrant Vector DB
                                        ↓           ↓
                                   Customer DB  Embeddings
                                        ↓           ↓
                                        └─────┬─────┘
                                              ↓
                                         Gemini AI
                                              ↓
                                      Response Stream
                                              ↓
                                         User (Web)
```

### 2. English Mania LINE Flow

```
LINE User → LINE Platform → Cloudflare Worker
                                   ↓
                            Verify Signature
                                   ↓
                            Extract user_id & role
                                   ↓
                            FastAPI Backend
                                   ↓
                       ┌───────────┴───────────┐
                       ↓                       ↓
                  PostgreSQL              Gemini AI
                  (User data)            (Conversation)
                       ↓                       ↓
                   └───────────┬───────────────┘
                               ↓
                      LINE Reply Message
                               ↓
                           LINE User
```

---

## 🗄️ Database Architecture

### PostgreSQL (Main Database)

```
┌─────────────────────────────────────────────┐
│            PostgreSQL 16                    │
├─────────────────────────────────────────────┤
│                                             │
│  📦 Universal Platform Tables               │
│  ├── organizations                          │
│  ├── workspaces                             │
│  ├── users                                  │
│  ├── database_connections                   │
│  ├── documents                              │
│  ├── chat_threads                           │
│  └── messages                               │
│                                             │
│  📦 English Mania Tables                    │
│  ├── em_users                               │
│  ├── em_students                            │
│  ├── em_enrollments                         │
│  ├── em_attendance                          │
│  ├── em_teaching_logs                       │
│  ├── em_homework                            │
│  ├── em_payments                            │
│  ├── em_expenses                            │
│  └── em_broadcasts                          │
│                                             │
│  📦 Shared Tables (from Google Sheets)      │
│  ├── courses                                │
│  ├── schedules                              │
│  ├── teachers                               │
│  ├── pricing                                │
│  ├── faq                                    │
│  └── policies                               │
│                                             │
└─────────────────────────────────────────────┘
```

**ดูรายละเอียด:** [database/README.md](database/README.md)

---

## 🧠 Vector Database (Qdrant)

```
┌─────────────────────────────────────────────┐
│              Qdrant 1.7+                    │
├─────────────────────────────────────────────┤
│                                             │
│  Collection: org_{org_id}_workspace_{ws_id} │
│  ├── Vector dimension: 3072                 │
│  ├── Distance metric: Cosine                │
│  └── Payload:                               │
│      ├── document_id                        │
│      ├── filename                           │
│      ├── chunk_index                        │
│      ├── text                               │
│      └── metadata                           │
│                                             │
└─────────────────────────────────────────────┘
```

**Features:**
- Multi-tenancy via collections
- Fast similarity search (<100ms)
- Filtering by metadata
- Batch operations

---

## 🔌 Integration Points

### 1. LLM Providers

```python
# Abstraction Layer
class LLMProvider(ABC):
    @abstractmethod
    async def generate(self, messages: List[Message]) -> str:
        pass
    
    @abstractmethod
    async def stream(self, messages: List[Message]) -> AsyncIterator[str]:
        pass

# Implementations
- GeminiProvider (Primary)
- OpenAIProvider
- ClaudeProvider
- LlamaProvider (self-hosted)
```

### 2. Database Connectors

```python
# SQL Agent
class DatabaseConnector(ABC):
    @abstractmethod
    async def execute_query(self, query: str) -> List[Dict]:
        pass
    
    @abstractmethod
    async def get_schema(self) -> Dict:
        pass

# Implementations
- MSSQLConnector
- MySQLConnector
- PostgreSQLConnector
```

### 3. Document Processors

```python
# Document Processor
class DocumentProcessor(ABC):
    @abstractmethod
    async def process(self, file: UploadFile) -> List[Chunk]:
        pass

# Implementations
- DOCXProcessor
- PDFProcessor (with OCR)
- XLSXProcessor
- TXTProcessor
- CSVProcessor
- MarkdownProcessor
```

---

## 🔐 Security Architecture

### Authentication Flow

```
User → Login Request
        ↓
   Verify Credentials (bcrypt)
        ↓
   Generate JWT Token
        ↓
   ┌─────────────────┐
   │ JWT Token       │
   │ ├── user_id     │
   │ ├── org_id      │
   │ ├── role        │
   │ └── exp (1h)    │
   └─────────────────┘
        ↓
   Store in HTTP-only Cookie
        ↓
   User gets Access
```

### Authorization (RBAC)

```
Request → Extract JWT → Decode Token
                            ↓
                    Check Permission
                            ↓
            ┌───────────────┼───────────────┐
            ↓               ↓               ↓
         Admin          Manager          Member
            │               │               │
            ↓               ↓               ↓
      All Access    Limited Access    Read Only
```

### Data Isolation

```
Multi-tenant Isolation:
├── Row-Level Security (RLS)
│   └── WHERE organization_id = current_org_id
├── Separate Vector Collections
│   └── org_{org_id}_workspace_{ws_id}
└── Encrypted Credentials
    └── AES-256 encryption for DB passwords
```

---

## 📦 Service Architecture

### Microservices (Future - Phase 4)

```
┌──────────────────────────────────────────────┐
│         API Gateway (Nginx/Kong)             │
└──────────┬───────────────────────────────────┘
           │
    ┌──────┼──────┬──────┬──────┬──────┐
    ↓      ↓      ↓      ↓      ↓      ↓
  Auth   Query  Vector  Doc   LINE  Admin
Service Service Store Process Hook Service
    │      │      │      │      │      │
    └──────┴──────┴──────┴──────┴──────┘
                   ↓
          Message Queue (RabbitMQ)
                   ↓
            Event Bus (Redis)
```

**Current (Phase 1-2):** Monolith FastAPI  
**Future (Phase 4):** Microservices

---

## 🚀 Deployment Architecture

### Phase 1: Self-hosted (Current)

```
Dell R730xd (128GB RAM, 30TB Storage)
├── Proxmox VE 8.2
│   ├── VM 1: Ubuntu 22.04 (Docker Host)
│   │   ├── PostgreSQL (16GB RAM, 500GB SSD)
│   │   ├── Qdrant (8GB RAM, 200GB SSD)
│   │   ├── Redis (4GB RAM, 50GB SSD)
│   │   ├── MinIO (8GB RAM, 10TB HDD)
│   │   ├── FastAPI Backend (16GB RAM)
│   │   ├── Next.js Frontend (8GB RAM)
│   │   └── Nginx (2GB RAM)
│   │
│   └── VM 2: Monitoring Stack
│       ├── Prometheus (4GB RAM)
│       └── Grafana (2GB RAM)
│
└── External Services
    ├── Cloudflare Worker (LINE webhook)
    ├── Gemini API (LLM)
    └── LINE Messaging API
```

### Phase 2: Hybrid (Year 2)

```
On-Premise (Dell R730xd):
├── PostgreSQL (main data)
├── Qdrant (vectors)
├── Redis (cache)
└── Customer Databases

Cloud:
├── Vercel (Next.js Frontend)
├── Cloudflare R2 (Documents)
├── Cloud Run (Workers)
└── External APIs
```

### Phase 3: Full Cloud (Year 3+)

```
Kubernetes Cluster (GKE/EKS)
├── Namespace: platform
│   ├── Deployment: api (3+ replicas)
│   ├── Deployment: web (2+ replicas)
│   ├── Deployment: workers (5+ replicas)
│   └── Service: Load Balancer
│
├── Namespace: data
│   ├── StatefulSet: postgresql (HA)
│   ├── StatefulSet: qdrant (clustered)
│   └── StatefulSet: redis (sentinel)
│
└── Namespace: monitoring
    ├── Prometheus
    ├── Grafana
    └── Loki
```

**ดูรายละเอียด:** [DEPLOYMENT.md](DEPLOYMENT.md)

---

## 🔄 Data Flow Diagrams

### Document Upload & Processing

```
User uploads file
        ↓
   Save to MinIO
        ↓
   Queue processing job (Celery)
        ↓
   Worker picks up job
        ↓
   ┌────────────────┐
   │ Process File   │
   │ ├── Extract    │
   │ ├── Chunk      │
   │ └── Embed      │
   └────────────────┘
        ↓
   Store vectors in Qdrant
        ↓
   Update document status
        ↓
   Notify user (WebSocket)
```

### Query Processing

```
User asks question
        ↓
   Classify query type
        ↓
   ┌─────────┴─────────┐
   ↓                   ↓
Database Query    Document Search
   ↓                   ↓
SQL Agent         Vector Search
   ↓                   ↓
Execute SQL       Retrieve chunks
   ↓                   ↓
Get results       Get context
   ↓                   ↓
   └─────────┬─────────┘
             ↓
      Combine context
             ↓
       Send to LLM
             ↓
    Generate answer
             ↓
    Stream to user
```

---

## 📊 Scalability Considerations

### Horizontal Scaling

```
Component          | Phase 1  | Phase 2  | Phase 3
-------------------|----------|----------|----------
FastAPI Backend    | 1        | 3        | 10+
Next.js Frontend   | 1        | 2        | 5+
PostgreSQL         | 1        | 1 (HA)   | 3 (cluster)
Qdrant             | 1        | 1        | 3 (cluster)
Redis              | 1        | 1        | 3 (sentinel)
Workers (Celery)   | 2        | 5        | 20+
```

### Bottlenecks & Solutions

```
Bottleneck              | Solution
------------------------|----------------------------------
Database queries        | Read replicas, Indexing, Caching
Vector search           | Qdrant clustering, Sharding
LLM API rate limits     | Request queuing, Multiple providers
File uploads            | CDN, Direct S3 upload
Concurrent users        | Horizontal scaling, Load balancing
```

---

## 🔧 Technology Decisions

### Why PostgreSQL?
- ✅ Mature RBAC support
- ✅ JSONB for flexible schemas
- ✅ Strong ACID guarantees
- ✅ Excellent tooling
- ✅ pgvector extension (optional)

### Why Qdrant?
- ✅ Self-hosted (privacy)
- ✅ Fast (<100ms searches)
- ✅ Built-in filtering
- ✅ Easy clustering
- ✅ Rust performance

### Why FastAPI?
- ✅ Async/await support
- ✅ Auto OpenAPI docs
- ✅ Type hints
- ✅ Fast development
- ✅ Great ecosystem

### Why Next.js?
- ✅ SSR + SSG support
- ✅ API routes
- ✅ Image optimization
- ✅ Great DX
- ✅ Vercel deployment

### Why Gemini API?
- ✅ Cheapest ($0.01/1K tokens)
- ✅ Large context (1M tokens)
- ✅ Fast response
- ✅ Multimodal support
- ✅ Thai language support

---

## 📈 Performance Targets

### Phase 1 (Self-hosted)
```
Metric                    | Target
--------------------------|------------
API Response Time         | <200ms (p95)
Query Processing          | <2s (p95)
Vector Search             | <100ms (p95)
Document Upload           | <5s (10MB file)
Concurrent Users          | 200
Uptime                    | 99% (8.76h downtime/year)
```

### Phase 3 (Cloud)
```
Metric                    | Target
--------------------------|------------
API Response Time         | <100ms (p95)
Query Processing          | <1s (p95)
Vector Search             | <50ms (p95)
Document Upload           | <2s (10MB file)
Concurrent Users          | 10,000
Uptime                    | 99.9% (52.6min downtime/year)
```

---

## 🔄 Future Enhancements

### Phase 4+
- [ ] GraphQL API
- [ ] WebRTC for video calls
- [ ] Real-time collaboration
- [ ] AI-powered recommendations
- [ ] Predictive analytics
- [ ] Edge computing (Cloudflare Workers)
- [ ] Blockchain for audit trail (optional)

---

**Last Updated:** 2025-10-19  
**Version:** 1.0.0