# 📄 PDF Ingestion & Text Utilities

This section explains **`ingest_pdf.py`** and **`utils.py`**, the two foundational files responsible for transforming raw PDF documents into clean, structured, and RAG-ready data.

These files work **before embeddings, vector databases, or LLMs** and form the backbone of the entire pipeline.

---

## 🧠 High-Level Overview

| File | Purpose |
|-----|--------|
| 🗂️ **ingest_pdf.py** | Controls the **document ingestion pipeline** (PDF → chunks) |
| 🛠️ **utils.py** | Provides **text processing utilities** (cleaning, splitting, chunking, prompting) |

👉 **Key idea:**  
`ingest_pdf.py` orchestrates the workflow, while `utils.py` contains the reusable intelligence.

---

## 📘 ingest_pdf.py — PDF Ingestion Pipeline

### 🎯 What this file does

`ingest_pdf.py` is responsible for converting a **raw PDF document** into **structured semantic chunks** that are ready for embedding and retrieval.

It performs the following steps:

1. 📥 Download the PDF (if not available locally)
2. 📄 Read the PDF page-by-page
3. 🧹 Clean extracted text
4. ✂️ Split text into sentences
5. 🧩 Group sentences into meaningful chunks
6. 🚫 Remove tiny / noisy chunks
7. 💾 Save final chunks to disk (`parquet`)

---

## 🔄 Pipeline Flow

```text
PDF
 ↓
Page-level text extraction
 ↓
Sentence splitting (spaCy)
 ↓
Sentence chunking
 ↓
Noise filtering
 ↓
chunks.parquet (RAG-ready)
```

---

### 🧩 Key Functions

#### `download_pdf()`
- Downloads the PDF only if it doesn’t exist
- Ensures reproducibility and automation

#### `open_and_read_pdf()`
- Extracts text per page using PyMuPDF
- Computes page-level statistics (tokens, words, sentences)

#### `add_sentences_to_pages()`
- Applies linguistic sentence segmentation
- Adds sentence lists to each page

#### `build_chunks_from_pages()`
- Groups sentences into paragraph-like chunks
- Preserves semantic meaning

#### `ingest_pdf()`
- Orchestrates the full ingestion pipeline
- Saves the final output to `chunks.parquet`

---

### 📦 Output

The output is a **structured dataset** where each row represents a meaningful text chunk with metadata:

- Page number
- Clean chunk text
- Token / word / character statistics

This file **never talks to embeddings, Pinecone, or LLMs**.

---

## 🛠️ utils.py — Text Processing & Prompt Utilities

### 🎯 What this file does

`utils.py` contains all **reusable helper functions** related to text processing and prompt construction.

It focuses on:
- Cleaning text
- Linguistic sentence splitting
- Semantic chunk creation
- Noise filtering
- RAG prompt formatting

---

### 🧠 Key Functions Explained

#### 🧹 `text_formatter()`
- Cleans raw PDF text
- Removes line breaks and spacing artifacts

#### ✂️ `split_sentences_spacy()`
- Uses spaCy’s sentencizer
- Produces clean sentence-level splits
- More reliable than regex-based splitting

#### 🧩 `create_sentence_chunks()`
- Groups sentences into fixed-size semantic chunks
- Preserves meaning and structure
- Computes chunk-level statistics

#### 🚫 `filter_chunks()`
- Removes very small or irrelevant chunks
- Eliminates headers, footers, and noise

#### 🧠 `prompt_formatter()`
- Builds a **structured RAG prompt**
- Injects retrieved context
- Enforces grounded, detailed answers
- Prevents hallucinations

---

## ⚖️ Difference Between ingest_pdf.py & utils.py

| Aspect | ingest_pdf.py | utils.py |
|-----|--------------|---------|
| Role | Pipeline controller | Utility functions |
| Responsibility | Orchestration | Text intelligence |
| Handles files | ✅ Yes | ❌ No |
| Reusable across projects | ❌ Mostly | ✅ Yes |
| Knows about PDFs | ✅ Yes | ❌ No |
| Knows about RAG prompting | ❌ No | ✅ Yes |

---

## 🔗 How They Work Together

```python
ingest_pdf.py
 ├─ calls text_formatter()
 ├─ calls split_sentences_spacy()
 ├─ calls create_sentence_chunks()
 └─ calls filter_chunks()

utils.py
 └─ provides all text-processing logic

