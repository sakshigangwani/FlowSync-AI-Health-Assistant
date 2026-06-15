# FlowSync — Multi-Agent AI Health Assistant

> A production-style, multi-agent Retrieval-Augmented Generation (RAG) system for women's health support. Built to demonstrate end-to-end ownership of an applied-LLM product: data ingestion, vector retrieval, agent orchestration, API design, and a real-time web client.

FlowSync routes each user question to one of three **specialized LLM agents** (Medical, Symptom, Lifestyle) through a hybrid LLM + heuristic **router**, grounds every answer in a **Pinecone** vector store via **RAG**, and serves it over a Flask REST API and a streaming chat UI.

---

## Why this project is interesting (engineering summary)

This isn't a single-prompt chatbot wrapper. The design problem it solves is **specialization + routing under cost and latency constraints**, which surfaces the trade-offs that matter in real LLM systems:

- **Multi-agent decomposition.** Instead of one bloated system prompt, responsibilities are split across three agents with independently tuned decoding parameters (temperature/`max_tokens`) and retrieval depth (`k`). This improves answer quality per domain and keeps each prompt small and debuggable.
- **Hybrid routing with graceful degradation.** A low-temperature LLM classifier decides which agent handles a query, with a **deterministic keyword-scoring fallback** that takes over on any LLM error or timeout — the system never hard-fails on the classification step.
- **Retrieval/generation separation.** Embedding, indexing, and retrieval live behind a `MedicalRetriever` abstraction, so the vector backend or embedding model can change without touching agent code.
- **Cost-aware model selection.** Routing uses a cheap 50-token call; only the chosen specialist agent does the expensive generation. Local Sentence-Transformers embeddings avoid per-call embedding API costs.
- **Clear seams for scale.** Indexing is an offline batch job; serving is stateless; agents are independent objects. Each piece can be scaled or replaced in isolation.

---

## System Architecture

```
                          ┌──────────────────────────────┐
   User query ──────────▶ │       QuestionRouter         │
                          │  LLM classifier (temp=0.1)   │
                          │  └─ fallback: keyword scoring│
                          └───────────────┬──────────────┘
                                          │ route: medical | symptom | lifestyle
            ┌─────────────────────────────┼─────────────────────────────┐
            ▼                             ▼                              ▼
   ┌────────────────┐          ┌────────────────────┐          ┌──────────────────┐
   │  MedicalAgent  │          │   SymptomAgent     │          │  LifestyleAgent  │
   │  temp 0.4, k=3 │          │  temp 0.3, k=4     │          │  temp 0.5, k=3   │
   │                │◀─────────│  (can delegate to  │          │                  │
   └───────┬────────┘ collab.  │   MedicalAgent)    │          └────────┬─────────┘
           │                   └─────────┬──────────┘                   │
           └──────────────────┬──────────┴──────────────────────────────┘
                              ▼
                   ┌─────────────────────┐        ┌───────────────────────────┐
                   │  MedicalRetriever   │───────▶│   Pinecone (serverless)    │
                   │  (RAG, cosine, 384d)│        │   index: medical-chatbot   │
                   └──────────┬──────────┘        └───────────────────────────┘
                              ▲
                              │ embeddings (offline + query time)
                   ┌──────────┴──────────────────────┐
                   │ HuggingFace Sentence-Transformers│
                   │   all-MiniLM-L6-v2 (384-dim)     │
                   └──────────────────────────────────┘
```

### Request lifecycle
1. **Classify** — `QuestionRouter` calls a low-temperature LLM to label the query `MEDICAL | SYMPTOM | LIFESTYLE`; on failure it falls back to keyword scoring.
2. **Retrieve** — the selected agent queries `MedicalRetriever`, which embeds the query and runs cosine top-`k` similarity search against Pinecone.
3. **Generate** — the agent composes a domain-specific prompt with the retrieved context and calls the LLM with agent-tuned decoding params.
4. **Respond** — Flask returns plain text (`/get`, for the UI) or structured JSON with agent metadata (`/api/chat`).

---

## Key Design Decisions & Trade-offs

| Decision | Rationale | Trade-off considered |
|---|---|---|
| **Three specialized agents** vs. one generalist | Smaller, auditable prompts; per-domain tuning; better grounding | More moving parts and a routing step to maintain |
| **Hybrid LLM + keyword router** | LLM gives semantic accuracy; keyword fallback guarantees availability and zero-cost degradation | Keyword fallback is coarser; accepted because it only triggers on LLM failure |
| **Local Sentence-Transformers** (`all-MiniLM-L6-v2`, 384-dim) | No per-query embedding API cost; fast; strong quality-for-size | Larger models could improve recall at higher compute cost |
| **Pinecone serverless** (cosine, `k=3–4`) | Managed, low-ops vector search that scales independently of the app | Vendor coupling, mitigated by the `MedicalRetriever` abstraction |
| **Per-agent decoding params** | Symptom answers stay conservative (`temp=0.3`); lifestyle is more open (`temp=0.5`) | Requires deliberate tuning rather than one global default |
| **Offline indexing job** (`store_index.py`) | Keeps the serving path fast and stateless; re-indexing is decoupled from request handling | Index can go stale; refreshed by re-running the batch job |
| **`RecursiveCharacterTextSplitter`** (500 chars, 20 overlap) | Balances retrieval granularity against context fragmentation | Smaller chunks raise recall but increase index size |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3 |
| LLM orchestration | LangChain (`langchain`, `langchain-openai`, `langchain-pinecone`, `langchain-huggingface`) |
| Generation model | OpenAI LLM (via `langchain-openai`) |
| Embeddings | HuggingFace Sentence-Transformers — `all-MiniLM-L6-v2` (384-dim) |
| Vector store | Pinecone (serverless, AWS `us-east-1`, cosine similarity) |
| Document processing | `pypdf`, `RecursiveCharacterTextSplitter` |
| Backend | Flask (REST + server-rendered chat) |
| Frontend | HTML/CSS/JS chat UI with typing & retrieval animations |
| Config | `python-dotenv` |

---

## Project Structure

```text
flowsync-ai-health-assistant/
├── agents/                  # Specialized LLM agents (independent, swappable)
│   ├── medical_agent.py     #   conditions, hormones, pathophysiology
│   ├── symptom_agent.py     #   symptom interpretation (+ delegation to MedicalAgent)
│   └── lifestyle_agent.py   #   nutrition, exercise, sleep, stress
├── orchestrator/
│   └── router.py            # Hybrid LLM + keyword query router
├── retrieval/
│   └── retriever.py         # MedicalRetriever — RAG abstraction over Pinecone
├── src/
│   ├── helper.py            # Loaders, chunking, embeddings, upsert
│   └── prompt.py            # Prompt templates
├── templates/chat.html      # Chat UI
├── static/styles.css
├── Data/                    # Source medical documents (PDF)
├── store_index.py           # Offline indexing / embedding pipeline
├── app.py                   # Flask app + REST API
├── check_setup.py           # Environment/config verification
├── quick_test.py            # Smoke tests
├── test_agents.py           # Agent-level tests
├── examples.py              # Example query flows
└── requirements.txt
```

---

## Getting Started

### 1. Clone & install
```bash
git clone https://github.com/sakshigangwani/flowsync-ai-health-assistant.git
cd flowsync-ai-health-assistant

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

### 2. Configure environment
Create a `.env` in the project root:
```env
OPENAI_API_KEY=your_openai_api_key
PINECONE_API_KEY=your_pinecone_api_key
```

### 3. Verify setup
```bash
python check_setup.py
```

### 4. Build the vector index (offline, one-time / on data change)
```bash
python store_index.py
```
Loads PDFs from `Data/` → splits into 500-char chunks → embeds with `all-MiniLM-L6-v2` → creates the `medical-chatbot` index (384-dim, cosine, serverless) if missing → upserts vectors.

### 5. Run the app
```bash
python app.py
# http://127.0.0.1:8080
```

---

## API Reference

### `POST /api/chat`
Structured chat endpoint with agent metadata.

**Request**
```json
{ "message": "What is PCOS?" }
```
**Response**
```json
{
  "answer": "...",
  "agent_type": "medical",
  "agent_name": "Medical Knowledge"
}
```

### `GET /api/agents`
Returns the catalog of available agents and their declared expertise.

### `POST /get`
Form-encoded endpoint backing the web UI; returns plain-text answer.

---

## Testing

```bash
python quick_test.py     # fast smoke test of the end-to-end path
python test_agents.py    # per-agent behavior
python examples.py       # representative query flows across all agents
```

---

## Example Queries
- **Medical** → "What is PCOS?" · "How does insulin resistance work?"
- **Symptom** → "I have acne and irregular periods" · "What causes fatigue?"
- **Lifestyle** → "What diet helps balance hormones?" · "Best exercises for weight loss?"

---
