# Docs Database Parser - Setup & Usage Guide

## Prerequisites
- **Python**: 3.9+

---

## 1. Ingestion & Cleaning

Detect source file types in a folder:
```bash
python scripts/parse_sources.py --detect ./input_docs
```

Run a dry run (extracts, cleans boilerplate, verifies completeness):
```bash
python scripts/parse_sources.py --source ./input_docs --dest ./output_db --dry-run
```

Compile structured database files:
```bash
python scripts/parse_sources.py --source ./input_docs --dest ./output_db --name "TechDocs"
```
