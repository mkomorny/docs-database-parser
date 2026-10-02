# Docs Database Parser

[![Automated Release](https://img.shields.io/badge/release-automated_batch_pipeline-blue.svg)](https://github.com/mkomorny)
[![Pipeline Execution](https://img.shields.io/badge/dispatched_by-background_script-informational.svg)](https://github.com/mkomorny)

> [!NOTE]
> **Automated Distribution**: This repository was automatically sanitized, packaged, and published via a scheduled background batch staging pipeline. All file bundling, licensing, and repository synchronization were dispatched automatically by an automated release runner.

An automated source-to-database knowledge extraction and structuring pipeline. Ingests heterogeneous technical documentation (HTML captures, user manuals, transcripts, and Markdown docs), removes web boilerplate, normalizes technical content, and prepares structured reference databases (SQLite FTS5, Parquet, and modular Markdown).

## Ingestion Pipeline

1. **Source Discovery**: Automatically scans and classifies file formats across documentation folders.
2. **Noise & Boilerplate Stripping**: Removes non-content artifacts (cookie banners, navigation headers, social footers) while strictly preserving all code blocks, tables, warning callouts, and parameters.
3. **Structure Normalization**: Repairs broken Markdown headings, table alignments, and code language tags.
4. **Knowledge Indexing**: Prepares tokenized full-text search indexes compatible with SQLite FTS5 and RAG retrieval pipelines.

## Dependencies

- **Python**: Version 3.9+ (Standard Library: `re`, `os`, `sys`, `json`, `pathlib`, `argparse`).
- **Optional**: `sqlite3` (built-in to Python), `pandas`/`pyarrow` for Parquet exports.

## Instructions

See [INSTRUCTIONS.md](./INSTRUCTIONS.md) for pipeline execution commands.

## License

This project is licensed under the GNU General Public License v3.0 (GPL-3.0) - see the [LICENSE](./LICENSE) file for details.
