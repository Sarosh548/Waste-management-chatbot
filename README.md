# MOKK

An AI-powered **Retrieval-Augmented Generation (RAG)** chatbot for technical questions about solid waste management. It retrieves relevant passages from engineering books and PDF references, then uses a language model to generate grounded answers with source metadata.

## Features

- PDF ingestion, text extraction, overlapping chunking, and metadata preservation.
- Local 384-dimensional embeddings using Sentence Transformers — `all-MiniLM-L6-v2`.
- Persistent vector search with ChromaDB.
- Configurable vector, keyword, and hybrid retriever implementations.
- LLM gateway supporting Groq (default), Google Gemini, OpenAI, Anthropic, and Ollama.
- FastAPI backend with chat, health, and document endpoints.
- SQLite-backed chat session history.
- Responsive standalone web chat and an embeddable Shadow DOM widget.
- Automated tests using pytest.

## Architecture

~~~text
PDF Books / Manuals
        |
        v
 PDF Parser -> Text Chunker -> Embeddings -> ChromaDB
                                               |
User Question -> Retriever -> Context Formatter
                                      |
                                      v
                                  LLM Gateway
                                      |
                                      v
                              Answer + Sources
                                      |
                                      v
                                  FastAPI
                                      |
                                      v
                           Web Chat / Widget
~~~

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.11+ |
| Backend | FastAPI, Uvicorn |
| PDF processing | PyMuPDF |
| Embeddings | Sentence Transformers, all-MiniLM-L6-v2 (384 dimensions) |
| Vector database | ChromaDB |
| Retrieval | Vector search, BM25 keyword and hybrid retrievers |
| LLM providers | Groq, Gemini, OpenAI, Anthropic, Ollama |
| Configuration | YAML, Pydantic, python-dotenv |
| Chat history | SQLite / aiosqlite |
| Frontend | HTML, CSS, JavaScript |
| Testing | pytest |

## Project Structure

~~~text
Waste-management-chatbot/
├── chatbot/
│   ├── api/
│   │   ├── routes/             # Chat, health, and document routes
│   │   ├── services/           # Request orchestration
│   │   ├── chat_history.py
│   │   ├── main.py
│   │   └── models.py
│   ├── frontend/
│   │   ├── index.html
│   │   ├── styles.css
│   │   ├── chat.js
│   │   ├── widget.js
│   │   └── widget-demo.html
│   ├── ingestion/              # PDF parsing, chunking, embeddings, storage
│   ├── llm_gateway/            # Provider integrations and prompt templates
│   ├── retriever/              # Vector, keyword, and hybrid retrieval
│   └── config.py
├── tests/
│   ├── api/
│   ├── ingestion/
│   ├── llm_gateway/
│   └── retriever/
├── config.yaml
├── requirements.txt
└── pyproject.toml
~~~

## Getting Started

### Prerequisites

- Python 3.11 or newer
- Git
- An API key for your chosen LLM provider (Groq is the default)
- PDF reference documents for your knowledge base

### 1. Clone the repository

~~~bash
git clone https://github.com/Sarosh548/Waste-management-chatbot.git
cd Waste-management-chatbot
~~~

### 2. Create a virtual environment

**Windows PowerShell:**

~~~powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
~~~

**macOS / Linux:**

~~~bash
python3 -m venv .venv
source .venv/bin/activate
~~~

### 3. Install dependencies

~~~bash
python -m pip install --upgrade pip
pip install -r requirements.txt
~~~

### 4. Set your API key

Create a `.env` file in the repository root. For the default Groq provider:

~~~dotenv
GROQ_API_KEY=your_groq_api_key
~~~

Never commit `.env` or publish API credentials.

### 5. Review configuration

Edit `config.yaml` to set the PDF source directory, chunking options, embedding model, vector store, retriever type, LLM provider/model, chat history, and server settings.

Default settings use `./Books` for source PDFs, ChromaDB for vector storage, `all-MiniLM-L6-v2` for embeddings, and Groq for language generation.

## Build the Knowledge Base

Place PDF books or manuals in the configured source directory (default: `Books/`), then run:

~~~bash
python -m chatbot.ingestion.cli --source ./Books
~~~

Useful CLI commands:

~~~bash
# Display vector-store statistics
python -m chatbot.ingestion.cli --stats

# Clear the configured vector collection
python -m chatbot.ingestion.cli --clear
~~~

**Warning:** `--clear` deletes data from the configured vector collection. Use it only when you intentionally want to rebuild the knowledge base.

The source PDF corpus and generated vector database are local data and are not included in this repository by default. Ingest your own documents before asking document-specific questions.

## Run the Application

From the repository root:

~~~bash
uvicorn chatbot.api.main:app --reload --host 0.0.0.0 --port 8000
~~~

Open:

- Chat interface: http://localhost:8000/
- API health: http://localhost:8000/api/health
- Interactive API documentation: http://localhost:8000/docs

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/chat` | Submit a chat message and receive an answer with source metadata |
| GET | `/api/health` | Check API health |
| GET | `/api/documents` | Get document/knowledge-base information |

Example request:

~~~bash
curl -X POST "http://localhost:8000/api/chat" \
  -H "Content-Type: application/json" \
  -d '{"message":"What are the main methods of solid waste treatment?"}'
~~~

A session identifier can be supplied to continue an existing conversation. Refer to `chatbot/api/models.py` and the interactive API documentation for the exact request and response schemas.

## Embeddable Widget

The project includes `chatbot/frontend/widget.js`. It creates a floating chat interface and uses the Shadow DOM to isolate widget styles from the host website.

After hosting the widget file and making the API reachable, add this script to the host page:

~~~html
<script
  src="https://YOUR-FRONTEND-HOST/widget.js"
  data-api-url="https://YOUR-API-HOST">
</script>
~~~

Replace the example hostnames with your actual frontend and API URLs. See `chatbot/frontend/widget-demo.html` for a local integration example.

## LLM Provider Configuration

Groq is selected by default. The codebase also includes integrations for Google Gemini, OpenAI, Anthropic, and Ollama. Change the provider and model in `config.yaml`, set the corresponding environment variable/API key where required, and restart the server. Refer to the provider implementation for the exact environment-variable name.

## Run Tests

~~~bash
python -m pytest
~~~

To run selected test groups:

~~~bash
python -m pytest tests/api/
python -m pytest tests/retriever/
~~~

## Data and Security Notes

- `config.yaml` contains application configuration.
- `.env` stores private credentials and must never be committed.
- `Books/` is the default input directory for PDF documents.
- `data/` is used for generated local data, including ChromaDB and chat history.
- Back up important source PDFs and vector-store data.
- Restrict CORS origins appropriately before exposing the API publicly.

## Deployment Note

The frontend and backend must be connected for the complete chatbot to work. A frontend-only deployment is not sufficient: the backend also needs to be running with its LLM credentials and an initialized knowledge base.

## License

No license file is currently included in this repository. Add a LICENSE file if you intend to distribute the project under a particular open-source license.
