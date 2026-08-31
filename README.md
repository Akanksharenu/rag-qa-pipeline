# RAG QA Pipeline

A production-ready Retrieval-Augmented Generation (RAG) system that lets you ask questions about any PDF document. Built with Anthropic Claude, ChromaDB, LangChain, and FastAPI.

## What it does

1. **Ingest** — upload any PDF, split it into chunks, embed and store in ChromaDB
2. **Retrieve** — given a question, find the most semantically relevant chunks
3. **Generate** — pass retrieved context to Claude to produce a grounded answer
4. **Evaluate** — score the answer on relevance and groundedness using LLM-as-judge

## Tech stack

| Layer | Technology |
|-------|-----------|
| LLM | Anthropic Claude (claude-sonnet-4-6) |
| Vector DB | ChromaDB (local, persistent) |
| Embeddings | ChromaDB default (sentence-transformers) |
| Document loading | LangChain + PyPDF |
| API | FastAPI + Uvicorn |
| Containerization | Docker |

## Project structure

```
rag-qa-pipeline/
├── src/
│   ├── rag_pipeline.py   # Core RAG logic: ingest, retrieve, generate, evaluate
│   └── api.py            # FastAPI REST API
├── data/                 # Drop your PDFs here
├── requirements.txt
├── Dockerfile
├── .env.example
└── README.md
```

## Getting started

### 1. Clone and install

```bash
git clone https://github.com/Akanksharenu/rag-qa-pipeline.git
cd rag-qa-pipeline

python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

### 2. Set your API key

```bash
cp .env.example .env
# Edit .env and add your Anthropic API key
export ANTHROPIC_API_KEY=your_key_here
```

### 3. Run from the command line

```bash
# Ingest a PDF
python src/rag_pipeline.py ingest data/your_document.pdf

# Ask a question
python src/rag_pipeline.py ask "What are the main findings?"
```

### 4. Run the API server

```bash
cd src
uvicorn api:app --reload
```

API docs available at: `http://localhost:8000/docs`

**Ingest a PDF:**
```bash
curl -X POST http://localhost:8000/ingest \
  -F "file=@data/your_document.pdf"
```

**Ask a question:**
```bash
curl -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "What are the key conclusions?"}'
```

### 5. Run with Docker

```bash
docker build -t rag-qa-pipeline .
docker run -p 8000:8000 -e ANTHROPIC_API_KEY=your_key rag-qa-pipeline
```

## Example output

```
[query] What are the key findings of the report?

[retrieve] Found 5 relevant chunks

[answer]
Based on the document, the key findings are:
1. ...
2. ...
3. ...

[eval] Relevance: 5/5 | Groundedness: 5/5
[eval] The answer directly addresses the question using information from the provided context.
```

## Key design decisions

**Why ChromaDB?** Runs locally with zero setup — no API keys, no cloud account needed. Persistent storage means embeddings survive restarts.

**Why chunk overlap?** 50-character overlap between chunks prevents context loss at boundaries, improving retrieval accuracy for questions that span multiple sections.

**Why LLM-as-judge?** Instead of relying on human evaluation, the pipeline self-scores every answer on relevance and groundedness. This creates a feedback loop for catching regressions without a golden dataset.

**Why separate retrieve and generate steps?** Decoupling retrieval from generation makes it easy to swap the vector DB or LLM independently, and enables logging retrieved chunks for debugging poor answers.

## Author

Akanksha Renukuntla — [LinkedIn](https://linkedin.com/in/renukuntla2801) | [Portfolio](https://my-portfolio-zeta-five-7715kte4lp.vercel.app)
