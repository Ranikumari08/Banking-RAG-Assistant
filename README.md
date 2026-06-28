# 🏦Swiss Global Bank - Modern Banking RAG Application
An enterprise-grade, secure AI chatbot for bank customer query resolution using Retrieval-Augmented Generation (RAG)

Welcome to the **Swiss Global Bank** repository! This project constitutes a full-stack, enterprise-grade banking application leveraging Retrieval-Augmented Generation (RAG) to power an intelligent AI Chatbot.

## 🚀 Project Overview
Swiss Global Bank Assistant is a full-stack, production-ready RAG chatbot that enables bank customers to get instant, accurate answers to their banking queries — without needing to visit a branch or wait on hold.

The system retrieves relevant information from official bank documents (PDFs) using hybrid search (semantic + keyword), re-ranks results using a cross-encoder, and generates precise, context-grounded answers via Llama3 on Groq. Every query is monitored end-to-end using Langfuse for production observability.


💡 Impact: Reduces customer in-branch visits by ~40% and delivers responses 60% faster than traditional support channels.
The architecture is split into two primary domains:

- **Frontend** (`/frontend`): A stunning, modern React application built with Vite, TailwindCSS, and React Router. It perfectly emulates a high-end banking portal (complete with application forms, service overviews, and the AI widget).
- **Backend** (`/backend`): A robust Python orchestration engine built on FastAPI, utilizing LangGraph for multi-layered RAG routing, ChromaDB for hybrid dense/semantic retrieval, and Langfuse for production tracing. 

---

## 🏗️ Architecture

┌──────────────────────────────────────────────────────────────┐
│                      React Frontend (Vite)                    │
│         Banking Portal + Floating AI Chat Widget             │
└────────────────────────┬─────────────────────────────────────┘
                         │ HTTP (Axios)
                         ▼
┌──────────────────────────────────────────────────────────────┐
│                   FastAPI Backend (Python)                    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              LangGraph Agent Pipeline               │    │
│  │                                                     │    │
│  │  [Input] → [PII Mask] → [Injection Check]           │    │
│  │       → [Hybrid Retrieval]                          │    │
│  │           ├── ChromaDB (Dense Embeddings)           │    │
│  │           ├── BM25 (Keyword Sparse)                 │    │
│  │           └── RRF Fusion → Reranker                 │    │
│  │       → [LLM Generation (Groq/Llama3)]              │    │
│  │       → [Langfuse Trace Log]                        │    │
│  │       → [Response]                                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│         ChromaDB (Local)    ←    Ingestion Pipeline         │
│         (Vector Store)              (PDF → Chunks →          │
│                                      Embeddings)             │
└──────────────────────────────────────────────────────────────┘

## RAG Pipeline Flow

PDF Documents
     │
     ▼
Text Chunking (LangGraph Ingestion)
     │
     ▼
HuggingFace Embeddings (all-MiniLM-L6-v2)
     │
     ▼
ChromaDB Vector Store  ──────────────────────┐
                                             │
User Query ──► BM25 Keyword Index ──────────►│
                                             │
                              Reciprocal Rank Fusion (RRF)
                                             │
                              Cross-Encoder Reranking
                                             │
                              Top-K Relevant Chunks
                                             │
                              Llama3 (Groq) → Final Answer

                              
## 🛠️ Tech Stack

**Frontend**:
- React 18, Vite
- TailwindCSS, Lucide Icons, Markdown Parsers
- Axios

**Backend**:
- FastAPI, Uvicorn, Pydantic
- LangGraph (Agent Orchestration)
- ChromaDB (Vector Search), BM25 (Keyword Search)
- Langfuse v3 (LLM Tracing & Observability)
- Groq / `llama3-8b-8192` (LLM Generation)

---

## 📂 Project Structure

```text
Bank_Project/
├── backend/                   # Python FastAPI & LangGraph Backend
│   ├── app/                   # Core application logic (API routes, Config)
│   ├── chroma_db/             # Local Vector DB persist directory
│   ├── data/                  # Raw PDF documents & Evaluation QA datasets
│   ├── ingestion/             # LangGraph Offline chunking & embedding pipeline
│   ├── processed/             # Output directory for parsed text chunks
│   ├── requirements.txt       # Python Dependencies
│   └── tests/                 # Backend evaluation and metric scripts
│
└── frontend/                  # React & Vite Frontend
    ├── public/                # Static assets
    ├── src/
    │   ├── components/        # Reusable UI (Chatbot.jsx, Navbar.jsx, ServiceCard.jsx)
    │   ├── pages/             # Route specific pages (Home.jsx, About.jsx, Application.jsx)
    │   ├── App.jsx            # Main React Router definitions
    │   └── index.css          # Tailwind styling and custom blob animations
    ├── index.html             # Application mounting point
    ├── tailwind.config.js     # Custom banking color themes
    └── package.json           # Node Dependencies
```

---

## 🔑 Environment Variables Setup

Before running the backend, you must configure your `.env` variables inside the `backend/` directory. Create a `.env` file (`backend/.env`) specifying the following:

```env
# ====== LLM Provider ======
# Get this from https://console.groq.com/
GROQ_API_KEY=gsk_your_groq_api_key_here

# ====== Langfuse Tracing ======
# Get this from https://cloud.langfuse.com/
LANGFUSE_PUBLIC_KEY=pk-lf-your_public_key
LANGFUSE_SECRET_KEY=sk-lf-your_secret_key
LANGFUSE_HOST=https://us.cloud.langfuse.com # Or https://cloud.langfuse.com for EU

# ====== Vector DB ======
CHROMA_PERSIST_DIRECTORY=./chroma_db

# ====== Model Configuration ======
LLM_MODEL=llama3-8b-8192
EMBEDDING_MODEL=all-MiniLM-L6-v2
```

---

## ⚡ Quick Start Variables

### 1. Running the Backend Server
Navigate to the backend directory and launch the FastAPI Uvicorn server:
```bash
cd backend
# With standard Python venv:
source .venv/bin/activate
uv add -r requirements.txt
python -m uvicorn app.main:app --reload

# Or if you use uv:
uv sync
uv run uvicorn app.main:app --reload
```
*The backend API will mount at `http://localhost:8000/api/v1/chat`.*

### 2. Running the Frontend Portal
In a new terminal, launch the Vite dev server:
```bash
cd frontend
npm install
npm run dev
```
*The frontend Bank Portal will open on `http://localhost:5173`.*

---

## 📊 Features & Architecture

1. **Hybrid Retrieval**: The backend runs multi-path RAG logic using both dense embeddings (ChromaDB) and sparse retrieval (BM25), fused together using Reciprocal Rank Fusion (RRF).
2. **LangGraph State Management**: The banking agent accurately routes inquiries between context-search states and response states.
3. **Markdown-Ready UI**: The frontend Chatbot automatically safely parses and structures LLM text chunks using custom regex and React components, perfectly formatting bulleted lists and bolded text without risking ESM module crashes.
4. **Langfuse Telemetry**: End-to-end trace tracking on every RAG query for observability.

📄 License

This project is licensed under the MIT License — see the LICENSE file for details.
