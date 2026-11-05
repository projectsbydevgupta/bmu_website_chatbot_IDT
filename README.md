# BMU Website Chatbot 2.0: Agentic RAG System

[![Production Status](https://img.shields.io/badge/Production-Live%20on%20AWS%20EC2-22c55e?style=for-the-badge&logo=amazonaws&logoColor=white)](http://13.51.178.39:8000/dashboard/)
[![CI/CD Quality Gate](https://img.shields.io/badge/CI%2FCD%20Gate-20%2F20%20Tests%20Passing-38bdf8?style=for-the-badge&logo=githubactions&logoColor=white)](https://github.com/projectsbydevgupta/bmu_website_chatbot_IDT/actions)
[![LangGraph](https://img.shields.io/badge/LangGraph-Tri--Agent%20RAG-f59e0b?style=for-the-badge&logo=langchain&logoColor=white)](https://langchain-ai.github.io/langgraph/)
[![Embeddings](https://img.shields.io/badge/BGE--Large-1%2C024--dim%20Dense%20Vectors-818cf8?style=for-the-badge)](https://huggingface.co/BAAI/bge-large-en-v1.5)
[![Arize Phoenix](https://img.shields.io/badge/Arize%20Phoenix-Observability-ec4899?style=for-the-badge)](http://13.51.178.39:6006/)

An enterprise-grade, institutional Agentic Retrieval-Augmented Generation (RAG) assistant for **BML Munjal University (BMU)**. Built on a decoupled multi-microservice architecture with an autonomous 3-agent LangGraph workflow, dual-stage JEV Guardrails, BGE-1024 dense vector retrieval over ChromaDB Cloud, full OpenTelemetry observability via Arize Phoenix, and automated GitHub Actions CI/CD.

---

## 🌐 Live Production Endpoints (AWS EC2)

| Service | Endpoint | Status | Description |
| :--- | :--- | :--- | :--- |
| **BMU Chatbot & Dashboard** | [http://13.51.178.39:8000/dashboard/](http://13.51.178.39:8000/dashboard/) | 🟢 **Live** | Web chat interface & Obsidian Telemetry dashboard |
| **API Docs (Swagger UI)** | [http://13.51.178.39:8000/docs](http://13.51.178.39:8000/docs) | 🟢 **Live** | OpenAPI endpoints specification (`/ask`, `/health`) |
| **BGE-1024 Retrieval Service** | [http://13.51.178.39:8001/health](http://13.51.178.39:8001/health) | 🟢 **Live** | Isolated BGE-Large 1,024-d semantic embedding server |
| **Arize Phoenix Observability** | [http://13.51.178.39:6006/](http://13.51.178.39:6006/) | 🟢 **Live** | Real-time OpenTelemetry trace waterfall & latency logs |

---

## 🏛️ System Architecture

```text
                      [ Student / User Browser ]
                                  │
                                  ▼
           [ AWS EC2 Cloud Instance: 13.51.178.39 (Ubuntu 24.04 LTS) ]
 ┌────────────────────────────────────────────────────────────────────────┐
 │                                                                        │
 │  PORT 8000: FastAPI Orchestrator (semantic.py)                         │
 │  ├── Frontend Chat UI & Obsidian Telemetry Analytics Dashboard         │
 │  ├── Stage 1 & Stage 2 JEV Guardrails (Pre & Post-Inference Checks)    │
 │  └── LangGraph Tri-Agent State Machine                                │
 │      ├── Agent 1: Query Specialist (Intent & Keyword Refinement)       │
 │      ├── Agent 2: Context Auditor (Evidence Sufficiency Verification)  │
 │      └── Agent 3: Response Synthesizer (Grounded Answer Generation)    │
 │                 │                              │                       │
 │    (HTTP Calls) │                              │ (OpenTelemetry OTLP)  │
 │                 ▼                              ▼                       │
 │  PORT 8001: Retrieval Microservice      PORT 6006: Arize Phoenix       │
 │  ├── BAAI/bge-large-en-v1.5             ├── OTLP Span Collection       │
 │  ├── 1,024-dim Vector Encoding          ├── Per-Agent Latency (p50/p95)│
 │  └── Hybrid Rank (0.68 Cos + 0.32 BM25) └── Token Cost Tracking       │
 │                 │                                                      │
 └─────────────────┼──────────────────────────────────────────────────────┘
                   ▼
      [ ChromaDB Cloud Collection ]
          "bmu_bge_knowledge"
     (1,258 Verified BMU Documents)
```

---

## 🚀 Key Technical Highlights

### 1. High-Density BGE-1024 Retrieval Engine
* Migrated from 384-d MiniLM to **`BAAI/bge-large-en-v1.5` (1,024 dimensions)**.
* **+70% higher semantic retrieval precision** on university terminology (programs, specializations, cut-offs).
* Hybrid scoring formula: $\text{Score} = 0.68 \times \text{Cosine Similarity} + 0.32 \times \text{BM25 Keyword Overlap}$.
* Zero external API rate limits (100% self-hosted inference on Port 8001).

### 2. Autonomous Tri-Agent LangGraph State Machine
* **Agent 1 (Query Specialist):** Reformulates casual student questions into precise academic search queries.
* **Tool (Hybrid Retrieval):** Searches ChromaDB Cloud and retrieves candidate documents with metadata and verified URLs.
* **Agent 2 (Context Auditor):** Evaluates candidate proof before generation. Employs a self-correcting fallback loop: returns `SUFFICIENT`, `TRY_FALLBACK` (evaluates next candidate), or `NOT_FOUND`.
* **Agent 3 (Response Synthesizer):** Drafts concise answers strictly bounded by the audited facts.

### 3. Dual-Stage JEV Guardrails (Zero-Trust Security)
* **Pre-Inference Input Guardrail:** Scans 12 prompt injection patterns (jailbreaks, instruction override) and domain scope in sub-milliseconds; blocks attacks scoring $\ge 0.70$.
* **Post-Inference Output Guardrail:** Measures n-gram grounding overlap against retrieved BMU context ($\ge 0.40$ threshold); appends an official portal verification disclaimer if weakly grounded and suppresses toxic outputs.

### 4. Dual LLM Provider Architecture
* **Primary (Cloud):** Groq LPU acceleration (`openai/gpt-oss-120b` & `openai/gpt-oss-20b`) delivering ~400 tokens/sec.
* **Fallback (Local):** Ollama (`llama3.2:3b` on Port 11434) providing 100% local data privacy and offline failover.

---

## 🛡️ CI/CD Pipeline & Automated Quality Gate

Our deployment pipeline runs on **GitHub Actions** (`.github/workflows/ci-cd.yml`). Every commit triggers an automated **7-Suite Quality Gate** containing **20 rigorous unit, integration, and security tests**:

1. **Suite 1: Environment Integrity** — Validates configuration keys, ports, and hosts.
2. **Suite 2: Syntax & Imports** — Ensures all microservice dependencies load cleanly.
3. **Suite 3: JEV Input Guardrail** — Simulates adversarial prompt injection and out-of-scope attacks.
4. **Suite 4: JEV Output Grounding** — Enforces factual grounding thresholds and blocks toxic outputs.
5. **Suite 5: BGE-1024 Vector Dimensions** — Verifies vector math and asserts embedding dimension == 1,024.
6. **Suite 6: Fast-Fail Health Endpoints** — Confirms keep-alive health endpoints respond HTTP 200.
7. **Suite 7: LLM-as-a-Judge Quality Gate** — An independent LLM evaluates factual grounding and relevance.

> **Pre-Deployment Block:** If any single test fails (e.g. 19/20), deployment is **instantly blocked**, protecting the live production AWS server from breaking regressions.

---

## 📁 Repository Structure

```text
BMU_WEBSITE_CHATBOT-main/
├── .github/workflows/
│   ├── ci-cd.yml                 # Automated 20-test CI/CD quality gate & deploy
│   └── keep-alive-cron.yml       # 5-minute health ping verification
├── frontend/                     # Web Chat UI & Obsidian Telemetry Dashboard
│   ├── index.html                # Main conversational chat interface
│   ├── evaluation.html           # 100-Question benchmark analytics dashboard
│   ├── app.js                    # Chat state management & streaming handler
│   └── evaluation.js             # Metrics visualization & export logic
├── evaluation_outputs/           # Benchmark datasets & RAGAS evaluation runs
├── bmu_rag_graph.py              # LangGraph 3-Agent state machine
├── semantic.py                   # FastAPI Orchestrator (Port 8000)
├── retrieval_service.py          # BGE-1024 Retrieval microservice (Port 8001)
├── phoenix_service.py            # Arize Phoenix tracing service (Port 6006)
├── phoenix_tracer.py             # OpenTelemetry batch span instrumentation
├── jev_guardrails.py             # Dual-stage JEV security & grounding engine
├── evaluate_metrics.py           # RAGAS quality evaluation & scoring formulas
├── benchmark_100_questions.py    # 100-Question benchmark dataset (20 categories)
├── test_ci_suite.py              # 7-Suite / 20-Test automated CI quality gate
├── start_services.py             # Stack supervisor with process auto-recovery
├── Dockerfile                    # Multi-port containerization specification
├── requirements.txt              # Production Python dependencies
└── run.bat                       # Local one-click stack launcher
```

---

## ⚡ Quickstart (Local Development)

### 1. Prerequisites
* Python 3.10 or 3.11
* Git

### 2. Clone & Install
```bash
git clone https://github.com/projectsbydevgupta/bmu_website_chatbot_IDT.git
cd bmu_website_chatbot_IDT
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 3. Configure Environment
Copy `.env.example` to `.env` and populate your API credentials:
```bash
cp .env.example .env
```
Key variables:
```ini
GROQ_API_KEY=your_groq_api_key
CHROMA_API_KEY=your_chroma_api_key
CHROMA_TENANT=your_tenant_id
CHROMA_DATABASE=your_database_name
CHROMA_COLLECTION_NAME=bmu_bge_knowledge
EMBEDDING_MODEL_NAME=BAAI/bge-large-en-v1.5
EMBEDDING_DIMENSION=1024
```

### 4. Run Locally
**Option A: One-Click Windows Launcher**
```bat
run.bat
```

**Option B: Python Process Supervisor**
```bash
python start_services.py
```

Open [http://localhost:8000/dashboard/](http://localhost:8000/dashboard/) to interact with the chatbot.

---

## 🏛️ Institutional Attribution

Developed as an academic capstone initiative for **BML Munjal University** (School of Engineering & Technology).
