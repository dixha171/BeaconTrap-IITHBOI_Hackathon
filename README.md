# 🛡️ BeaconTrap

**A Generative AI-powered malware analysis and fraud intelligence platform for Android APKs, built for the PSB Cybersecurity, Fraud & AI Hackathon (IIT Hyderabad).**

BeaconTrap ingests suspicious Android APKs — banking trojans, KYC-spoofing apps, SMS interceptors, overlay attacks — and runs them through a full static + dynamic analysis pipeline, correlates them into campaigns using graph intelligence, scores their risk with an explainable engine, and produces tamper-evident, blockchain-anchored forensic reports for analysts, bank officers, and regulators.

🔗 Live frontend: [beacon-trap-frontend.vercel.app](https://beacon-trap-frontend.vercel.app)

---

## Why BeaconTrap

Banking-fraud malware on Android (fake KYC apps, SMS interceptors, overlay trojans, RATs) moves faster than manual triage can keep up with. BeaconTrap is a SOC-style command center that:

- Automates static + dynamic APK analysis instead of relying on manual reverse engineering
- Uses a multi-agent AI pipeline to explain *why* an app is malicious, not just flag it
- Correlates individual samples into **campaigns** (shared infrastructure, certificates, C2 domains) via a graph database
- Anchors every piece of evidence to a public blockchain so findings can't be silently altered after the fact
- Produces outputs tailored to three different audiences: security analysts, bank compliance officers, and affected citizens

---

## Architecture

```
                 React Frontend (SOC Dashboard)
                            │
                  HTTPS / WebSocket
                            │
                  FastAPI API Gateway
                            │
                  JWT Auth + RBAC Layer
                            │
                  RabbitMQ Job Queue
              ┌─────────────┴─────────────┐
              │                           │
      Static Analysis Worker      Dynamic Analysis Worker
    (JADX, Androguard, Semgrep)   (Emulator, Frida, mitmproxy)
              │                           │
              └─────────────┬─────────────┘
                             │
                    MinIO Artifact Store
                             │
                LangGraph AI Orchestrator
   ┌──────────┬──────────┬──────────┬──────────┬──────────┐
   │          │          │          │          │          │
Deobf.    Network     MITRE       GRC        Risk      Report
Agent      Intel      Mapping   Compliance  Scoring   Generation
   │          │          │          │          │          │
   └──────────┴────┬─────┴──────────┴──────────┴──────────┘
                    │
           Campaign DNA (Neo4j Graph)
                    │
        PostgreSQL · Redis · MinIO · Ethereum Sepolia
                    │
         SOC Dashboard, Executive Reports & Evidence Ledger
```

**Design principles:** modular over monolithic, event-driven and asynchronous processing, explainable AI at every scoring step, zero-trust security, and immutable forensic artifacts. One failed module (e.g. dynamic analysis timing out) never blocks the rest of the pipeline — it degrades gracefully to a deterministic report.

---

## Core Features

| Module | What it does |
|---|---|
| **SOC Command Center** | Real-time dashboard — active cases, threat metrics, MITRE ATT&CK heatmap, global threat map, live alert feed |
| **APK Ingestion & Sandbox** | Drag-and-drop upload, validation, and queued static/dynamic analysis jobs |
| **Static Analysis** | Manifest parsing, permission mapping, certificate inspection, JADX decompilation, Semgrep rule scanning |
| **Dynamic Analysis** | Emulator execution with Frida instrumentation, network capture (mitmproxy/tcpdump), real SMS/API hook interception |
| **AI Multi-Agent Pipeline** | LangGraph DAG — deobfuscation, MITRE ATT&CK mapping, network intelligence, GRC compliance, risk scoring, report generation, each as an independent, structured-context agent |
| **AI Copilot** | Conversational assistant that answers analyst questions using the case's actual artifacts |
| **Risk Engine** | Weighted, explainable risk scoring with confidence estimation and actionable recommendations |
| **Campaign DNA Graph** | Neo4j-backed correlation of APKs, domains, IPs, certificates, and malware families across campaigns |
| **Blockchain Evidence Ledger** | Every case's evidence hash is anchored on-chain (Ethereum Sepolia) via a Solidity smart contract for tamper-proof audit trails |
| **Executive & Regulatory Reporting** | Auto-generated PDF/HTML dossiers mapped to RBI, DPDP Act, and IT Act obligations, plus a printable audit view |
| **Citizen Impact View** | Plain-language, multilingual (i18next) summaries with text-to-speech for non-technical stakeholders |
| **RBAC** | JWT-based auth separating Analyst vs. Officer permissions server-side |

---

## Tech Stack

**Frontend** — React 19, TypeScript, Vite, TailwindCSS, TanStack Query, React Flow (`@xyflow/react`), Recharts, Framer Motion, ethers.js, i18next

**Backend** — Python, FastAPI, SQLAlchemy (PostgreSQL/SQLite), Redis, RabbitMQ (pika), MinIO, Androguard, Semgrep, Frida, JADX

**AI / Orchestration** — LangGraph, Gemini (Google Generative AI), Groq, Ollama

**Data & Intelligence** — Neo4j (campaign graph), PostgreSQL (transactional), Redis (cache/pub-sub), MinIO (artifact store)

**Blockchain** — Solidity smart contract (`EvidenceAnchor.sol`), Web3.py (backend), ethers.js (frontend), deployed on Ethereum Sepolia testnet

**Infra** — Docker Compose, per-service Dockerfiles for backend/frontend

---

## Project Structure

```
├── backend/            FastAPI application
│   ├── app/
│   │   ├── api/v1/        REST endpoints
│   │   ├── agents/        AI agent implementations
│   │   ├── genai/          LangGraph orchestrator + nodes
│   │   ├── llm/            LLM provider integrations (Gemini, Groq, Ollama)
│   │   ├── blockchain/     Web3/evidence-anchoring logic
│   │   ├── risk_engine/    Scoring, weights, confidence, recommendations
│   │   ├── report_engine/  PDF/HTML report generation
│   │   ├── repositories/   Data access layer
│   │   ├── models/         SQLAlchemy models
│   │   ├── services/       Manifest/JADX/Semgrep/Frida services
│   │   └── core/           Config, security, queueing
│   └── requirements.txt
├── frontend/            React + Vite SOC dashboard
│   └── src/
│       ├── components/     dashboard, lab, copilot, auth, layout panels
│       ├── context/        global state (AnalysisContext)
│       ├── hooks/, lib/, utils/, types/
├── workers/
│   ├── static_worker/      static analysis job consumer
│   └── dynamic_worker/     dynamic analysis job consumer + Frida hooks
├── contracts/            EvidenceAnchor.sol + ABI (Solidity)
├── docker/               Backend & frontend Dockerfiles
├── docs/                 Architecture, API reference, DB schema, feature docs
├── apks/                 Sample/test APKs (benign + malicious) for demo purposes
├── scripts/              APK generation / archive utilities
└── docker-compose.yml    Full local stack (Postgres, Redis, RabbitMQ, Neo4j, MinIO, backend, frontend)
```

Full docs live in [`/docs`](./docs) — see `ARCHITECTURE.md`, `API_REFERENCE.md`, `DATABASE_SCHEMA.md`, `FEATURES.md`, `DYNAMIC_ANALYSIS.md`, and `RISK_ENGINE.md` for deep dives into each subsystem.

---

## Getting Started

### Prerequisites
- Docker & Docker Compose
- Node.js 18+ (for local frontend dev)
- Python 3.11+ (for local backend dev)

### Option 1 — Full stack via Docker Compose (recommended)

```bash
git clone https://github.com/dixha171/BeaconTrap-IITHBOI_Hackathon.git
cd BeaconTrap-IITHBOI_Hackathon
cp .env.example .env   # fill in your API keys (see below)
docker compose up --build
```

This spins up PostgreSQL, Redis, RabbitMQ, Neo4j, MinIO, the FastAPI backend, and the React frontend.

- Frontend → `http://localhost:3000`
- Backend API → `http://localhost:8000`
- RabbitMQ management → `http://localhost:15672`
- Neo4j browser → `http://localhost:7474`
- MinIO console → `http://localhost:9001`

### Option 2 — Run services locally

```bash
# Backend
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# Frontend
cd frontend
npm install
npm run dev
```

### Environment variables

Copy `.env.example` to `.env` and configure:

```
DATABASE_URL, REDIS_URL, RABBITMQ_URL
NEO4J_URL, NEO4J_USER, NEO4J_PASSWORD
MINIO_ENDPOINT, MINIO_ACCESS_KEY, MINIO_SECRET_KEY
SECRET_KEY, ACCESS_TOKEN_EXPIRE_MINUTES
VIRUSTOTAL_API_KEY, ABUSEIPDB_API_KEY   # threat intel enrichment
GEMINI_API_KEY, OPENAI_API_KEY          # LLM providers for the AI agents
```

---

## Smart Contract

Evidence hashes are anchored via [`contracts/EvidenceAnchor.sol`](./contracts/EvidenceAnchor.sol), a minimal Solidity contract exposing `anchor(caseId, evidenceHash)` and `verify(caseId, hashToCheck)`, deployed on **Ethereum Sepolia**. The frontend signs anchoring transactions client-side via MetaMask (ethers.js); the backend independently verifies on-chain state via Web3.py — so no single party can quietly rewrite the evidence trail.

---

## License

This project is licensed under the [MIT License](./LICENSE).
