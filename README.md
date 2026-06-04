# Cloud-Based Document Processing Pipeline

A scalable Databricks notebook for extracting, parsing, sorting, and searching through PDF and Word documents stored in cloud storage.

## Overview

This project implements a distributed document processing pipeline that:

- **Ingests** binary PDF and Word documents from cloud storage (Databricks Volumes)
- **Extracts** document titles and content using intelligent metadata parsing
- **Sorts** documents by title across distributed nodes
- **Searches** for keywords and highlights matching text in PDFs
- **Outputs** highlighted documents to cloud storage

## Features

### 1. **Intelligent Document Extraction**

- **PDF Processing**: Extracts metadata titles with fallback to largest font text or first line
- **Word Processing**: Parses `.docx` files with support for styled title detection
- **Error Handling**: Graceful fallbacks for corrupted or malformed files

### 2. **Distributed Processing**

- Leverages Apache Spark for parallel execution across cluster nodes
- Processes large document volumes efficiently using RDDs and DataFrames
- Automatic schema flattening for downstream pipelines

### 3. **Full-Text Search**

- Pattern matching across document content
- Automatic PDF highlight generation for matched text
- Performance metrics for search operations

### 4. **Cloud Storage Integration**

- Direct integration with Databricks Unity Catalog Volumes
- Binary file reading from mounted cloud storage
- Output written back to cloud destinations

## Requirements

### Libraries

- `pymupdf` (fitz) - PDF processing
- `python-docx` - Word document parsing
- `pyspark` - Distributed computing framework
- Python 3.7+

### Infrastructure

- Databricks workspace with mounted cloud storage volumes
- Configured paths pointing to raw documents and output directories

## Installation

1. Install required packages (runs automatically in Databricks):

```bash
%pip install pymupdf python-docx
```

2. Configure cloud storage paths in the notebook:

```python
SOURCE_DOCUMENTS_DIR = "/Volumes/test/default/raw_docs_volume/"
OUTPUT_HIGHLIGHTED_DIR = "/Volumes/test/default/raw_docs_volume/highlighted_output/"
```

## Usage

### Step 1: Document Ingestion & Extraction

```python
# Reads binary files and extracts titles and content
raw_binary_df = spark.read.format("binaryFile").load(SOURCE_DOCUMENTS_DIR)
documents_base_df = parsed_df.select(...)
```

### Step 2: Document Sorting

```python
# Sorts documents by title across nodes
sorted_df, sort_duration = execute_document_sorting(documents_base_df)
```

### Step 3: Text Search & Highlighting

```python
# Searches for keywords and highlights matching PDFs
keyword = "Cloud Computing"
matched_df, count, duration = search_and_highlight_documents(
    sorted_df,
    keyword,
    OUTPUT_HIGHLIGHTED_DIR
)
```

## Project Structure

```
cloud-based-project/
├── cloude based project.py    # Main Databricks notebook
├── README.md                  # This file
└── [Output files created in cloud storage]
```

## Output

The pipeline produces:

- **Parsed Documents DataFrame** - Structured data with titles, content, and processing status
- **Sorted Dataset** - Documents ordered alphabetically by title
- **Highlighted PDFs** - Original PDFs with search terms highlighted (saved to `OUTPUT_HIGHLIGHTED_DIR`)

## Performance Metrics

The notebook logs:

- Total documents successfully parsed
- Sorting execution time (seconds)
- Search operation duration (seconds)
- Number of documents with highlighted matches

## Error Handling

- Skips empty or corrupted files with status message
- Returns "Unsupported Extension" for non-PDF/DOCX files
- Captures exceptions with detailed error messages
- Filters failed records before downstream processing

## Configuration

Modify these variables to customize behavior:

| Variable                 | Purpose                                  |
| ------------------------ | ---------------------------------------- |
| `SOURCE_DOCUMENTS_DIR`   | Path to input documents in cloud storage |
| `OUTPUT_HIGHLIGHTED_DIR` | Path for saving highlighted PDFs         |
| `keyword`                | Search term for text matching            |
| `file_size_bytes`        | Document size in extraction output       |

## Dependencies Overview

| Package       | Purpose                                                |
| ------------- | ------------------------------------------------------ |
| `pymupdf`     | PDF metadata and content extraction, text highlighting |
| `python-docx` | Word document parsing and title detection              |
| `pyspark`     | Distributed DataFrame operations and UDFs              |

## Future Enhancements

- Support for additional formats (TXT, RTF, Excel)
- OCR capabilities for scanned PDFs
- Machine learning-based title detection
- Advanced search with regex and fuzzy matching
- Document classification and tagging

## Troubleshooting

- **Files not found**: Verify `SOURCE_DOCUMENTS_DIR` path and cloud storage mount
- **Empty results**: Check file extensions (.pdf, .docx) and file permissions
- **Memory issues**: Reduce batch sizes or optimize Spark partitioning
- **Highlight failures**: Ensure output directory exists and has write permissions
