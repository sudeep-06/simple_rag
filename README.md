# Simple RAG Workflow with LlamaIndex

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![LlamaIndex](https://img.shields.io/badge/LlamaIndex-RAG-orange.svg)](https://www.llamaindex.ai/)
[![Ollama](https://img.shields.io/badge/Ollama-Local_LLM-black.svg)](https://ollama.com/)

A practical Retrieval-Augmented Generation (RAG) example built with **LlamaIndex**, **Ollama**, and local Hugging Face embeddings. The project demonstrates document ingestion, vector retrieval, response synthesis, and RAG evaluation.

## Features

- Loads documents from a directory with `SimpleDirectoryReader`
- Builds an in-memory vector index with `VectorStoreIndex`
- Uses the `BAAI/bge-small-en-v1.5` embedding model
- Retrieves the two most relevant document chunks for each query
- Generates streaming answers with Ollama
- Includes a reusable Python workflow and an interactive Jupyter example
- Includes an Opik-based evaluation notebook

## Repository Structure

```text
.
├── data/
│   └── DeepSeek.pdf          # Default document used by workflow.ipynb
├── eval-data/
│   ├── paul_graham/          # Documents used for RAG evaluation
│   └── test.csv              # Question, answer, and context dataset
├── workflow.py               # Reusable RAG workflow implementation
├── workflow.ipynb            # Interactive end-to-end workflow
├── evaluation.ipynb          # Tracing and RAG evaluation with Opik
├── .env.example              # Optional environment variable template
└── README.md
```

## How It Works

1. `SimpleDirectoryReader` loads files from a source directory.
2. Documents are converted into embeddings with Hugging Face.
3. `VectorStoreIndex` stores the embeddings and enables similarity search.
4. The workflow retrieves the two most relevant nodes for a question.
5. `CompactAndRefine` combines the retrieved context with the Ollama model's response generation.

## Prerequisites

- Python 3.10 or newer
- [Ollama](https://ollama.com/)
- Jupyter, for running the notebooks

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/sudeep-06/simple_rag.git
cd simple_rag
```

### 2. Create and activate a virtual environment

**macOS/Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Windows**

```powershell
python -m venv .venv
.venv\Scripts\activate
```

### 3. Install Python dependencies

```bash
python -m pip install --upgrade pip
pip install llama-index nest-asyncio llama-index-llms-ollama llama-index-embeddings-huggingface jupyter python-dotenv
```

For the evaluation notebook, also install:

```bash
pip install opik pandas
```

### 4. Install Ollama

**macOS**

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**Linux**

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Windows users can download the installer from the [Ollama website](https://ollama.com/download).

### 5. Download a model

The workflow defaults to `llama3.2`:

```bash
ollama pull llama3.2
```

Gemma 3 is also supported by the evaluation notebook:

```bash
ollama pull gemma3
```

## Start Ollama

Ollama must be running before executing the workflow or notebook:

```bash
ollama serve
```

Leave that process running in a separate terminal.

## Usage

### Run the Python workflow

```bash
python workflow.py
```

This loads `data/DeepSeek.pdf`, asks `How was DeepSeekR1 trained?`, and streams the generated answer.

### Use the workflow in your own code

```python
import asyncio
from workflow import RAGWorkflow


async def main():
    workflow = RAGWorkflow(model_name="llama3.2")

    await workflow.ingest_documents("data")
    response = await workflow.query("How was DeepSeekR1 trained?")

    async for chunk in response.async_response_gen():
        print(chunk, end="", flush=True)


asyncio.run(main())
```

You can use another local model by changing `model_name`:

```python
workflow = RAGWorkflow(model_name="gemma3")
```

### Run the interactive notebook

```bash
jupyter notebook workflow.ipynb
```

## Evaluation

`evaluation.ipynb` demonstrates tracing and evaluation with [Opik](https://www.comet.com/site/products/opik/). It:

- Switches between Ollama models
- Logs LlamaIndex operations
- Creates an evaluation dataset from `eval-data/test.csv`
- Measures hallucination, answer relevance, context precision, and context recall

Start Jupyter and open the notebook:

```bash
jupyter notebook evaluation.ipynb
```

You may need to configure Opik and environment variables before running all notebook cells:

```bash
cp .env.example .env
```

> `.env.example` is a template. Add only the credentials required by the services you enable.

## Customizing the Workflow

| Setting | Location | Default |
| --- | --- | --- |
| Chat model | `RAGWorkflow(model_name=...)` | `llama3.2` |
| Embedding model | `RAGWorkflow(embedding_model=...)` | `BAAI/bge-small-en-v1.5` |
| Input directory | `ingest_documents(directory)` | `data` |
| Retrieved chunks | `similarity_top_k` in `workflow.py` | `2` |

## Troubleshooting

### Ollama connection errors

Make sure Ollama is running:

```bash
ollama serve
```

Confirm that the requested model is installed:

```bash
ollama list
```

### Missing model

```bash
ollama pull llama3.2
```

### Hugging Face embedding download

The embedding model is downloaded automatically the first time it is used. The machine needs internet access for the initial download.

### PDF or document not found

Pass a directory that exists and contains supported documents:

```python
await workflow.ingest_documents("path/to/your/documents")
```

## License

No license is currently specified for this repository.
