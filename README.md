# GPT-OSS-120B Document RAG

A Streamlit app that lets you upload a PDF and ask questions about it, powered by **LangChain**, **ChromaDB**, and **Groq's GPT-OSS-120B** model.

## Overview

Upload any PDF through the browser, and the app will process it into a searchable vector database. You can then ask natural-language questions about the document's content and get grounded, source-based answers.

### How It Works

**Ingestion (on upload):**
```
Uploaded PDF → saved to disk → UnstructuredPDFLoader → RecursiveCharacterTextSplitter (chunking)
             → HuggingFace Embeddings → Chroma vector store (persisted to disk)
```

**Question answering:**
```
User question → Chroma retriever → relevant chunks → RetrievalQA chain (GPT-OSS-120B via Groq)
             → generated answer
```

## Tech Stack

- **[Streamlit](https://streamlit.io/)** — web UI (file upload, text input, response display)
- **LangChain** (`langchain`, `langchain-community`, `langchain-huggingface`, `langchain-text-splitters`, `langchain-chroma`, `langchain-groq`) — RAG orchestration
- **[Unstructured](https://unstructured.io/)** (`unstructured`, `unstructured[pdf]`, `langchain-unstructured`) — PDF parsing
- **[ChromaDB](https://www.trychroma.com/)** — vector store
- **[Groq](https://groq.com/)** — fast inference for the `openai/gpt-oss-120b` model
- **python-dotenv** — environment variable management

## Project Structure

```
.
├── app.py                # Streamlit UI — file upload, question input, answer display
├── rag_utility.py         # Core RAG logic — document processing and Q&A
├── requirements.txt
├── env.txt                # Template for your .env file
└── doc_vectorstore/        # Generated Chroma vector store (created on first upload)
```

## Setup

### Prerequisites

- Python 3.11+ (see note below on Python 3.14 compatibility)
- A [Groq API key](https://console.groq.com/keys)

### Installation

```bash
# Clone the repo
git clone <your-repo-url>
cd <your-repo-name>

# Create and activate a virtual environment
python3 -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Environment Variables

Copy the provided template and fill in your own key:

```bash
cp env.txt .env
```

Then edit `.env` to set your actual key:

```
GROQ_API_KEY="Your API Key"
```

> Make sure `.env` is listed in your `.gitignore` so it's never committed. Only `env.txt` (with a placeholder) should be tracked in the repo.

## Usage

Run the app with:

```bash
streamlit run app.py
```

This opens the app in your browser (typically at `http://localhost:8501`). From there:

1. Upload a PDF using the file uploader
2. Wait for "Document Processed Successfully" to confirm it's been embedded and stored
3. Type a question about the document in the text box
4. Click **Answer** to get a response grounded in the document's content

## Configuration

Key parameters you can tune in `rag_utility.py`:

| Parameter | Description |
|---|---|
| `model` | Groq model used for generation (currently `openai/gpt-oss-120b`) |
| `temperature` | LLM sampling temperature (default: 0 for deterministic answers) |
| `chunk_size` | Size of each text chunk (default: 2000) |
| `chunk_overlap` | Overlap between chunks (default: 200) |

## Notes

- Groq's model lineup changes over time — if you hit a `model_decommissioned` error, check the [Groq models page](https://console.groq.com/docs/models) for the current recommended replacement.
- This project uses `langchain-classic` for `RetrievalQA`, since that chain was moved out of core `langchain` in recent versions (1.x+). Make sure `langchain-classic` is included in your `requirements.txt` if it isn't already.
- If running on Python 3.14, some dependencies (e.g. `tiktoken`) may not have prebuilt wheels yet and can fail to install. If you hit build errors, use Python 3.11 or 3.12 instead.
- Uploaded PDFs and the vector store are saved locally in the project directory — for a shared or production deployment, consider using separate, session-scoped storage per user.

## License

Add your license of choice here (e.g. MIT).
