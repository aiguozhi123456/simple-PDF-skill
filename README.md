# Simple PDF Skill

PDF processing skill package: Code examples, best practices, and workflows for creating, editing, extracting, and merging PDFs using Python libraries.

## Core Libraries

- **reportlab** - Create new PDFs with text, shapes, and layouts
- **PyMuPDF (fitz)** - Preferred editing library: page editing, content injection, merge, split, annotations
- **pypdf** - PDF manipulation: merge, split, rotate, encrypt
- **pdfplumber** - Extract text and tables from PDFs
- **pypdfium2** - Render PDF pages to images

## Quick Start

### Basic PDF Reading
```python
from pypdf import PdfReader

reader = PdfReader("document.pdf")
print(f"Pages: {len(reader.pages)}")
```

### Basic PDF Creation
```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

c = canvas.Canvas("hello.pdf", pagesize=letter)
c.drawString(100, 500, "Hello World!")
c.save()
```

### Basic PDF Editing
```python
import fitz  # PyMuPDF

doc = fitz.open("document.pdf")
page = doc[0]

# Draw highlight
rect = fitz.Rect(100, 400, 400, 450)
page.draw_rect(rect, color=(1, 1, 0), fill=(1, 1, 0.9))

# Add annotation
annot = page.add_text_annot(fitz.Point(410, 425), "Important!")
annot.update()
doc.save("edited.pdf")
```

## Quick Reference

| Task | Best Tool |
|------|-----------|
| Create PDFs | reportlab |
| Edit existing PDFs | PyMuPDF (fitz) |
| Merge PDFs | PyMuPDF or pypdf |
| Split PDFs | PyMuPDF or pypdf |
| Extract text | PyMuPDF or pdfplumber |
| Extract tables | PyMuPDF or pdfplumber |
| Extract images | PyMuPDF |
| Render to images | pypdfium2 |
| Password protection | pypdf |
| Rotate/crop pages | PyMuPDF or pypdf |

## Additional Files

- PATTERNS.md - Core code patterns and functions
- EXAMPLES.md - Complete working examples
- REFERENCE.md - Colors, fonts, quick reference
- SCENARIOS.md - Real-world use cases and solutions
- WORKFLOWS.md - Library selection and cross-library workflows
- reportlab-guide.md - Full API documentation for PDF creation
- chart-guide.md - Chart and graphics generation
- pymupdf-guide.md - Advanced PDF editing and manipulation
- pypdf-guide.md - PDF merging, splitting, bookmarks, forms, and metadata
- pdfplumber-guide.md - Reading PDFs documentation
- pypdfium2-guide.md - PDF to image rendering

## License

Apache 2.0 - See LICENSE.txt for complete terms.