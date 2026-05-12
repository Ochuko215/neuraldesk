## NeuralDesk  AI-Powered SaaS Workspace

> A multi-tenant SaaS platform combining an AI writing assistant, code generation, and team collaboration  built for scale.

![Next.js](https://img.shields.io/badge/Next.js_14-000000?style=for-the-badge&logo=next.js&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)
![Python](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=for-the-badge&logo=openai&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)

-----

## Overview

NeuralDesk is a production grade, multi-tenant AI SaaS platform where teams can write, code, and collaborate — powered by OpenAI GPT-4o and Claude. It features a real-time document editor with AI inline assistance, a code generation sandbox, workspace/team management, and Stripe subscription billing with usage metering.

This is a full product  not a demo. It handles multi-tenancy at the database level (row-level security), streaming LLM responses, token usage tracking per workspace, and rate-limiting per subscription tier.

-----

## Features

### Core Product

 **AI Writing Assistant** Inline GPT-4o suggestions, rewrite, summarize, expand, tone adjustment
 **Code Generation Sandbox**  AI code generation with syntax highlighting and in-browser execution
 **Rich Document Editor** Block-based editor (TipTap) with slash commands and real-time autosave
 **Semantic Search** Vector-powered search across all workspace documents (pgvector + embeddings)
 **Team Workspaces**  Invite members, role-based access (owner/admin/member), per-workspace AI usage limits

### Platform

 **Multi-Tenancy**  Full tenant isolation with PostgreSQL Row-Level Security
 **Subscription Billing**  Stripe Billing with Free/Pro/Team tiers and usage-based metering
 **Auth**  OAuth (Google, GitHub) + magic link via NextAuth.js
**Usage Dashboard**  Real-time token usage, cost tracking, and quota management per workspace
 **Streaming Responses** SSE-based streaming for instant AI output feel
 **Internationalization**  i18n support via next-intl

-----

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Client (Next.js)                  │
│  App Router · RSC · TipTap Editor · shadcn/ui        │
└──────────────────────┬──────────────────────────────┘
                       │ HTTPS / SSE
┌──────────────────────▼──────────────────────────────┐
│              API Gateway (Next.js API Routes)         │
│  Auth middleware · Rate limiting · Tenant resolution  │
└──────────┬───────────────────────┬──────────────────┘
           │                       │
┌──────────▼──────────┐  ┌────────▼────────────────┐
│   AI Service        │  │   Core API              │
│   (FastAPI/Python)  │  │   (Node.js/Express)     │
│                     │  │                         │
│ • OpenAI SDK        │  │ • Workspace CRUD        │
│ • Anthropic SDK     │  │ • Document management   │
│ • LangChain         │  │ • User/billing/roles    │
│ • Embedding gen     │  │ • Stripe webhooks       │
│ • Token metering    │  └────────────┬────────────┘
└──────────┬──────────┘               │
           │                          │
┌──────────▼──────────────────────────▼──────────────┐
│                  Data Layer                          │
│  PostgreSQL (RLS) · pgvector · Redis · S3           │
└─────────────────────────────────────────────────────┘
```

**Key architectural decisions:**

 **Separate AI microservice (FastAPI)**  decouples LLM logic from business logic; allows independent scaling of AI compute
**PostgreSQL RLS for multi-tenancy** enforced at the database level, not application level  eliminates entire class of data leakage bugs
 **pgvector for embeddings**  keeps vector search co-located with relational data, avoiding a separate vector DB for this scale
 **Redis for rate limiting + session cache**  token bucket algorithm per workspace/tier
**SSE over WebSockets for streaming**  simpler infra, works through proxies, sufficient for unidirectional AI output

-----

## Tech Stack

|Layer     |Technology                                              |Reason                                     |
|----------|--------------------------------------------------------|-------------------------------------------|
|Frontend  |Next.js 14 (App Router), TypeScript, Tailwind, shadcn/ui|RSC for performance, type safety throughout|
|Editor    |TipTap                                                  |Extensible block editor, works with React  |
|AI Service|FastAPI (Python), LangChain, OpenAI SDK                 |Python ecosystem best for LLM tooling      |
|Core API  |Node.js, Express                                        |Familiar, fast, good Stripe/auth ecosystem |
|Database  |PostgreSQL + pgvector                                   |Relational + vector in one DB              |
|Cache     |Redis (Upstash)                                         |Rate limiting, session, queue              |
|Auth      |NextAuth.js                                             |OAuth + magic link out of the box          |
|Billing   |Stripe (subscriptions + metering)                       |Industry standard                          |
|Storage   |AWS S3                                                  |File/image uploads                         |
|Infra     |AWS ECS (Fargate), RDS, ElastiCache                     |Serverless containers, managed DB/cache    |
|CI/CD     |GitHub Actions                                          |Build, test, deploy on merge to main       |

-----

## Getting Started

### Prerequisites

- Node.js 18+, Python 3.11+
- PostgreSQL 15+ with pgvector extension
- Redis
- OpenAI API key
- Stripe account

### Installation

```bash
git clone https://github.com/Ochuko215/neuraldesk.git
cd neuraldesk

# Backend (AI Service)
cd ai-service
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # add OPENAI_API_KEY, ANTHROPIC_API_KEY

# Backend (Core API)
cd ../api
npm install
cp .env.example .env

# Frontend
cd ../web
npm install
cp .env.example .env.local

# Database setup
cd ../api
npm run db:migrate
npm run db:seed

# Start everything
docker-compose up  # or run each service individually
```

### Environment Variables

```env
# AI Service
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
DATABASE_URL=postgresql://...

# Core API
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Web
NEXTAUTH_SECRET=...
NEXTAUTH_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:4000
```

-----

## Project Structure

```
neuraldesk/
├── web/                    # Next.js 14 frontend
│   ├── app/                # App Router pages
│   │   ├── (auth)/         # Login, signup
│   │   ├── (dashboard)/    # Workspace, documents
│   │   └── (marketing)/    # Landing page, pricing
│   ├── components/         # UI components
│   └── lib/                # API client, hooks, utils
├── api/                    # Node.js core API
│   ├── src/
│   │   ├── routes/         # REST endpoints
│   │   ├── middleware/      # Auth, tenant, rate limit
│   │   ├── db/             # Prisma schema + migrations
│   │   └── services/       # Stripe, email, storage
├── ai-service/             # FastAPI AI microservice
│   ├── routers/            # /complete, /embed, /search
│   ├── services/           # LLM clients, token metering
│   └── models/             # Pydantic schemas
├── infra/                  # Terraform (AWS)
└── docker-compose.yml
```

-----

## 📊 Subscription Tiers

|Feature        |Free|Pro ($19/mo)|Team ($49/mo)|
|---------------|----|------------|-------------|
|AI tokens/month|50K |500K        |2M           |
|Documents      |10  |Unlimited   |Unlimited    |
|Workspaces     |1   |3           |Unlimited    |
|Members        |1   |1           |10           |
|Code sandbox   |❌   |✅           |✅            |
|Semantic search|❌   |✅           |✅            |

-----

##  Security

- All tenant data isolated via PostgreSQL Row-Level Security policies
- API keys encrypted at rest (AES-256)
- Rate limiting per workspace per tier (Redis token bucket)
- Input sanitization and prompt injection hardening on AI service
- SOC 2-ready audit logging

-----

## License

MIT © Ogheneochuko Iredia
