# AskIIITA 🎓

> An AI-powered RAG chatbot for IIIT Allahabad — ask questions about academics, fees, calendar, placements, and more. Powered by **Gemini** + **MongoDB Atlas Vector Search** + **LangChain** + **Streamlit**.

---

## Project Structure

```
AskIIITA/
├── app.py              # Streamlit chat UI
├── ingest.py           # PDF → chunks → embeddings → Atlas
├── check_setup.py      # Pre-flight environment checker
├── .env                # Your secrets (never commit this!)
├── requirements.txt
└── data/
    └── *.pdf           # Put your IIITA PDFs here
```

---

## Quick Start

### 1 · Clone & install dependencies

```bash
pip install -r requirements.txt
```

### 2 · Fill in `.env`

```
MONGO_URI=mongodb+srv://user:password@cluster.mongodb.net/?retryWrites=true&w=majority
GEMINI_API_KEY=your_key_here
MONGO_DB_NAME=askiiita
MONGO_COLLECTION_NAME=documents
ATLAS_VECTOR_SEARCH_INDEX=vector_index
```

### 3 · MongoDB Atlas setup

| Step | What to do |
|------|-----------|
| **Cluster** | Create a free **M0** cluster at [cloud.mongodb.com](https://cloud.mongodb.com) |
| **Database** | Create database `askiiita`, collection `documents` |
| **Network** | Add `0.0.0.0/0` to **Network Access** (for testing) |
| **Vector Index** | Go to **Atlas Search → Create Index** and paste the JSON below |

**Vector Search Index JSON** (paste in Atlas UI → JSON editor):

```json
{
  "fields": [
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 768,
      "similarity": "cosine"
    }
  ]
}
```

> **Index name**: `vector_index` (must match `ATLAS_VECTOR_SEARCH_INDEX` in `.env`)

### 4 · Check your setup

```bash
python check_setup.py
```

All ✅? Move on. ❌ means something is misconfigured — fix it first.

### 5 · Drop PDFs into `./data/` and ingest

```bash
# Create the data folder
mkdir data

# Copy your PDFs there, then:
python ingest.py                        # all PDFs in ./data/
python ingest.py data/fee_structure.pdf # single file
```

### 6 · Launch the chat app

```bash
streamlit run app.py
```

Open [http://localhost:8501](http://localhost:8501) in your browser.

---

## How It Works

```
User Question
      │
      ▼
Gemini Embedding (text-embedding-004)
      │
      ▼
MongoDB Atlas Vector Search  ─────► Top-4 relevant chunks
      │
      ▼
Gemini LLM (gemini-1.5-flash)
  + System Prompt + Chunks
      │
      ▼
Grounded Answer  →  Chat UI
```

---

## Tips

- **Start small**: ingest 1-2 PDFs first, confirm end-to-end works, then add more.
- **Chunk size**: 800 chars with 100 overlap is a good default; adjust in `ingest.py` if answers feel cut-off.
- **Re-ingesting**: Running `ingest.py` again will add new chunks (no duplicates check). If you want a clean slate, drop and recreate the Atlas collection.
- **Cost**: Gemini free tier is generous; MongoDB M0 is always free.

---

## Environment Variables Reference

| Variable | Description | Default |
|----------|-------------|---------|
| `MONGO_URI` | Atlas connection string | *required* |
| `GEMINI_API_KEY` | From aistudio.google.com | *required* |
| `MONGO_DB_NAME` | Database name | `askiiita` |
| `MONGO_COLLECTION_NAME` | Collection name | `documents` |
| `ATLAS_VECTOR_SEARCH_INDEX` | Atlas Search index name | `vector_index` |
