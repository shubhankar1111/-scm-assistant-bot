# SCM Assistant Bot

RAG-powered supply chain management chatbot built with Flowise Cloud.
Answers questions about a 116-supplier network using purchase order data and the BQBYTE Technologies Supplier Governance & Compliance Policy.

---

## Links

| Resource | URL |
|---|---|
| **Public Chatbot** | https://cloud.flowiseai.com/chatbot/6dd7e0d4-8716-4113-9e09-5f8bb4b9f6a4 |
| **GitHub Repo** | `https://github.com/shubhankar1111/scm-assistant-bot` |

---

## Stack

| Component | Choice | Reason |
|---|---|---|
| **Platform** | Flowise Cloud | Managed hosting, no infra required |
| **LLM** | Groq `llama-3.3-70b-versatile` | Fast, free, strong instruction-following |
| **Embeddings** | HuggingFace `sentence-transformers/all-MiniLM-L6-v2` | Free, good semantic search quality |
| **Vector Store** | Pinecone (`scmstore`) | Managed cloud vector DB, free tier |
| **Retrieval** | Conversational Retrieval QA Chain | Maintains context across follow-up questions |

---

## Data Files

| File | Description |
|---|---|
| `supplier_performance_data.csv` | 2,000 purchase orders · 116 suppliers · 27 columns (OTD rate, defect rate, compliance score, risk level, disruption flags, PO value, etc.) |
| `SupplyChain_Governance_Policy_v3.2.pdf` | 10-section supplier governance policy — tier thresholds, SLAs, penalties, audit rules, disruption response procedures |

---

## Chunk Configurations Tested

### Config A — Smaller Chunks (used in final build)

| Parameter | Value |
|---|---|
| Chunk Size | 500 tokens |
| Chunk Overlap | 50 tokens |
| Text Splitter | Recursive Character |
| **CSV chunk count** | 4,000 chunks |
| **PDF chunk count** | 35 chunks |

**Observations:** Higher precision on targeted lookups (e.g., specific supplier names, exact policy section numbers). Occasionally required the retriever to stitch 2–3 chunks to answer multi-part questions.

---

### Config B — Larger Chunks

| Parameter | Value |
|---|---|
| Chunk Size | 1,200 tokens |
| Chunk Overlap | 150 tokens |
| Text Splitter | Recursive Character |
| **CSV chunk count** | ~2,000 chunks |
| **PDF chunk count** | ~15 chunks |

**Observations:** Better for policy questions requiring broader context (e.g., reading an entire policy section in one chunk). Slightly noisier for CSV lookups — chunks contained unrelated supplier rows, which occasionally caused the LLM to conflate suppliers.

**Verdict:** Config A (500/50) produced more accurate answers for the 5 validation questions, especially Q1, Q2, and Q4 which required matching multiple supplier attributes simultaneously.

---

## Sample Q&A (Verbatim Chatbot Output)

### Q1 — Tier-3 suppliers with active disruption flag

**Question:** Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?

**Answer:**
> 11 Tier-3 suppliers: Dravex Components India, Plataforma Metales SA, Maghreb Castworks, Helios Pack Greece, Cerromax Mineria, Orinoco Pack SAPI, Quetzal Textiles, Sibertek Molding, Archipelago PCB Corp, Varna Electronics EAD, Deltaforge Vietnam. All are High Risk with an active flag → Level 3 Activate per Policy §9 (CPO escalation + alternate supplier at minimum 40% volume).

---

### Q2 — Volume Rebate Program qualifiers

**Question:** Which suppliers qualify for the annual Volume Rebate Program and how many are there?

**Answer:**
> 19 suppliers qualify: Borealis Composites, Crestline Chemical Supply, Fenwick Alloy Solutions, Hanguk Circuit Works, Hokkaido Alloy Tech, Krauss-Polymex GmbH, Lakeshore Components, Lumivex Semiconductor NL, Maplewood Polymer Corp, Norbec Alloy Works, Nordloom Finland Oy, Orrentek Precision Mfg, Ostwind Composites AG, PrecisionForge Taiyuan, Solveig Eco Packaging, Straits Packaging Hub, Tasman Circuit Boards, Toreval Electronics, Valdoro Special Alloys. Criteria (Policy §4.2): Tier-1 + OTD ≥ 93% + Defect < 0.5% + Sustainability Score ≥ 85.

---

### Q3 — Region with highest PO value / concentration breach

**Question:** Which region has the highest total PO value, and does it breach the concentration limit?

**Answer:**
> EMEA at $193,987,179.91 — approximately 48.5% of total spend ($399,563,494.10). This breaches the 45% regional concentration cap (Policy §5.3), requiring a Diversification Plan within 60 days.

---

### Q4 — Supplier Watch List status

**Question:** Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?

**Answer:**
> 11 suppliers (Compliance Score < 60): Deltaforge Vietnam, Maghreb Castworks, Helios Pack Greece, Cerromax Mineria, Orinoco Pack SAPI, Varna Electronics EAD, Quetzal Textiles, Plataforma Metales SA, Archipelago PCB Corp, Dravex Components India, Sibertek Molding. SWL restricts new PO issuance to 20% of prior quarter volume (Policy §3.4).

---

### Q5 — Highest defect rate by product category

**Question:** Which product category has the highest average defect rate and does it exceed the Tier-2 limit?

**Answer:**
> Mechanical Components — average 2.12% across 360 POs. Below the Tier-2 ceiling of 2.50% (Policy §3.2), so no breach — but approaching the limit.

---

## What I'd Improve

1. **Structured data retrieval for CSV:** Plain text chunking of CSV rows is lossy — a dedicated SQL or pandas tool would allow exact aggregations (SUM, GROUP BY region) instead of relying on the LLM to approximate from retrieved text chunks.

2. **Hybrid search:** Combine dense (embedding) retrieval with sparse (BM25 keyword) retrieval. Supplier names are exact-match lookups where BM25 outperforms cosine similarity significantly.

3. **Source citation in answers:** Configure the chain to append the source chunk to each answer, making outputs auditable.

4. **Metadata filtering:** Tag each chunk with structured metadata and apply pre-filters before vector search, reducing irrelevant chunk retrieval.

5. **Guardrails / hallucination detection:** Add a self-verification step where the LLM confirms each numeric claim against retrieved context before responding.

6. **Chat history persistence:** Adding a Redis or Postgres memory layer would support ongoing supplier review sessions.

---

## Repository Structure

```
scm-assistant-bot/
├── scm_assistant.json       # Exported Flowise chatflow
├── README.md                # This file
├── .gitignore               # Excludes .env and API keys
└── screenshots/
    ├── 01_document_store.png
    ├── 02_chunk_config_a.png
    ├── 03_chatflow_canvas.png
    ├── 04_share_chatbot.png
    └── 05_sample_qa.png
```

---

## Security

API keys are **never** committed to this repository. All credentials are managed via environment variables and excluded by `.gitignore`.

---

*Trinamix INC · Junior AI Engineer Hiring Task · Ref: TX-JrAI-003*
