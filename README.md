# 🩺 FlowSync AI Health Assistant

### Multi-Agent AI Health Assistant for Women's Health Support using Retrieval-Augmented Generation (RAG)

FlowSync AI Health Assistant is a **multi-agent conversational AI system** designed to provide guidance on women's health topics such as hormonal health, symptoms, wellness, and general medical knowledge.

The assistant combines **specialized AI agents** with **Retrieval-Augmented Generation (RAG)** to provide grounded, contextual responses using medical knowledge sources.

The system integrates:

- OpenAI LLM responses
- Pinecone vector database
- HuggingFace embeddings
- Flask backend
- Chat-based web interface

---

# 📌 Overview

The system intelligently routes each user query to the most suitable **specialist AI agent**.

### Agents

| Agent | Responsibility |
|------|------|
| **MedicalAgent** | Explains medical conditions, concepts, and treatments |
| **SymptomAgent** | Interprets symptoms and suggests possible next steps |
| **LifestyleAgent** | Provides guidance on wellness, nutrition, sleep, stress, and habits |

Each agent can retrieve contextual information from the **vector database** to generate accurate responses.

---

# 🚀 Key Features

- Multi-agent AI architecture
- Intelligent query routing
- Retrieval-Augmented Generation (RAG)
- Pinecone vector database integration
- Document-based medical knowledge
- Real-time conversational chat UI
- Typing animation for AI responses
- Thinking animation during retrieval
- Modular and scalable Python codebase
- REST API endpoints for chat and system checks

---

# 🧠 System Architecture

```
User Query
    │
    ▼
Question Router
    │
    ▼
Specialist Agent
    │
    ▼
Retriever (Pinecone)
    │
    ▼
LLM Response
    │
    ▼
Chat UI / API
```

### Core Modules

| Module | Description |
|------|------|
| `agents/` | Specialized AI agents |
| `orchestrator/router.py` | Agent routing logic |
| `retrieval/retriever.py` | Pinecone retriever setup |
| `app.py` | Flask backend API |
| `store_index.py` | Document embedding pipeline |

---

# 📂 Project Structure

```text
flowsync-ai-health-assistant/
│
├── agents/
│   ├── medical_agent.py
│   ├── symptom_agent.py
│   └── lifestyle_agent.py
│
├── orchestrator/
│   └── router.py
│
├── retrieval/
│   └── retriever.py
│
├── src/
│   ├── helper.py
│   └── prompt.py
│
├── templates/
│   └── chat.html
│
├── static/
│   └── styles.css
│
├── Data/
│
├── app.py
├── store_index.py
├── check_setup.py
├── quick_test.py
├── test_agents.py
├── examples.py
├── requirements.txt
└── README.md
```

---

# ⚙️ Setup

## 1. Clone the Repository

```bash
git clone https://github.com/sakshigangwani/flowsync-ai-health-assistant.git
cd flowsync-ai-health-assistant
```

---

## 2. Create Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate
```

For Windows:

```bash
.venv\Scripts\activate
```

---

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

## 4. Environment Variables

Create a `.env` file in the project root.

```env
OPENAI_API_KEY=your_openai_api_key
PINECONE_API_KEY=your_pinecone_api_key
```

---

## 5. Verify Setup

```bash
python check_setup.py
```

---

# ▶️ Running the Project

## Step 1 — Build Vector Index

Convert documents into embeddings and store them in Pinecone.

```bash
python store_index.py
```

This step:

- Loads medical documents
- Splits text into chunks
- Generates embeddings
- Stores vectors in Pinecone

---

## Step 2 — Start the Application

```bash
python app.py
```

Open your browser:

```
http://127.0.0.1:8080
```

---

# 💬 Chat Interface

The web interface includes:

- FlowSync AI health assistant chat
- Typing animation for responses
- Thinking animation while retrieving knowledge
- Clean responsive UI

Example questions:

- What is PCOS?
- Why are my periods irregular?
- What causes hormonal acne?
- What diet helps balance hormones?

---

# 🌐 API Endpoints

## Chat

**POST**

```
/api/chat
```

### Example Request

```json
{
  "message": "What is PCOS?"
}
```

### Example Response

```json
{
  "answer": "...",
  "agent_type": "medical",
  "agent_name": "Medical Knowledge Agent"
}
```

---

## List Agents

**GET**

```
/api/agents
```

Returns available agents and descriptions.

---

# 🧪 Testing

Run various tests to verify system behavior.

### Quick Test

```bash
python quick_test.py
```

### Agent Tests

```bash
python test_agents.py
```

### Example Flows

```bash
python examples.py
```

---

# 🔮 Future Improvements

- Multi-document knowledge base
- Conversation memory for personalized guidance
- Voice-based health assistant
- Advanced symptom analysis
- Integration with FlowSync mobile app
- Personalized health insights

---
