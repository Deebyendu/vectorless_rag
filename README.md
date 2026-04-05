# 🔍 Vectorless RAG with PageIndex & Groq

A **vector-free Retrieval-Augmented Generation (RAG)** pipeline that uses [PageIndex](https://pageindex.ai) to build a semantic tree index over PDF documents and [Groq](https://groq.com) (LLaMA 3.3 70B) to reason over that tree — no embeddings, no vector databases required.

---

## 🧠 How It Works

Traditional RAG systems rely on chunking documents, embedding those chunks, and running similarity searches. This project takes a different approach:

1. **PageIndex** parses a PDF and builds a **hierarchical tree index** — each node represents a section with a title, page number, and content.
2. An **LLM (via Groq)** receives the query and the compressed tree structure, then **reasons over the tree** to identify the most relevant node IDs.
3. The content of those nodes is retrieved and passed as context to a second LLM call that **generates a cited answer**.

```
PDF Document
     │
     ▼
PageIndex Tree (nodes with titles + page refs)
     │
     ▼
LLM Tree Search  ──► Relevant Node IDs
     │
     ▼
Node Content Retrieval
     │
     ▼
Answer Generation (with section + page citations)
```

---

## 📁 Project Structure

```
VECTORLESS RAG/
├── .gitignore
├── .python-version
├── GOVERNMENT OF INDIA.pdf       # Sample document used for testing
├── pageindex_vectorless_rag.ipynb # Main notebook
├── pyproject.toml
└── requirements.txt
```

---

## ⚙️ Setup

### 1. Clone the repository

```bash
git clone https://github.com/Deebyendu/vectorless_rag.git
cd vectorless-rag
```

### 2. Create and activate a virtual environment

```bash
python -m venv venv
source venv/bin/activate        # Linux/macOS
venv\Scripts\activate           # Windows
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Create a `.env` file in the project root:

```env
PAGEINDEX_API_KEY=your_pageindex_api_key_here
GROQ_API_KEY=your_groq_api_key_here
```

- Get your **PageIndex API key** at [pageindex.ai](https://pageindex.ai)
- Get your **Groq API key** at [console.groq.com](https://console.groq.com)

---

## 🚀 Usage

Open and run `pageindex_vectorless_rag.ipynb` in Jupyter. The notebook walks through each step:

### Step 1 — Submit a Document

```python
result = pi_client.submit_document("./GOVERNMENT OF INDIA.pdf")
doc_id = result["doc_id"]
```

### Step 2 — Wait for the Tree Index to Build

```python
# Polls until status == "completed"
# The index is cached by PageIndex for reuse
```

### Step 3 — Inspect the Tree Structure

```python
print_tree(pageindex_tree)
# [0000] Preface (p.1)
# [0001] MEMORANDUM OF ASSOCIATION (EXTRACT OF MAIN OBJECTS) (p.2)
# [0002] DIRECTORS / SIGNATORY DETAILS (p.2)
```

### Step 4 — Run a Query

```python
answer = vectorless_rag(
    query="What is the company name?",
    tree=pageindex_tree
)
```

**Example output:**
```
=======================================================
Query: What is the company name?
=======================================================

Reasoning: To find the company name, we need to look for sections that
typically contain this information...
Retrieved Node IDs: ['0001']
Sections found: ['MEMORANDUM OF ASSOCIATION (EXTRACT OF MAIN OBJECTS)']

Final Answer:
The company name is NexusGrid AI Solutions Private Limited
(MEMORANDUM OF ASSOCIATION, Page 2).
```

---

## 🔑 Key Functions

| Function | Description |
|---|---|
| `llm_tree_search(query, tree)` | Sends query + compressed tree to LLM; returns relevant node IDs and reasoning |
| `find_nodes_by_ids(tree, target_ids)` | Recursively retrieves full node content by ID |
| `generate_answer(query, nodes)` | Generates a cited answer from retrieved node content |
| `vectorless_rag(query, tree, verbose)` | End-to-end pipeline combining all three steps above |

---

## 📦 Dependencies

```
pageindex       # Tree-based document indexing
groq            # LLM inference (LLaMA 3.3 70B Versatile)
python-dotenv   # Environment variable management
```

---

## 💡 Why Vectorless?

| | Traditional RAG | Vectorless RAG |
|---|---|---|
| Indexing | Chunk + Embed | Tree structure |
| Retrieval | Vector similarity search | LLM reasoning over tree |
| Infrastructure | Vector DB required | No vector DB needed |
| Interpretability | Low (cosine scores) | High (LLM explains its reasoning) |
| Best for | Large corpora | Structured / hierarchical documents |

---

## 📄 License

MIT License — feel free to use, modify, and distribute.
