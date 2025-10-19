# 🤖 Enterprise RAG Platform + Education Management System

> **Universal Multi-Tenant Platform** for transforming Database + Documents into AI Assistant
> **Complete School Management System** integrated with LINE Official Account

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.109+-00a393.svg)](https://fastapi.tiangolo.com/)
[![Next.js](https://img.shields.io/badge/Next.js-14-black)](https://nextjs.org/)

---

## 🎯 Overview

This system combines **2 main platforms**:

### 1️⃣ Enterprise RAG Platform (Universal)
Self-hosted platform enabling any organization to create an AI Assistant that can:
- 🔍 **Query Real-time Database** - Sales, inventory, customer data
- 📚 **Search Documents** - Policies, manuals, reports
- 🤖 **Multi-LLM Support** - OpenAI, Gemini, Claude, Llama
- 🏢 **Multi-Tenant** - Multiple organizations + workspaces
- 💬 **Multi-Channel** - Web + LINE Official Account
- 🔒 **Privacy-First** - Data stays on your server

### 2️⃣ Education Management System (Specialized)
Complete Education Management System on LINE OA for:
- 👨‍🏫 **Teachers** - Teaching logs, attendance, homework assignment
- 👨‍👩‍👧 **Parents** - View grades, payments, download receipts
- 👶 **Students** - View schedule, homework, check scores
- 🏢 **Administrators** - Financial overview, statistics, decision making

---

## ✨ Key Features

### 🏢 Universal Platform Features

- ✅ **Multi-Tenant Architecture** - Multiple organizations, separate workspaces, RBAC
- ✅ **Universal Database Connector** - MSSQL, MySQL, PostgreSQL (read-only)
- ✅ **Document Processing** - DOCX, PDF, XLSX, TXT, CSV, MD + OCR
- ✅ **Pluggable LLM** - Gemini, OpenAI, Claude, Llama, Custom endpoints
- ✅ **Vector Database** - Qdrant, Pinecone, Weaviate, Milvus
- ✅ **Multi-Channel** - Web App + LINE OA (supports 80,000+ users)

### 🎓 Education System Features

#### 👨‍🏫 Teacher Portal (LINE OA)
- ✅ Student attendance - Quick check, auto notify parents
- ✅ Teaching logs - Topics, activities, homework, prep notes
- ✅ Homework assignment - Auto distribute to students
- ✅ Grading & Feedback - Grade homework, provide comments

#### 👨‍👩‍👧 Parent Portal (LINE OA)
- ✅ View child information - All children, courses, schedules
- ✅ Payment - Upload slip (OCR), payment history
- ✅ Download receipts - Auto-generate PDF
- ✅ Check remaining hours - Course-based + Hour-based
- ✅ View academic results - Scores, attendance, progress

#### 👶 Student Portal (LINE OA)
- ✅ View schedule - Weekly schedule, next class
- ✅ Homework - View pending, submit work, check grades
- ✅ Gamification - Points, badges, leaderboard, rewards

#### 🏢 Admin Dashboard (MCP Server + Web)
- ✅ Financial overview - Revenue, expenses, profit/loss, trends
- ✅ Payment approval - Review slips, generate receipts
- ✅ Broadcast messages - Announcements, targeted messaging

---

## 🛠️ Tech Stack

### Backend
- **Python 3.11+** - FastAPI 0.109+
- **Database** - PostgreSQL 16 (main), Qdrant (vectors)
- **Cache** - Redis 7
- **Storage** - MinIO (S3-compatible)
- **AI/ML** - LangChain 0.1+, Gemini 1.5 Pro, text-embedding-3-large

### Frontend
- **Next.js 14** (App Router) - TypeScript 5+
- **React 18** - Tailwind CSS 3
- **State** - Zustand / TanStack Query

### DevOps
- **Docker 24+** - Docker Compose / Kubernetes (k3s)
- **Reverse Proxy** - Nginx / Traefik
- **Monitoring** - Prometheus + Grafana
- **SSL** - Let's Encrypt

---

## 🚀 Quick Start

```bash
# 1. Clone repository
git clone https://github.com/bornja55/RAG-LLM-Platform.git
cd RAG-LLM-Platform

# 2. Run installation wizard
./install.sh

# 3. Access platform
https://your-domain.com

# Default credentials
Email: admin@example.com
Password: (generated during installation)
```

**For detailed installation:** See [`docs/install-guide.md`](docs/install-guide.md)

---

## 📚 Documentation

### For Users
- 📖 User Guide - How to use the platform
- 🎓 Education System Guide - School management manual
- ❓ FAQ - Frequently asked questions

### For Developers
- 🏗️ [Architecture](docs/architecture-doc.md) - System architecture
- 📋 [Development Tasks](docs/dev-tasks.md) - Development roadmap
- 🗄️ [Database Schema](docs/database-readme.md) - Database structure
- 🔄 [User Flows](docs/flows-readme.md) - User flows for all roles
- 🚀 [Deployment Guide](docs/deployment-guide.md) - Deployment manual
- 📊 [Cost Analysis](docs/cost-analysis.md) - Cost breakdown

### API Documentation
- 🔗 API Reference - API documentation
- 🌐 Interactive Docs:
  - Swagger UI: `https://your-domain.com/docs`
  - ReDoc: `https://your-domain.com/redoc`

---

## 🎯 Roadmap

### ✅ Phase 0: Foundation (Week 1-4) - DONE
- Architecture design
- Tech stack selection
- Database schema design

### 🚧 Phase 1: Universal Platform Core (Week 5-12) - IN PROGRESS
- Multi-tenant architecture
- Database integration (MSSQL, MySQL, PostgreSQL)
- Document RAG
- Web + LINE basic interface

### 📋 Phase 2: Education System Core (Week 13-20)
- Role-based system (Teacher/Parent/Student/Admin)
- Payment system with OCR
- Homework & grading system
- MCP Server for admin

### 📋 Phase 3: Enhanced UX (Week 21-28)
- LINE Flex Messages & Rich Menu
- Advanced analytics
- Auto report generation
- Performance optimization

### 📋 Phase 4: Enterprise Features (Week 29-40)
- SSO integration
- High availability
- Mobile apps
- Advanced customization

**For detailed roadmap:** See [`docs/dev-tasks.md`](docs/dev-tasks.md)

---

## 💰 Cost Overview

### Self-hosted (Year 1)
```
Monthly: $400-800
- Gemini API: $250-500
- LINE API: $80
- Electricity: $20-30
- Internet: $50-150

Hardware: $0 (owned Dell R730xd)
```

### Hybrid (Year 2)
```
Monthly: $1,000-2,000
- On-premise: Database, Redis
- Cloud: Frontend (Vercel), Storage (R2), Workers
```

### Full Cloud (Year 3+)
```
Monthly: $5,000-15,000
- Kubernetes cluster
- Managed databases
- Multi-region deployment
- 99.9% uptime SLA
```

**For detailed cost analysis:** See [`docs/cost-analysis.md`](docs/cost-analysis.md)

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

### Development Setup

```bash
# Clone repo
git clone https://github.com/bornja55/RAG-LLM-Platform.git
cd RAG-LLM-Platform

# Backend setup
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements-dev.txt

# Frontend setup
cd ../frontend
npm install

# Start development servers
docker-compose -f docker-compose.dev.yml up
```

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 📞 Support

- 🐛 Issues: https://github.com/bornja55/RAG-LLM-Platform/issues
- 📧 Email: support@rag-platform.com
- 📚 Documentation: Coming soon

---

## 🌟 Star History

If you find this project useful, please consider giving it a star!

---

<p align="center">
  Made with ❤️ for the community
</p>

<p align="center">
  <a href="https://github.com/bornja55/RAG-LLM-Platform">GitHub</a> •
  <a href="docs/architecture-doc.md">Documentation</a> •
  <a href="docs/dev-tasks.md">Roadmap</a>
</p>
