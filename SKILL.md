---
name: database-parser
description: Transforms web guide captures, YouTube transcript downloads, user manuals, knowledge guides, training docs, and raw PDFs into exhaustively complete, senior-engineer-quality LLM reference databases (SQLite FTS5, modular Markdown, Parquet, JSONL, search.py, INDEX.md, COVERAGE_AUDIT.md, MANIFEST.md). Strips only true junk (nav chrome, cookie banners, ads, boilerplate) and preserves EVERY substantive fact, parameter, code block, table, warning, and procedure step. Triggered by 'parse this into a database', 'build a reference database from', 'turn these guides into an LLM database', 'database-parser', or /database-parser.
---

# Database-Parser

Transform heterogeneous source material into exhaustively complete, production-grade LLM reference databases written to the standard of senior software engineers at top-tier AI model companies.

**Critical rule: preserve ALL substantive content.** Only strip true junk — navigation chrome, cookie consent, social widgets, ad blocks, duplicate boilerplate across multi-page captures. When in doubt, KEEP the content and flag it in the coverage audit for human review.

---

## MANDATORY RULE: Destination Must Be Specified First

> [!IMPORTANT]
> **Never start processing without confirming the destination directory.**
>
> If the user asks to parse sources into a database but does NOT explicitly name the destination:
> 1. **STOP immediately.**
> 2. Ask: *"Where should the database be created? (e.g., `/data/output` or `/data/output`)"*
> 3. Only proceed after the destination path is confirmed.

---

## When to Activate

Trigger this skill whenever the user says things like:
* *"Parse this into a database"* / *"Turn these guides into an LLM database"*
* *"Build a reference database from these captures"*
* *"Convert these YouTube guides and manuals into a searchable database"*
* *"Database-parser this folder"* / `/database-parser`
* *"Turn this guide-capture output into a proper database"*
* *"Make an LLM-usable reference from these training docs"*
* *"Rebuild the databases"* / *"Rebuild the datasets"*

---

## Source Detection (Auto-Recognized Input Types)

The parser automatically detects and handles these input types within a source folder:

| Source Type | Detection Signal | Parsing Strategy |
|---|---|---|
| **guide-capture output** | Contains `guide.md` + `manifest.json` + `images/` | Parse `guide.md`, strip placeholder figure notes, preserve all procedural content |
| **youtube-capture output** | Contains `guide.md` with `Video timestamp:` markers | Parse `guide.md`, preserve timestamps, prerequisites, troubleshooting, all steps |
| **scanner output** | Contains multiple `.md` files from crawled pages | Deduplicate repeated nav/header/footer blocks across pages |
| **Raw PDF files** | `.pdf` extension | PyMuPDF → PyPDF fallback, layout-aware paragraph joining |
| **Markdown / TXT docs** | `.md`, `.txt` extension | Direct read with minimal cleanup |
| **Mixed folders** | Any combination of the above | Process each file by its detected type |

---

## Run

```bash
python "<skill_dir>/scripts/parse_sources.py" \
  --source "<SOURCE_PATH>" \
  --dest "<DEST_PATH>" \
  --name "<DatasetName>"
```

`<skill_dir>` is this skill's directory (e.g., `~/.agents/skills/database-parser`).

### CLI Flags

| Flag | Default | Description |
|---|---|---|
| `--source <path>` | *(required)* | Source directory containing raw captures, guides, PDFs, or markdown |
| `--dest <path>` | *(required)* | Destination directory for the compiled database |
| `--name <name>` | Folder name | Custom dataset name |
| `--description <text>` | Auto-generated | Custom overview description |
| `--domain <preset>` | `auto` | Domain preset for INDEX.md (`auto`, `music`, `m4l`, `general`, etc.). `auto` detects from content |
| `--chunk-words <int>` | `500` | Target words per semantic chunk |
| `--chunk-overlap <int>` | `50` | Overlap words between adjacent chunks |
| `--dry-run` | off | Parse and clean without compiling (outputs cleaned markdown + audit only) |
| `--detect` | off | Only detect source types (no processing) |
| `--strict` | off | Fail on any flagged content loss instead of just auditing |
| `--builder-path <path>` | Auto-discovered | Explicit path to `build_dataset.py` from the `dataset-builder` skill |

---

## Intelligent Parsing Pipeline (6 Stages)

### Stage 1: Source Discovery
Walk the input directory. Classify each file or subfolder by source type (guide-capture, youtube-capture, scanner, PDF, markdown, text, mixed). Build a full manifest of processable files.

### Stage 2: Content Extraction
Extract clean text from each source using the appropriate extractor:
- **PDFs**: PyMuPDF (fitz) with page-aware extraction, fallback to PyPDF.
- **Markdown/TXT**: Direct UTF-8 read.
- **Guide/YouTube captures**: Parse the structured `guide.md` with awareness of section boundaries.
- **Scanner output**: Read all `.md` snapshots, maintaining page-level boundaries.

### Stage 3: Junk Stripping (Conservative)
Remove ONLY verified non-content artifacts. The junk detector targets:
- Navigation bars, breadcrumbs, and sidebar menus
- Cookie consent banners and GDPR notices
- Footer boilerplate (copyright lines, "powered by", social links)
- "Related articles" / "You might also like" blocks
- Advertisement placeholders and tracking pixels
- Duplicate headers/footers repeated across multi-page captures
- Empty or placeholder figure notes (keep substantive figure notes)
- Social share button text ("Share on Twitter", "Like on Facebook")
- Print/PDF buttons and page navigation controls

**Critical**: When the junk detector is uncertain, it KEEPS the content and flags it in the coverage audit. Better to include a cookie notice than to strip a procedure step.

### Stage 4: Completeness Verification
For each source document, compare word counts before and after cleaning:
- If a section loses ≤5% → clean pass, no flag.
- If a section loses 5–15% → minor flag in `COVERAGE_AUDIT.md`.
- If a section loses >15% → major flag with the stripped content shown for human review.

### Stage 5: Structure Normalization
- Normalize heading hierarchy (no H1→H4 jumps without H2, H3).
- Repair mismatched code block fences (backtick count balancing).
- Verify and fix table alignment (column count consistency).
- Merge paragraphs fragmented by page breaks mid-sentence.
- Preserve all code blocks, parameter tables, CLI examples, and warning callouts verbatim.
- Strip trailing whitespace and normalize line endings.

### Stage 6: Compilation
Write cleaned markdown files to a staging directory, then delegate to the existing `dataset-builder` engine (`build_dataset.py`) for final indexing:
- SQLite FTS5 database with Porter stemming + BM25 ranking
- Apache Parquet columnar archive (Snappy compressed)
- JSON Lines chunks and document catalogs
- Standalone `search.py` CLI and Python module
- `INDEX.md` router table with topic routing
- `SCHEMA.md` and `README.md`

Additionally generate:
- `MANIFEST.md` — provenance chain for every source (original path, URL, capture date, source skill, file type)
- `COVERAGE_AUDIT.md` — per-source word count comparison, flagged sections, and any content marked for human review

---

## Quality Standards (Senior Engineer Reference Corpus)

The output database must meet these standards:
1. **Exhaustive**: Every fact, parameter value, code example, table row, warning, caveat, and procedure step from the source material is present in the database.
2. **Structured**: Content is organized into logical categories with consistent heading hierarchies.
3. **Searchable**: BM25 full-text search returns relevant, well-ranked results for domain-specific queries.
4. **Traceable**: Every chunk carries provenance metadata (source file, page number, original URL if available).
5. **Auditable**: The coverage audit proves no substantive content was dropped.
6. **Token-efficient**: Individual markdown files stay under 15,000 characters for low-token LLM context windows.
7. **Self-documenting**: README.md, SCHEMA.md, INDEX.md, and MANIFEST.md provide complete documentation of the database structure, contents, and provenance.

---

## Output Directory Structure

```text
<DatasetName>/
├── markdown/                     ← Categorized, cleaned markdown files (<15k chars each)
│   ├── <category_1>/
│   │   ├── doc_001.md
│   │   └── doc_002.md
│   └── <category_2>/
├── <dataset>.db                  ← SQLite FTS5 database (documents, chunks, categories, chunks_fts)
├── <dataset>.parquet             ← Apache Parquet columnar archive (Snappy compressed)
├── <dataset>_chunks.jsonl        ← Semantic ~500-word chunks with metadata, tags, and provenance
├── <dataset>_docs.jsonl          ← Document-level catalog with word counts and summaries
├── search.py                     ← Interactive standalone BM25 full-text search CLI
├── INDEX.md                      ← High-density topic router table and document catalog
├── SCHEMA.md                     ← Comprehensive SQL and JSON schema documentation
├── README.md                     ← Dataset overview, statistics, and dual-access usage guide
├── MANIFEST.md                   ← Source provenance chain (paths, URLs, dates, source skills)
└── COVERAGE_AUDIT.md             ← Per-source completeness audit with flagged sections
```

---

## Integration with Existing Skills

- **Reads output from**: `guide-capture`, `youtube-capture`, `scanner`, `pdf-parsing`
- **Delegates compilation to**: `dataset-builder` (`build_dataset.py`)
- **Databases searchable by**: `ai-database-browser`, `producer-pal-genius`, and any skill that calls `search.py`
