# Pdfplumber API Guide

## Overview

pdfplumber is a Python library for extracting text, tables, and other data from PDF files. It's built on top of pdfminer.six and provides a more Pythonic API.

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    # ... operations ...
    pass
```

## Opening PDFs

```python
import pdfplumber

# Open with context manager (recommended)
with pdfplumber.open("file.pdf") as pdf:
    # Process PDF
    pass

# Open without context manager
pdf = pdfplumber.open("file.pdf")
# ... operations ...
pdf.close()

# Open from file-like object
with open("file.pdf", "rb") as f:
    pdf = pdfplumber.open(f)
```

## PDF Properties

```python
with pdfplumber.open("file.pdf") as pdf:
    # Basic properties
    num_pages = len(pdf.pages)        # Number of pages
    metadata = pdf.metadata           # Metadata dictionary
    
    # Get specific page (0-indexed)
    first_page = pdf.pages[0]
    last_page = pdf.pages[-1]
    
    # Iterate through pages
    for page in pdf.pages:
        pass
```

## Metadata

```python
with pdfplumber.open("file.pdf") as pdf:
    metadata = pdf.metadata
    
    # Common metadata fields
    title = metadata.get("Title", "")
    author = metadata.get("Author", "")
    creator = metadata.get("Creator", "")
    producer = metadata.get("Producer", "")
    creation_date = metadata.get("CreationDate", "")
    modification_date = metadata.get("ModDate", "")
```

## Page Properties

```python
page = pdf.pages[0]

# Dimensions
page_width = page.width          # Page width in points
page_height = page.height        # Page height in points

# Bounding box (x0, top, x1, bottom)
bbox = page.bbox

# Parent PDF reference
parent_pdf = page.pdf

# Page number (1-indexed)
page_number = page.page_number
```

## Extracting Text

### Full Page Text

```python
page = pdf.pages[0]

# Extract all text as string
text = page.extract_text()

# Extract text with layout (preserves some formatting)
text = page.extract_text(layout=True)

# Extract words with positions
words = page.extract_words()

# Extract characters with positions
chars = page.extract_chars()
```

### Extracted Text Format

```python
text = """
Line 1 of the PDF document
Line 2 of the PDF document
Line 3 of the PDF document
"""
```

### Words Format

```python
words = [
    {
        "x0": 50,
        "x1": 100,
        "top": 700,
        "bottom": 690,
        "text": "Hello",
        "fontname": "Helvetica",
        "size": 12,
        "non_stroking_color": (0, 0, 0)
    },
    # ... more words
]
```

### Filtered Text Extraction

```python
# Extract words within a bounding box
bbox = (50, 600, 400, 700)
words = page.extract_words(bbox=bbox)

# Extract text with custom tolerance
text = page.extract_text(x_tolerance=2, y_tolerance=2)
```

## Extracting Tables

### Simple Table Extraction

```python
page = pdf.pages[0]

# Extract first table found
table = page.extract_table()

# Extract all tables on page
tables = page.extract_tables()
```

### Table Format

```python
table = [
    ["Header 1", "Header 2", "Header 3"],
    ["Row 1 Col 1", "Row 1 Col 2", "Row 1 Col 3"],
    ["Row 2 Col 1", "Row 2 Col 2", "Row 2 Col 3"]
]
```

### Advanced Table Extraction

```python
# Extract table with custom settings
table = page.extract_table({
    "vertical_strategy": "text",      # or "lines", "explicit"
    "horizontal_strategy": "text",   # or "lines", "explicit"
    "min_words_vertical": 3,
    "min_words_horizontal": 1,
    "intersection_tolerance": 3,
    "join_tolerance": 3
})

# Extract table within bounding box
bbox = (50, 100, 500, 700)
table = page.extract_table(table_settings={"vertical_strategy": "lines", 
                                           "bbox": bbox})
```

### Table Settings

```python
table_settings = {
    # Vertical detection strategy
    "vertical_strategy": "text",      # "text", "lines", "explicit"
    
    # Horizontal detection strategy
    "horizontal_strategy": "text",   # "text", "lines", "explicit"
    
    # Explicit vertical lines (if using "explicit" strategy)
    "explicit_vertical_lines": [100, 200, 300, 400],
    
    # Explicit horizontal lines
    "explicit_horizontal_lines": [600, 500, 400, 300],
    
    # Snap tolerance
    "snap_tolerance": 5,
    
    # Join tolerance
    "join_tolerance": 3,
    
    # Edge minimum length
    "edge_min_length": 3,
    
    # Words required to consider vertical text
    "min_words_vertical": 3,
    
    # Words required to consider horizontal text
    "min_words_horizontal": 1,
    
    # Keep blank rows
    "keep_blank_rows": False,
    
    # Text inside cells
    "text": page.extract_text(),
    
    # Bounding box
    "bbox": (x0, top, x1, bottom)
}
```

## Extracting Images

```python
page = pdf.pages[0]

# Extract images (if available)
images = page.images

# Image format
for image in images:
    image_data = {
        "x0": 100,
        "x1": 300,
        "top": 600,
        "bottom": 400,
        "width": 200,
        "height": 200,
        "stream": ...  # PDF stream object
    }
```

## Visual Debugging

```python
# Draw elements on the page for debugging
im = page.to_image()

# Draw all words
im.draw_words()

# Draw all tables
im.draw_tables()

# Draw all lines
im.draw_lines()

# Draw specific elements
im.draw_rects(page.chars)

# Save the debug image
im.save("debug_page.png")

# Display inline (Jupyter)
im
```

## Page Cropping

```python
page = pdf.pages[0]

# Crop to bounding box (returns new Page object)
bbox = (50, 100, 400, 600)
cropped = page.crop(bbox)

# Crop relative to page
cropped = page.crop((page.width/4, page.height/4, 
                     3*page.width/4, 3*page.height/4))

# Crop within bbox (clip to page bounds)
cropped = page.crop(bbox, relative=False, strict=True)
```

## Finding Specific Content

### Search for Text

```python
# Search for text on page
search_results = page.search("search term")

# Search with regex
import re
search_results = page.search(re.compile(r"\d{3}-\d{3}-\d{4}"))

# Search results format
for result in search_results:
    print(result)  # {'x0': ..., 'x1': ..., 'top': ..., 'bottom': ..., 'text': ...}
```

### Find Elements by Position

```python
# Find all words above a certain Y coordinate
words_above = [w for w in page.extract_words() if w["top"] > 600]

# Find all words within a column
column_words = [w for w in page.extract_words() 
                if 100 < w["x0"] < 300]
```

## Combining Multiple Operations

### Extract Table with Text Context

```python
def extract_table_with_header(page, header_text):
    # Find header position
    header_results = page.search(header_text)
    if not header_results:
        return None
    
    header = header_results[0]
    
    # Define bounding box below header
    bbox = (0, 0, page.width, header["top"])
    
    # Crop and extract table
    cropped = page.crop(bbox)
    return cropped.extract_table()
```

### Extract Multi-Column Layout

```python
def extract_columns(page, col_width, gap=20):
    words = page.extract_words()
    
    # Sort words by position
    words_sorted = sorted(words, key=lambda w: (w["top"], w["x0"]))
    
    # Separate into columns
    col1_words = [w for w in words if w["x0"] < col_width]
    col2_words = [w for w in words if w["x0"] > col_width + gap]
    
    return col1_words, col2_words
```

## Performance Optimization

```python
# Lazy loading - only load needed pages
with pdfplumber.open("large.pdf") as pdf:
    # Get metadata without loading all pages
    metadata = pdf.metadata
    
    # Load only specific page
    page = pdf.pages[0]
    text = page.extract_text()

# Limit text extraction
text = page.extract_text(x_tolerance=3, y_tolerance=3)

# Skip blank pages
for page in pdf.pages:
    if page.extract_text().strip():
        # Process non-blank page
        pass
```

## Error Handling

```python
import pdfplumber

try:
    with pdfplumber.open("file.pdf") as pdf:
        page = pdf.pages[0]
        text = page.extract_text()
        
except FileNotFoundError:
    print("PDF file not found")
    
except pdfplumber.pdfminer.PDFSyntaxError:
    print("Invalid PDF format")
    
except Exception as e:
    print(f"Error processing PDF: {e}")
```

## Common Use Cases

### Extract All Text from PDF

```python
def extract_all_text(pdf_path):
    with pdfplumber.open(pdf_path) as pdf:
        full_text = ""
        for page in pdf.pages:
            text = page.extract_text()
            if text:
                full_text += text + "\n"
        return full_text
```

### Extract All Tables from PDF

```python
def extract_all_tables(pdf_path):
    with pdfplumber.open(pdf_path) as pdf:
        all_tables = []
        for page in pdf.pages:
            tables = page.extract_tables()
            if tables:
                all_tables.extend(tables)
        return all_tables
```

### Find Page with Keyword

```python
def find_page_with_keyword(pdf_path, keyword):
    with pdfplumber.open(pdf_path) as pdf:
        for i, page in enumerate(pdf.pages):
            text = page.extract_text()
            if keyword.lower() in text.lower():
                return i + 1  # Return page number (1-indexed)
    return None
```

### Extract Data from Invoice

```python
def extract_invoice_data(pdf_path):
    with pdfplumber.open(pdf_path) as pdf:
        page = pdf.pages[0]
        
        # Find invoice number
        invoice_results = page.search("Invoice")
        if invoice_results:
            # Extract nearby text
            words = page.extract_words()
            # Find invoice number logic here
            pass
        
        # Extract table (line items)
        table = page.extract_table()
        
        return {
            "invoice_num": "...",
            "table": table
        }
```

## Best Practices

1. **Use context managers** (`with pdfplumber.open()`) for automatic cleanup
2. **Check for None** before accessing table results
3. **Crop strategically** to isolate sections before extraction
4. **Use visual debugging** (`page.to_image()`) for complex layouts
5. **Adjust tolerance settings** for better table detection
6. **Process pages one at a time** for large PDFs to manage memory
7. **Validate extracted data** before using it
8. **Handle errors gracefully** for malformed PDFs
9. **Use regex** for pattern matching when searching
10. **Test on multiple samples** to ensure robustness

## Limitations

- **Scanned PDFs**: Requires OCR (pdfplumber can't extract text from images)
- **Complex layouts**: May require manual cropping and multiple extractions
- **Rotated text**: May not be properly extracted
- **Password-protected**: Requires decryption before processing
- **Forms**: Cannot extract form field data
- **Annotations**: Cannot extract PDF annotations

## Advanced Features

### Custom Extraction Logic

```python
def custom_extract(page, bbox, filter_func):
    cropped = page.crop(bbox)
    elements = cropped.extract_words()
    return [e for e in elements if filter_func(e)]
```

### Multi-Column Table Detection

```python
def detect_multi_column_tables(page):
    # Detect vertical lines
    vlines = page.extract_text_lines()
    # Analyze for column separation
    # Return column positions
    pass
```

### Text Alignment Detection

```python
def detect_text_alignment(page):
    words = page.extract_words()
    # Analyze x positions for alignment patterns
    # Return alignment info
    pass
```

## Integration with Other Libraries

### With pandas

```python
import pandas as pd
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    page = pdf.pages[0]
    table = page.extract_table()
    
    # Convert to DataFrame
    df = pd.DataFrame(table[1:], columns=table[0])
```

### With regex

```python
import re
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    page = pdf.pages[0]
    text = page.extract_text()
    
    # Find phone numbers
    phones = re.findall(r"\d{3}-\d{3}-\d{4}", text)
```

### With PIL/Pillow

```python
from PIL import Image
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    page = pdf.pages[0]
    
    # Convert page to image
    im = page.to_image()
    pil_img = im.original
    
    # Process with Pillow
    pil_img.save("page.png")
```
