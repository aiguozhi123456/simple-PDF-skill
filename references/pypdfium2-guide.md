# pypdfium2 API Guide

## Overview

pypdfium2 is a Python binding for PDFium (Chromium's PDF library). It's excellent for fast PDF rendering, image generation, and serves as a PyMuPDF replacement.

```python
import pypdfium2 as pdfium

pdf = pdfium.PdfDocument("document.pdf")
```

## Installation

```bash
pip install pypdfium2
```

## Loading PDFs

```python
import pypdfium2 as pdfium

# Open PDF file
pdf = pdfium.PdfDocument("document.pdf")

# Open from bytes
with open("document.pdf", "rb") as f:
    pdf_data = f.read()
    pdf = pdfium.PdfDocument(pdf_data)

# Get page count
num_pages = len(pdf)
print(f"Total pages: {num_pages}")
```

## Page Access

```python
# Access individual pages (0-indexed)
first_page = pdf[0]
last_page = pdf[-1]

# Iterate through all pages
for i, page in enumerate(pdf):
    print(f"Page {i}: {page.get_width()}x{page.get_height()}")
```

## Rendering to Images

### Basic Rendering

```python
import pypdfium2 as pdfium
from PIL import Image

pdf = pdfium.PdfDocument("document.pdf")
page = pdf[0]

# Render page to bitmap
bitmap = page.render(
    scale=2.0,  # Scale factor (higher = better quality)
    rotation=0   # No rotation
)

# Convert to PIL Image
img = bitmap.to_pil()
img.save("page_1.png", "PNG")
```

### Advanced Rendering Options

```python
# Render with specific options
bitmap = page.render(
    scale=3.0,              # Scale factor (default: 1.0)
    rotation=0,              # Rotation: 0, 90, 180, 270
    crop=(0, 0, width, height),  # Crop box: (left, bottom, right, top)
    colour=(255, 255, 255, 255), # Background color for transparent areas
    alpha=True,               # Enable alpha channel
    optimise_mode=pdfium.BitmapOptimiseMode.LCD  # Optimization for LCD displays
)

# Save as different formats
img = bitmap.to_pil()
img.save("page.jpg", "JPEG", quality=95)
```

### Batch Rendering

```python
import pypdfium2 as pdfium
from PIL import Image

pdf = pdfium.PdfDocument("document.pdf")

# Render all pages
for i, page in enumerate(pdf):
    bitmap = page.render(scale=2.0)
    img = bitmap.to_pil()
    img.save(f"page_{i+1}.png", "PNG")
    print(f"Rendered page {i+1}")
```

## Text Extraction

```python
# Extract text from page
page = pdf[0]
text = page.get_text()

print(f"Page text: {text}")

# Extract text from all pages
for i, page in enumerate(pdf):
    text = page.get_text()
    print(f"--- Page {i+1} ---")
    print(text)
```

### Text with Positioning

```python
# Get text with bounding boxes
text_page = page.get_textpage()

# Iterate through text segments
for i in range(text_page.count_chars()):
    char = text_page.get_char(i)
    
    # Get character properties
    origin = char.origin  # (x, y) position
    char_box = char.bbox  # Bounding box
    char_unicode = char.u  # Unicode character
    
    print(f"Char: {char_unicode} at {origin}")
```

## Page Manipulation

```python
# Get page dimensions
page = pdf[0]
width = page.get_width()
height = page.get_height()
print(f"Page size: {width} x {height} points")

# Get page rotation
rotation = page.get_rotation()
print(f"Rotation: {rotation} degrees")

# Check if page has any content
has_content = len(page.get_text()) > 0
```

## Searching and Navigation

```python
# Find text on pages
for i, page in enumerate(pdf):
    text = page.get_text()
    if "search term" in text:
        print(f"Found on page {i+1}")

# Get page label (if present)
page = pdf[0]
label = page.get_label()
if label:
    print(f"Page label: {label}")
```

## Working with Forms

```python
# Check if page has form fields
page = pdf[0]
form_objects = page.get_objects(filter=[pdfium.FPDF_FORMOBJ])

for form_obj in form_objects:
    print(f"Form field: {form_obj}")
```

## PDF Metadata

```python
# Get PDF metadata
pdf = pdfium.PdfDocument("document.pdf")

# Access metadata dictionary
metadata = pdf.get_pdf_version()
print(f"PDF Version: {metadata}")

# Get page mode
for page in pdf:
    print(f"Page mode: {page.get_mode()}")
```

## Performance Tips

### High-Quality Rendering

```python
# Use higher scale for better quality
bitmap = page.render(scale=3.0)  # 3x resolution

# Optimize for LCD displays
bitmap = page.render(
    scale=2.0,
    optimise_mode=pdfium.BitmapOptimiseMode.LCD
)
```

### Memory Management

```python
# Process pages one at a time to save memory
pdf = pdfium.PdfDocument("large_file.pdf")

for i in range(len(pdf)):
    page = pdf[i]
    bitmap = page.render(scale=1.5)
    img = bitmap.to_pil()
    img.save(f"page_{i+1}.png", "PNG")
    
    # Page and bitmap are freed after each iteration
```

### Parallel Processing

```python
from concurrent.futures import ThreadPoolExecutor
import pypdfium2 as pdfium

def render_page(page_num):
    pdf = pdfium.PdfDocument("document.pdf")
    page = pdf[page_num]
    bitmap = page.render(scale=2.0)
    img = bitmap.to_pil()
    img.save(f"page_{page_num+1}.png", "PNG")

# Render pages in parallel
with ThreadPoolExecutor(max_workers=4) as executor:
    for i in range(len(pdf)):
        executor.submit(render_page, i)
```

## Common Use Cases

### Create Thumbnails

```python
import pypdfium2 as pdfium
from PIL import Image

pdf = pdfium.PdfDocument("document.pdf")

# Create thumbnail for each page
for i, page in enumerate(pdf):
    bitmap = page.render(scale=0.5)  # Lower scale for thumbnails
    img = bitmap.to_pil()
    img.save(f"thumb_{i+1}.png", "PNG", optimize=True)
```

### Convert PDF to Images

```python
import pypdfium2 as pdfium
from PIL import Image

def pdf_to_images(pdf_path, output_dir, scale=2.0):
    pdf = pdfium.PdfDocument(pdf_path)
    
    for i, page in enumerate(pdf):
        bitmap = page.render(scale=scale)
        img = bitmap.to_pil()
        
        # Save as JPEG for smaller file size
        img.save(f"{output_dir}/page_{i+1}.jpg", "JPEG", quality=85)
        
    print(f"Converted {len(pdf)} pages to images")

# Usage
pdf_to_images("document.pdf", "output_images", scale=2.0)
```

### Extract Text with Page Numbers

```python
import pypdfium2 as pdfium

pdf = pdfium.PdfDocument("document.pdf")

full_text = ""
for i, page in enumerate(pdf):
    text = page.get_text()
    full_text += f"--- Page {i+1} ---\n"
    full_text += text + "\n"

print(full_text)
```

## Comparison with Other Libraries

| Feature | pypdfium2 | pdfplumber | pypdf | PyMuPDF |
|----------|------------|-----------|-------|----------|
| Rendering speed | ⭐⭐⭐⭐ | ⭐⭐ | ⭐ | ⭐⭐⭐⭐ |
| Text extraction | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Form filling | ⭐ | ⭐⭐ | ⭐ | ⭐⭐⭐ |
| Image quality | ⭐⭐⭐⭐ | ⭐ | ⭐ | ⭐⭐⭐⭐ |
| Memory usage | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Platform support | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |

## Best Practices

1. **Use pypdfium2 for rendering** - Fastest library for PDF to image conversion
2. **Use pdfplumber for text extraction** - Better layout understanding and table extraction
3. **Use pypdf for manipulation** - Best for merge/split/rotate operations
4. **Close PDFs properly** - Use context managers or explicit close
5. **Optimize scale based on use** - Lower scale for previews, higher for final output
6. **Handle exceptions** - PDFs can be corrupted or password-protected

## Error Handling

```python
import pypdfium2 as pdfium

try:
    pdf = pdfium.PdfDocument("document.pdf")
    
    for page in pdf:
        text = page.get_text()
        print(text)
        
except pdfium.PdfiumError as e:
    print(f"PDFium error: {e}")
except FileNotFoundError:
    print("PDF file not found")
except Exception as e:
    print(f"Unexpected error: {e}")
```

## Limitations

- Does not support creating new PDFs (use reportlab or pypdf)
- Limited form field manipulation compared to specialized libraries
- May have issues with very old or non-standard PDFs
- Platform-dependent (Windows/Linux/macOS support varies)
