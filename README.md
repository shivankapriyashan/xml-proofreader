# XML Proofreader with AI-Powered Error Detection

An intelligent XML proofreading tool that uses OpenAI's GPT models and RAG (Retrieval-Augmented Generation) to detect and annotate errors in XML documents based on custom style guides.

## Features

- **AI-Powered Error Detection**: Detects spelling, grammar, punctuation, capitalization, style guide violations, and clarity issues
- **RAG-Based Style Guide Integration**: Uses FAISS vector store for efficient retrieval of relevant style guide rules
- **Production-Grade Libraries**: Built with LangChain, FAISS, and OpenAI embeddings
- **Parallel Processing**: 5x faster processing with ThreadPoolExecutor (5 concurrent workers)
- **Text-Length Invariance**: Ensures output text length matches input (errors are wrapped, not replaced)
- **Persistent Vector Store**: Cache embeddings for faster subsequent runs
- **Flexible Logging Levels**: Control output verbosity with `--warning`, `--info`, or `--verbose` flags
- **Detailed Logging**: Comprehensive logs with timestamps saved to `logs/` directory

## Prerequisites

- Python 3.11+
- OpenAI API key
- Style guide in `.docx` format

## Installation

### 1. Clone or Download the Repository

```bash
cd "C:\Users\ROG\Downloads\AI Engineer Assignment\xml-proofreader"
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

**Dependencies include:**
- `lxml` - XML parsing
- `openai` - OpenAI API client
- `python-docx` - DOCX file reading
- `langchain` - Text splitting and document processing
- `langchain-openai` - OpenAI embeddings integration
- `langchain-community` - FAISS vector store
- `faiss-cpu` - Fast similarity search
- `psutil` - Performance monitoring
- `python-dotenv` - Environment variable management
- `langcodes` - Language code handling
- `numpy` - Numerical operations

### 3. Set Up Environment Variables

Create a `.env` file in the project root:

```env
API_KEY=your_openai_api_key_here
MODEL=gpt-4o-mini
```

**Environment Variables:**
- `API_KEY` (required): Your OpenAI API key
- `MODEL` (required): OpenAI model to use (e.g., `gpt-4o-mini`, `gpt-4`, `gpt-3.5-turbo`)

## Project Structure

```
xml-proofreader/
├── xml_proofreader.py          # Main application script
├── requirements.txt             # Python dependencies
├── .env                         # Environment variables (create this)
├── README.md                    # This file
├── data/
│   ├── system_prompt_template.txt  # System prompt template
│   └── Style Guide.docx         # Your style guide (place here)
├── files/
│   ├── example_input.xml        # Example input files
│   └── sample_input.xml
├── logs/                        # Auto-generated logs
│   └── xml-proofreader_YYYYMMDD_HHMMSS.log
└── vector_store/                # Auto-generated FAISS vector store
    ├── index.faiss
    └── index.pkl
```

## Usage

### First Run (Create Vector Store)

On the first run, you must provide the style guide to create the vector store:

```bash
python xml_proofreader.py --input ./files/example_input.xml --lang en --style-guide ".\data\Style Guide.docx"
```

**What happens:**
1. Loads the style guide from DOCX
2. Splits it into chunks using RecursiveCharacterTextSplitter
3. Creates embeddings using OpenAI's `text-embedding-3-small`
4. Saves FAISS vector store to `vector_store/` directory
5. Processes the XML file and outputs corrected version

### Subsequent Runs (Use Cached Vector Store)

After the first run, you can omit the `--style-guide` argument:

```bash
python xml_proofreader.py --input ./files/example_input.xml --lang en
```

**What happens:**
1. Loads existing vector store from cache
2. Processes the XML file (much faster startup)

### Update Vector Store

To update the vector store with a new style guide:

```bash
python xml_proofreader.py --input ./files/example_input.xml --lang en --style-guide ".\data\New Style Guide.docx"
```

This will overwrite the existing vector store.

### Logging Levels

Control the amount of output with logging flags:

**WARNING level (minimal output):**
```bash
python xml_proofreader.py --input ./files/example_input.xml --lang en --warning
# OR
python xml_proofreader.py --input ./files/example_input.xml --lang en -w
```

**INFO level (default - progress updates):**
```bash
python xml_proofreader.py --input ./files/example_input.xml --lang en --info
# OR
python xml_proofreader.py --input ./files/example_input.xml --lang en -i
```

**DEBUG level (detailed technical information):**
```bash
python xml_proofreader.py --input ./files/example_input.xml --lang en --verbose
# OR
python xml_proofreader.py --input ./files/example_input.xml --lang en -v
```

**Verbose output includes:**
- Retrieved style guide rules for each paragraph
- Individual error details per paragraph
- Text-length validation checks
- Vector store operations

## Command-Line Arguments

| Argument | Required | Description | Example |
|----------|----------|-------------|---------|
| `--input` | Yes | Input XML file path | `./files/example_input.xml` |
| `--lang` | Yes | Language code (BCP-47) | `en`, `fr`, `es` |
| `--style-guide` | No* | Path to style guide DOCX | `.\data\Style Guide.docx` |
| `--warning`, `-w` | No | Minimal output (errors/warnings only) | - |
| `--info`, `-i` | No | Progress updates and results | - |
| `--verbose`, `-v` | No | Detailed debug logging | - |

*Required on first run to create vector store

## Output

### Output File

The corrected XML file is saved with `.corrected.xml` suffix:

```
Input:  files/example_input.xml
Output: files/example_input.corrected.xml
```

### Error Annotation Format

Errors are wrapped with `<error>` tags:

```xml
<error type="spelling" correction="committee" reason="Misspelled word">committe</error>
<error type="styleguide" correction="$2 million" reason="Spell out million, billion, etc.">$2,000,000</error>
<error type="punctuation" correction="its" reason="Possessive its has no apostrophe">it's</error>
```

**Error Types:**
- `spelling` - Misspelled words
- `grammar` - Subject-verb agreement, wrong word choice, tense errors
- `punctuation` - Incorrect apostrophes, missing commas, periods
- `capitalization` - Proper nouns, names, places, days of week
- `styleguide` - Style guide violations
- `clarity` - Unclear or awkward phrasing

### Log Files

Logs are saved to `logs/xml-proofreader_YYYYMMDD_HHMMSS.log`:

```
2026-04-05 14:03:04,274 - INFO - [SUCCESS] Total errors found: 48
2026-04-05 14:03:04,274 - INFO - [SUCCESS] Paragraphs with errors: 14/14
2026-04-05 14:03:04,275 - INFO - [SUCCESS] Output written to: files\sample_input.corrected.xml
```

## Performance Metrics

The tool reports performance metrics after processing:

```
============================================================
PERFORMANCE METRICS:
  Total processing time: 5.43 seconds
  Memory used: 18.75 MB
  Peak memory: 165.00 MB
============================================================
```

**Performance Highlights:**
- **Parallel Processing**: 5x faster than sequential processing
- **14 paragraphs processed in ~5.5 seconds** (vs ~27 seconds sequential)
- **5 concurrent workers** handle multiple API calls simultaneously
- **Low memory footprint**: ~19 MB used, ~165 MB peak
- **Efficient caching**: Vector store reused across runs

## Examples

### Example 1: Basic Usage

```bash
# First run - create vector store
python xml_proofreader.py --input ./files/example_input.xml --lang en --style-guide ".\data\Style Guide.docx"

# Subsequent runs - use cached vector store
python xml_proofreader.py --input ./files/example_input.xml --lang en
```

### Example 2: Different Language

```bash
python xml_proofreader.py --input ./files/french_document.xml --lang fr --style-guide ".\data\French Style Guide.docx"
```

### Example 3: Logging Levels

```bash
# Minimal output (warnings/errors only)
python xml_proofreader.py --input ./files/example_input.xml --lang en --warning

# Progress updates (default)
python xml_proofreader.py --input ./files/example_input.xml --lang en --info

# Detailed debug output
python xml_proofreader.py --input ./files/example_input.xml --lang en --verbose
```

### Example 4: Update Style Guide

```bash
# Simply run with new style guide - it will automatically overwrite the old vector store
python xml_proofreader.py --input ./files/example_input.xml --lang en --style-guide ".\data\Updated Style Guide.docx"
```

**Note:** The script automatically overwrites the existing vector store when you provide `--style-guide`.

## How It Works

### 1. Style Guide Processing

1. **Load DOCX**: Reads style guide from `.docx` file
2. **Text Splitting**: Uses `RecursiveCharacterTextSplitter` with:
   - Chunk size: 600 characters
   - Chunk overlap: 100 characters
   - Separators: `\n\n`, `\n`, `. `, ` `, ``
3. **Embedding Creation**: Creates embeddings using OpenAI's `text-embedding-3-small`
4. **Vector Store**: Saves to FAISS vector store for fast retrieval

### 2. XML Processing (Parallel)

1. **Parse XML**: Loads XML file preserving structure
2. **Extract Paragraphs**: Finds all `<p>` elements
3. **Parallel Processing** (5 concurrent workers):
   - **ThreadPoolExecutor** creates pool of 5 worker threads
   - **All paragraphs submitted** to thread pool immediately
   - **For Each Paragraph** (in parallel):
     - Retrieve top-5 relevant style guide rules using FAISS similarity search
     - Build dynamic prompt with language instruction and relevant rules
     - Send to OpenAI GPT model for error detection
     - Parse JSON response with detected errors
   - **Results collected** as threads complete (out of order)
   - **XML updated in-place** maintaining original document order
   - Annotate text with `<error>` tags
   - Validate text-length invariance
4. **Save Output**: Write corrected XML to output file

**Parallel Processing Benefits:**
- 5 paragraphs processed simultaneously
- 5x faster total processing time
- Same accuracy and error detection
- XML element order preserved

### 3. Error Detection

The AI model detects:
- **Minimal error spans** (only incorrect words, not full sentences)
- **Short, concise reasons** (5-10 words)
- **Multiple error types** in a single pass

## Troubleshooting

### Issue: "No vector store found"

**Error:**
```
ValueError: No vector store found. Please run with --style-guide first to create it.
```

**Solution:**
Run with `--style-guide` argument to create the vector store:
```bash
python xml_proofreader.py --input ./files/example_input.xml --lang en --style-guide ".\data\Style Guide.docx"
```

### Issue: "API_KEY environment variable not set"

**Error:**
```
Error: API_KEY environment variable not set
```

**Solution:**
Create a `.env` file with your OpenAI API key:
```env
API_KEY=sk-your-api-key-here
MODEL=gpt-4o-mini
```

### Issue: "Style guide file not found"

**Error:**
```
Error: Style guide file not found: .\data\Style Guide.docx
```

**Solution:**
Ensure the style guide file exists at the specified path or provide the correct path.

### Issue: FAISS AVX2 Warning

**Warning:**
```
Could not load library with AVX2 support due to:
ModuleNotFoundError("No module named 'faiss.swigfaiss_avx2'")
```

**Solution:**
This is a harmless warning. FAISS will fall back to the standard version. Performance is not significantly affected.

### Issue: Text-length invariant violated

**Error:**
```
Text-length invariant violated for paragraph X
```

**Solution:**
This indicates the error annotation changed the text length. This is a bug in the annotation logic. Check the `annotate_text` method.

## Advanced Configuration

### Customize System Prompt

Edit `data/system_prompt_template.txt` to customize:
- Error detection rules
- Error span requirements
- Reason format
- Style guide emphasis

After editing, recreate the vector store:
```bash
python xml_proofreader.py --input ./files/example_input.xml --lang en --style-guide ".\data\Style Guide.docx"
```

### Adjust Chunking Parameters

Edit `xml_proofreader.py` line 79-84:

```python
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=600,        # Increase for larger chunks
    chunk_overlap=100,     # Increase for more context overlap
    length_function=len,
    separators=["\n\n", "\n", ". ", " ", ""]
)
```

### Change Retrieval Count

Edit `xml_proofreader.py` line 124:

```python
relevant_rules = self._get_relevant_rules(text, top_k=5)  # Change top_k value
```

### Adjust Parallel Workers

Edit `xml_proofreader.py` line 223:

```python
with ThreadPoolExecutor(max_workers=5) as executor:  # Increase for more parallelism
```

**Recommendations:**
- `max_workers=3`: Conservative (slower but safer)
- `max_workers=5`: Balanced (recommended)
- `max_workers=10`: Aggressive (faster but may hit API rate limits)

## API Costs

**Estimated costs per 1000 paragraphs:**
- Embeddings: ~$0.01 (one-time for style guide)
- GPT-4o-mini: ~$0.50-$1.00 (per run)
- GPT-4: ~$15-$30 (per run)

**Cost optimization:**
- Use cached vector store (avoid re-creating embeddings)
- Use `gpt-4o-mini` instead of `gpt-4`
- Parallel processing reduces wall-clock time (not API costs)
- Suppress HTTP logs with built-in filtering

## License

This project is provided as-is for evaluation purposes.

## Support

For issues or questions, please contact the development team.

---

**Last Updated:** April 5, 2026
