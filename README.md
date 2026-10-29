# Hindi RAG Assistant — Document-Based Question Answering

A Retrieval-Augmented Generation (RAG) application that answers questions in Hindi and English using information from a Hindi PDF document. The system retrieves relevant passages from the document, generates grounded answers using a language model, and provides source citations for verification.

## Overview

This project implements an end-to-end RAG pipeline designed to improve question-answering accuracy for Hindi documents, including text with Devanagari characters and structured tables.

Instead of relying entirely on the language model's general knowledge, the application retrieves relevant document passages and uses them as evidence when generating answers.

### Key Features

- **Bilingual Question Answering:** Ask questions in Hindi or English.
- **Document-Based Responses:** Generate answers grounded in the supplied PDF.
- **Source Citations:** Include page numbers, section names, chunk IDs, and retrieval scores.
- **Multilingual Embeddings:** Use `BAAI/bge-m3` for cross-lingual retrieval.
- **Persistent Vector Database:** Store document embeddings using ChromaDB.
- **Table-Aware Retrieval:** Process table rows as individual searchable chunks.
- **Multiple LLM Providers:** Support Groq, Gemini, OpenAI, Anthropic, Ollama, and local Hugging Face models.
- **Offline Options:** Use a local language model or extractive answering mode without a cloud API key.
- **Evaluation Tools:** Measure retrieval performance, answer accuracy, and citation quality.
- **Interactive Chat:** Ask questions through a command-line chat interface.

## System Architecture

The application follows this pipeline:

```text
             Hindi PDF Document
                     |
                     v
          PDF Text and Table Extraction
                     |
                     v
         Devanagari Text Normalization
                     |
                     v
          Structure-Aware Chunking
                     |
                     v
        BAAI/bge-m3 Embedding Model
                     |
                     v
             ChromaDB Index
                     |
                     v
             User Question
                     |
                     v
          Semantic Search (Top 4)
                     |
                     v
        Retrieved Context and Metadata
                     |
                     v
           Language Model (LLM)
                     |
                     v
       Grounded Answer with Citations
```

## Technology Stack

| Component | Technology |
|---|---|
| Programming Language | Python |
| PDF Processing | PyMuPDF |
| Text Embeddings | BAAI/bge-m3 |
| Vector Database | ChromaDB |
| LLM Integration | Groq, Gemini, OpenAI, Anthropic, Ollama, Hugging Face |
| Configuration | Environment variables |
| Testing | pytest |
| Version Control | Git and GitHub |

## Project Structure

```text
rag-hindi/
├── data/                 # Source documents
├── docs/                 # Design notes and documentation
├── eval/                 # Evaluation datasets
├── hindi_rag/            # Core RAG implementation
├── outputs/               # Test logs and evaluation results
├── tests/                 # Unit tests
├── .env.example           # Environment configuration template
├── .gitignore
├── pytest.ini
├── rag.py                 # Command-line interface
├── requirements.txt       # Python dependencies
└── README.md
```

## Getting Started

### Prerequisites

- Python 3.13 (tested on Windows and Linux)
- Git
- Internet access for installing dependencies and downloading models
- An API key for a supported cloud language model, or sufficient resources for local inference

### 1. Clone the Repository

```bash
git clone https://github.com/Tusharbunny06/rag-hindi.git
cd rag-hindi
```

### 2. Create a Virtual Environment

**Windows PowerShell:**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**Linux or macOS:**

```bash
python -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Configure Environment Variables

Create a local `.env` file from the example template:

**Windows PowerShell:**

```powershell
Copy-Item .env.example .env
```

**Linux or macOS:**

```bash
cp .env.example .env
```

Open `.env` and configure the desired language-model provider using the variable names documented in `.env.example`.

For example, to use Groq, obtain an API key from [Groq Console](https://console.groq.com/) and enter it using the variable name specified in the template.

**Security:** Never commit your `.env` file or expose API keys in source code. Keep `.env.example` free of real credentials.

### 5. Build the Knowledge Base

Run the ingestion command:

```bash
python rag.py ingest
```

This command extracts document content, normalizes the text, creates chunks, generates embeddings, and stores the searchable vectors in ChromaDB.

**Note:** The first run downloads the `BAAI/bge-m3` embedding model, approximately 2.3 GB. Allow sufficient disk space and time for the download.

### 6. Run the Application

**Ask a question:**

```bash
python rag.py ask "कलाम का जन्म कहाँ हुआ?"
```

**Start interactive chat:**

```bash
python rag.py chat
```

**Inspect indexed chunks:**

```bash
python rag.py chunks
```

## Testing and Evaluation

The repository includes commands for testing the complete RAG pipeline and evaluating retrieval quality.

### Run the Required Test Queries

```bash
python rag.py test
```

### Run Extended Tests

```bash
python rag.py test --extra
```

### Evaluate Retrieval Performance

```bash
python rag.py eval
```

### Run Unit Tests

```bash
python -m pytest
```

The unit tests run without downloading the embedding model.

### Reported Evaluation Results

The following results are reported by the project's existing evaluation suite. They are benchmark results, not a guarantee of performance on every new document.

| Metric | Reported Result |
|---|---:|
| Retrieval Hit@1 | 95% |
| Retrieval Hit@3 | 95% |
| Retrieval Hit@5 | 100% |
| Cross-lingual English-to-Hindi retrieval | 12/12 |
| Answer accuracy on labelled questions | 20/20 |
| Correct-page citations | 20/20 |
| Out-of-scope question rejection | 3/3 |
| Median end-to-end time to first token | 1.07 seconds |

The evaluation uses 20 hand-labelled questions, including Hindi and English questions, plus three out-of-scope questions. Performance may vary with different documents, questions, hardware, and language-model providers.

Detailed logs and evaluation reports are available in the `outputs/` directory.

## How It Works

### 1. Document Processing

PyMuPDF extracts text and tables from the source PDF. The extracted text undergoes Unicode normalization and Devanagari-specific cleanup to improve retrieval quality.

### 2. Intelligent Chunking

The document is divided into smaller, searchable passages while respecting numbered sections, headings, paragraph boundaries, and Hindi sentence punctuation.

Table rows are converted into self-contained chunks to improve answers to questions involving dates, names, and structured information.

### 3. Semantic Search

The `BAAI/bge-m3` embedding model converts document passages and user questions into vectors. ChromaDB searches these vectors to retrieve the most relevant passages.

The system retrieves the top four passages and can expand retrieved context with related rows from small tables.

### 4. Answer Generation

The selected language model receives the retrieved passages and generates an answer in the language of the question. It is instructed to use the supplied evidence and indicate when the answer is not available in the document.

### 5. Citation Generation

The system maps answer references back to stored passage metadata. Citations can include the source page, section, chunk ID, and retrieval score, allowing users to verify the answer against the original document.

## Supported Language Model Backends

The project supports multiple backends:

- Groq
- Google Gemini
- OpenAI
- Anthropic
- Ollama
- Local Hugging Face models
- Extractive answering without an LLM

Provider credentials and configuration depend on the selected backend. Consult `.env.example` and `hindi_rag/config.py` for the supported settings.

When no cloud API key is configured, the application can use its local-model fallback. This may require downloading a model of approximately 3 GB and can be slow on CPU-only systems.

To use extractive answering without an LLM, run:

```bash
python rag.py ask "Your question here" --llm extractive
```

## Future Improvements

Potential enhancements include:

- Hybrid retrieval combining semantic search with BM25 keyword matching.
- Cross-encoder reranking to improve the relevance of retrieved passages.
- Larger evaluation datasets with independent human validation.
- Better support for Romanized Hindi queries.
- Optimized inference using ONNX, quantization, or GPU acceleration.
- More robust handling of different PDF layouts and document formats.
- Improved refusal thresholds for questions outside the source document.

## Troubleshooting

**PowerShell blocks virtual environment activation**

If PowerShell reports that script execution is disabled, you may enable local scripts for your user account:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Then activate the environment again.

**The application cannot find an API key**

Check that `.env` exists in the project root, that the variable name matches the example configuration, and that the selected provider's key is valid.

**The first run takes a long time**

The embedding model must be downloaded and loaded. Local language-model inference can also be slow on CPU-only hardware.

**Answers are not sufficiently accurate**

Verify that ingestion completed successfully, check the retrieved passages, and review the evaluation logs in `outputs/`.

## License

Add the license that applies to your project and its dependencies before redistributing it.

## Acknowledgements

This project uses open-source tools and models for PDF processing, multilingual embeddings, vector search, and language-model inference.

---

**Project Goal:** Build a reliable, bilingual, document-grounded question-answering system for Hindi content, with transparent citations and measurable retrieval and answer quality.
