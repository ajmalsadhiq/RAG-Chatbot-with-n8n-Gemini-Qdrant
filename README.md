#  AI-Powered RAG Chatbot — n8n · Gemini · Qdrant

A full end-to-end **Retrieval-Augmented Generation (RAG)** pipeline built from scratch — including designing the knowledge base, automating document ingestion, and deploying an AI chatbot that answers questions grounded strictly in company documents.

---

## What Makes This Different

Most RAG demos use random PDFs downloaded from the internet.

For this project, I **designed the entire knowledge base myself** — a fictional company called **NovaTech Solutions** — and authored five realistic, interconnected HR and IT policy documents totalling 15+ pages of structured business content.

This was intentional: to properly test a RAG system, you need documents with:
- **Overlapping topics** (e.g. leave policy mentioned in both the handbook and the leave policy)
- **Specific numbers and thresholds** that a keyword search would miss but vector search should retrieve
- **Tables, tiered data, and conditional logic** (e.g. salary bands, travel allowance tiers, leave accrual rules)
- **Realistic ambiguity** to test whether the model retrieves the right chunk

---

## 📄 Knowledge Base — Designed from Scratch

All documents were authored specifically for this project to stress-test the RAG pipeline:

| Document | Contents | Pages |
|---|---|---|
| `Employee_Handbook.pdf` | Company overview, working hours, remote work policy, code of conduct, performance management, probation | 3 |
| `Leave_Policy.pdf` | Annual, sick, maternity, paternity, bereavement, study leave entitlements with notice periods and conditions | 3 |
| `Benefits_and_Compensation_Policy.pdf` | Salary review bands, bonus multipliers, health insurance limits, L&D budget, ESOP vesting, wellness perks | 3 |
| `IT_Security_Policy.pdf` | Password policy, device encryption, data classification tiers, acceptable use, incident reporting | 3 |
| `Travel_and_Expenses_Policy.pdf` | Flight class rules, daily allowances (Tier 1/2/GCC), hotel limits, meal reimbursement, approval thresholds | 3 |

### Why This Matters for the RAG System

The documents were structured to create **real retrieval challenges**:

- "What is the salary increase for an Exceeds Expectations rating?" → requires finding a table inside the Benefits doc
- "Can I work from home during probation?" → answer requires combining info from both the Handbook and Remote Work section
- "What happens if I lose my company laptop?" → buried in the IT Security Policy under Device Security
- "How many days can I carry over from annual leave?" → specific number (10 days) inside a longer policy paragraph

These are questions a naive keyword search would fail on but a well-built RAG system handles correctly.

---

## 🏗️ System Architecture

### Workflow 1 — Document Ingestion Pipeline

```
Google Drive (PDF Upload Trigger)
        ↓
  Download File
        ↓
  Extract Text from PDF
        ↓
  Chunk & Embed (Gemini text-embedding-004)
        ↓
  Store Vectors in Qdrant
```

### Workflow 2 — RAG Chatbot

```
User Question (Chat Interface)
        ↓
     AI Agent
        ↓
  Qdrant Vector Search  ←── finds semantically relevant chunks
        ↓
  Gemini 1.5 Flash  ←── generates answer grounded in retrieved context
        ↓
     Response
```

---

## 💬 Example Q&A Output

These are real answers the chatbot produced using only the documents above:

> **Q:** How many employees does NovaTech have?
> **A:** NovaTech Solutions employs over 1,200 professionals across offices in the UAE, UK, Singapore, and the United States.

> **Q:** What flight class am I entitled to for a 10-hour flight?
> **A:** For flights over 9 hours, employees at Director level and below are entitled to Business Class.

> **Q:** What is the L&D budget per employee?
> **A:** Each employee receives OMR 750 (approximately USD 1,950) per calendar year for learning and development expenses.

> **Q:** Can I carry over unused annual leave?
> **A:** Yes, a maximum of 10 days may be carried over to the next calendar year. Any unused days above this threshold are forfeited.

---

## 🛠️ Tech Stack

| Component | Tool |
|---|---|
| Workflow Automation | n8n |
| LLM | Google Gemini 1.5 Flash |
| Embeddings | Gemini text-embedding-004 |
| Vector Database | Qdrant |
| Document Trigger | Google Drive |

---

## 📁 Repository Structure

```
├── workflow-ingestion.json       # n8n: PDF → Embed → Qdrant
├── workflow-chatbot.json         # n8n: Chat → Retrieve → Generate
├── documents/
│   ├── Employee_Handbook.pdf
│   ├── Leave_Policy.pdf
│   ├── Benefits_and_Compensation_Policy.pdf
│   ├── IT_Security_Policy.pdf
│   └── Travel_and_Expenses_Policy.pdf
├── screenshots/
│   ├── ingestion-workflow.png
│   ├── chatbot-workflow.png
│   └── demo-answer.png
└── README.md
```

---

## 🚀 How to Reproduce This

### Prerequisites
- [n8n](https://n8n.io/) account (cloud trial or self-hosted via Docker)
- [Qdrant](https://qdrant.tech/) account (free tier works)
- Google Cloud project with Gemini API enabled
- Google Drive folder for document uploads

### Steps

1. **Clone this repo**
```bash
git clone https://github.com/YOUR_USERNAME/RAG-Chatbot-n8n-Gemini-Qdrant
```

2. **Create a Qdrant collection**
   - Collection name: `documents`
   - Vector size: `768` (Gemini embedding dimensions)
   - Distance: `Cosine`

3. **Import workflows into n8n**
   - `Workflows` → `Import from File`
   - Import ingestion workflow first, then chatbot workflow

4. **Add credentials in n8n**
   - Gemini API key
   - Qdrant API URL + key
   - Google Drive OAuth

5. **Upload the PDFs from `/documents/` to your Google Drive folder**
   — the ingestion workflow triggers automatically

6. **Open the Chat node in Workflow 2 and start asking questions**

---

## 🧩 Key Concepts Demonstrated

- **RAG architecture** — grounding LLM responses in a private knowledge base
- **Vector embeddings** — representing text semantically using Gemini's embedding model
- **Cosine similarity search** — retrieving the most contextually relevant document chunks
- **AI Agent design** — giving an LLM access to a retrieval tool at inference time
- **Event-driven pipelines** — Google Drive trigger initiates the full ingestion flow
- **Knowledge base design** — structuring documents to challenge and validate retrieval quality

---

## 🔮 Possible Extensions

- Add chunk overlap for better boundary handling
- Support additional file types (Word, CSV, web scraping)
- Build a React frontend connected via n8n webhook
- Add conversation memory to the AI agent
- Deploy Qdrant self-hosted for production use
