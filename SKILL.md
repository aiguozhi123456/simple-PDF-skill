---
name: simple-pdf-skill
description: PDF processing skill for creating, editing, extracting, and merging PDFs using Python libraries.
  
  Use when:
  - Creating new PDFs from data (reports, invoices, receipts)
  - Editing existing PDFs (adding highlights, annotations, watermarks, images)
  - Extracting text, tables, or images from PDFs
  - Merging, splitting, or manipulating PDF pages
  - Adding Chinese text to PDFs (requires font registration)
  - Batch processing PDF documents
  - Converting PDFs to images
  - Password protection or encryption
  
  NOT suitable for:
  - Complex HTML/CSS to PDF conversion (use WeasyPrint or pdfkit)
  - Rich media/interactive PDFs (use commercial libraries)
  - High-performance vector graphics
---

# Simple PDF Skill

Quick guide for PDF processing using Python libraries.

## Library Selection Guide

Choose the right library based on your task:

| Task | Library | Guide |
|------|---------|-------|
| Create new PDFs | **reportlab** | [reportlab-guide.md](references/reportlab-guide.md) |
| Edit existing PDFs | **PyMuPDF (fitz)** | [pymupdf-guide.md](references/pymupdf-guide.md) |
| Extract text/tables | **pdfplumber** or **PyMuPDF** | [pdfplumber-guide.md](references/pdfplumber-guide.md) |
| Merge/split PDFs | **PyMuPDF** or **pypdf** | [pymupdf-guide.md](references/pymupdf-guide.md) |
| Add annotations | **PyMuPDF** | [pymupdf-guide.md](references/pymupdf-guide.md) |
| Extract images | **PyMuPDF** | [pymupdf-guide.md](references/pymupdf-guide.md) |
| Render to images | **pypdfium2** | [pypdfium2-guide.md](references/pypdfium2-guide.md) |
| Password protection | **pypdf** | [pypdf-guide.md](references/pypdf-guide.md) |
| Generate charts | **matplotlib + reportlab** | [chart-guide.md](references/chart-guide.md) |

## Quick Start Workflow

### 1. Identify the Task Type

**Creating PDFs:**
- Use `reportlab` for new documents from scratch
- See [reportlab-guide.md](references/reportlab-guide.md) for complete API

**Editing Existing PDFs:**
- Use `PyMuPDF (fitz)` for any modifications
- Common edits: highlights, annotations, watermarks, merging, splitting
- See [pymupdf-guide.md](references/pymupdf-guide.md)

**Extracting Content:**
- Text extraction: `pdfplumber` or `PyMuPDF`
- Table extraction: `pdfplumber` (better for tables)
- Image extraction: `PyMuPDF`
- See [pdfplumber-guide.md](references/pdfplumber-guide.md) or [pymupdf-guide.md](references/pymupdf-guide.md)

### 2. Special Considerations

**Chinese Text Support:**
- CRITICAL: Default fonts do not support Chinese
- Must register Chinese font before use in reportlab
- See [reportlab-guide.md](references/reportlab-guide.md) → Chinese Font Support section
- Recommended fonts: WQY Microhei (4.4MB), Noto Sans SC (15MB)

**Performance:**
- For large PDFs, process in chunks
- Use `fitz` (PyMuPDF) for best performance on editing tasks
- Use `pdfplumber` for reliable text extraction

### 3. Implementation Reference

For implementation patterns and examples:
- **Code patterns**: [PATTERNS.md](references/PATTERNS.md)
- **Complete examples**: [EXAMPLES.md](references/EXAMPLES.md)
- **Real-world scenarios**: [SCENARIOS.md](references/SCENARIOS.md)
- **Workflow details**: [WORKFLOWS.md](references/WORKFLOWS.md)

## Installation

Install required libraries:
```bash
pip install reportlab
pip install pymupdf
pip install pdfplumber
pip install pypdf
pip install pypdfium2
pip install fonttools  # For TTF font extraction
```

For advanced features (OCR, CLI tools):
```bash
# For OCR (scanned PDFs)
pip install pytesseract pdf2image
sudo apt-get install tesseract-ocr

# For command-line tools
sudo apt-get install poppler-utils
sudo apt-get install qpdf
```

## Key Rules

- **reportlab**: Canvas coordinates (0,0 at bottom-left), use `Pt()` for font sizes, `Inch()` for positioning
- **PyMuPDF**: Uses RGB tuples (0-1 range), not 0-255
- **Always**: Call `save()` to finalize documents, close documents to free resources
- **Chinese fonts**: ALWAYS register Chinese fonts before using Chinese text in reportlab
- **Large PDFs**: Process in chunks to avoid memory issues
- **Encrypted PDFs**: Handle gracefully with proper password management
