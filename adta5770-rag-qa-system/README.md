# Sustainable Supply Chain Management — Q&A Search System

> **Retrieval-Augmented Generation (RAG) pipeline on Google Cloud Vertex AI**
> ADTA 5770 — Generative AI with LLMs · UNT Spring 2026 · Group 4 Final Project

A production-grade Q&A system that answers natural-language questions about Sustainable Supply Chain Management, grounded in 100 academic PDFs. Built with Vertex AI Matching Engine, Gemini 2.5 Pro, and LangChain.

---

## 🎯 What This Project Does

Given a question like *"What is sustainable supply chain management?"*, the system:

1. Embeds the question with `text-embedding-005` (768-dim)
2. Searches a Vertex AI Matching Engine index of 17,416 chunks from 100 SCM PDFs
3. Returns the top-10 most semantically similar chunks
4. Stuffs those chunks into a prompt template alongside the question
5. Sends the prompt to Gemini 2.5 Pro
6. Gemini returns a **grounded, cited answer** synthesized from the retrieved chunks

For out-of-domain questions like *"What are the symptoms of type 2 diabetes?"*, the system **refuses to fabricate** an answer — demonstrating critical RAG grounding safety.

## 🏗️ High-Level Design

![HLD](./docs/hld_architecture_diagram.png)

The HLD mirrors the canonical RAG pattern from Dr. Nguyen's lecture notebook:
- **Outer dashed box** = the Retrieval-Augmented Generator pipeline itself
- **Left inner box** = Information Retrieval (IR) System
- **Right inner box** = Text Generation Subsystem

## 🛠️ Technology Stack

| Layer | Technology |
|---|---|
| Cloud Platform | Google Cloud Platform — Vertex AI |
| Vector Store | Vertex AI Matching Engine (tree-AH, STREAM_UPDATE) |
| Embedding Model | `text-embedding-005` (768 dimensions) |
| Generator LLM | Gemini 2.5 Pro (`temp=0.2, top_p=0.8, top_k=40, max_tokens=8192`) |
| Orchestration | LangChain — modern API |
| Document Loader | `PyPDFLoader` (LangChain Community) |
| Chunking | `RecursiveCharacterTextSplitter` (`chunk_size=1000`, `overlap=200`) |
| IDE | Google Colab (Python 3.12) |
| Storage | GCS (`gs://adta5770-docs-folder-group4-kp`) |

## 🔄 10-Phase Pipeline

| Phase | Goal | Key Code |
|---|---|---|
| 1. Setup | Auth, imports, init Vertex AI | `aiplatform.init(project=..., location='us-central1')` |
| 2. Process | Load 100 PDFs from GCS | `PyPDFLoader(...).load()` |
| 3. Chunk | Split → 17,416 chunks | `RecursiveCharacterTextSplitter(1000, 200)` |
| 4. Index + Deploy | Tree-AH index + endpoint | `aiplatform.MatchingEngineIndex.create_tree_ah_index(...)` |
| 5. Configure | Wrap as VectorStore | `VectorSearchVectorStore.from_components(...)` |
| 6. Embed + Stream | Embeddings → Matching Engine | `vsvectordb.add_texts(texts, metadatas)` |
| 7. Smoke Test | Verify retrieval | `vsvectordb.similarity_search(query, k=3)` |
| 8. Q&A Subsystem | LLM + chains | `create_retrieval_chain(retriever, combine_docs_chain)` |
| 9. Test Queries | 8 FOUND + 2 NOT FOUND | `ask("What is sustainable SCM?")` |
| 10. Cleanup | Undeploy + delete | `endpoint.undeploy_index(...)`, `index.delete()` |

## 📊 Sample Results

### FOUND query — *"What is sustainable supply chain management?"*

Retrieved chunks from 5 distinct papers:
- *Digital Transformation in Supply Chains: AI, Blockchain, IoT*
- *Inventory Management as a Key Driver of Sustainability*
- *Digital Sustainable SC Competitiveness under Uncertainties*
- *Design-Based SC Operations Research Model — Resilience*
- *Enabling Global Sourcing in SC During Challenging Times*

Gemini's grounded answer:
> *SSCM is the integration of sustainability into supply chain operations — a fundamental component of contemporary business strategy. It involves three dimensions: economic, environmental, and social. Goals: mitigate GHG emissions and resource depletion while improving compliance and resilience. Strategy: innovation, stakeholder involvement, and capacity development.*

### NOT FOUND query — *"Symptoms and treatment of type 2 diabetes?"*

Gemini's response:
> *"I cannot determine the answer to that."*

→ Refusal-to-hallucinate verified.

## 📁 Repository Structure

```
adta5770-rag-qa-system/
├── README.md                                               # this file
├── notebook/
│   ├── adta5770_group4_qasearch_ALL_PHASES.ipynb           # Phases 1–10 (full pipeline)
│   └── ADTA5770_Group4_Notebook1_PHASE_1-2.ipynb           # Phases 1–2 only (setup + ingest)
├── docs/
│   └── hld_architecture_diagram.png                        # High-Level Design
└── requirements.txt                                        # Python dependencies
```

## 🚀 Reproducing This Work

### Prerequisites
- Google Cloud Platform account with Vertex AI enabled
- Project with `aiplatform`, `Cloud Storage`, and `gcloud` permissions
- ~$15 in credit (Matching Engine deployment ≈ $0.30/hour)

### Steps
1. Open `notebook/adta5770_group4_qasearch_ALL_PHASES.ipynb` in Google Colab
2. Update `PROJECT_ID`, `REGION`, and `BUCKET_NAME` to your own
3. Run cells phase-by-phase (Phase 4 deploy takes 30–60 min)
4. Phase 9 contains 10 sample queries you can modify

### Cost Optimization
**Always undeploy the Matching Engine endpoint when not in use:**
```python
my_index_endpoint.undeploy_index(deployed_index_id="...")
```
This stops node-hour charges (~$0.30/hr) immediately. Index storage continues at pennies/day.

## 🧗 Key Engineering Lessons

1. **Vertex API rate limits matter** — embedding 17,416 chunks needs throttling (BATCH=25, SLEEP=3s) and 90s back-off on HTTP 429
2. **Endpoint deploys can hang** — always write a diagnostic cell that lists endpoints + deployed indexes before assuming failure
3. **LangChain API moved fast** — `RetrievalQA.from_chain_type` is deprecated; use `create_retrieval_chain` + `create_stuff_documents_chain`
4. **Singular vs plural matters** — `find_neighbors()` accepts `filter`, not `filters`
5. **GCS PDF loading is slow** — `GCSFileLoader` takes 2+ min/PDF; download locally first via `blob.download_to_filename()` then use `PyPDFLoader` (~5 min for 100 PDFs total)

## 👥 Team — Group 4

- **Karan Parekh** — Group Leader (infrastructure, RAG pipeline, HLD)
- **Sanjana Pendyala Ravinder** — System analysis, written deliverables
- **Sana Mhapsekar** — Instructor communication, prompt engineering
- **Medina Maloku** — Document curation, technical writing

**Instructor:** Dr. Thuan L. Nguyen, Ph.D. — Clinical Professor, College of Sciences, UNT

## 📜 License

MIT — see [../LICENSE](../LICENSE) at the repo root.
