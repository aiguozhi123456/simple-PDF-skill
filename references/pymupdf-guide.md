# PyMuPDF (fitz) Guide

## Overview

PyMuPDF (imported as `fitz`) is a high-performance library for PDF document manipulation. It provides advanced editing capabilities including drawing shapes, adding annotations, extracting content and modifying existing PDF pages directly.

**Key Features:**
- Draw shapes (rectangles, circles, lines, polygons) on pages
- Add text annotations and highlights
- Extract text, images, and tables
- Merge, split, and crop PDFs
- Rotate and transform pages
- Add watermarks and stamps
- Modify page content directly

```python
import fitz  # PyMuPDF
```

## Installation

```bash
pip install pymupdf
```

## Basic Operations

### Open and Save PDF

```python
import fitz

# Open PDF document
doc = fitz.open("input.pdf")

# Get document info
print(f"Total pages: {len(doc)}")
print(f"Metadata: {doc.metadata}")

# Access a page
page = doc[0]  # First page (0-indexed)
rect = page.rect  # Page rectangle (x0, y0, x1, y1)
print(f"Page size: {rect.width} x {rect.height}")

# Save document
doc.save("output.pdf")

# Close document
doc.close()
```

### Get Page Information

```python
doc = fitz.open("document.pdf")
page = doc[0]

# Page dimensions
rect = page.rect
print(f"Width: {rect.width}, Height: {rect.height}")
print(f"Top-left: ({rect.x0}, {rect.y0})")
print(f"Bottom-right: ({rect.x1}, {rect.y1})")

# Page rotation
print(f"Rotation: {page.rotation}")

# Page text
text = page.get_text()
print(text)
```

## Drawing Shapes

### Draw Rectangle

```python
import fitz

doc = fitz.open("input.pdf")
page = doc[0]

# Draw rectangle (filled)
rect = fitz.Rect(50, 50, 200, 150)
page.draw_rect(rect, color=(1, 0, 0), fill=(1, 0.9, 0.9), width=2)

# Draw rectangle (outline only)
rect2 = fitz.Rect(50, 200, 200, 300)
page.draw_rect(rect2, color=(0, 0, 1), width=3)

doc.save("output.pdf")
```

### Draw Circle

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Draw circle (filled)
center = fitz.Point(300, 200)
radius = 50
page.draw_circle(center, radius, color=(0, 1, 0), fill=(0.9, 1, 0.9), width=2)

# Draw circle (outline only)
center2 = fitz.Point(450, 200)
page.draw_circle(center2, radius, color=(0, 0, 1), width=3)

doc.save("output.pdf")
```

### Draw Line

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Draw straight line
start = fitz.Point(50, 50)
end = fitz.Point(300, 200)
page.draw_line(start, end, color=(1, 0, 0), width=2)

# Draw multiple lines
points = [fitz.Point(50, 300), fitz.Point(150, 200), fitz.Point(250, 300)]
for i in range(len(points) - 1):
    page.draw_line(points[i], points[i+1], color=(0, 0, 1), width=2)

doc.save("output.pdf")
```

### Draw Arrow (using lines)

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Draw arrow
arrow_start = fitz.Point(50, 100)
arrow_end = fitz.Point(200, 100)

# Main line
page.draw_line(arrow_start, arrow_end, color=(1, 0.4, 0), width=3)

# Arrow head lines
page.draw_line(arrow_end, fitz.Point(arrow_end.x - 10, arrow_end.y - 5), color=(1, 0.4, 0), width=3)
page.draw_line(arrow_end, fitz.Point(arrow_end.x - 10, arrow_end.y + 5), color=(1, 0.4, 0), width=3)

doc.save("output.pdf")
```

## Text Operations

### Insert Text

```python
import fitz

doc = fitz.open("input.pdf")
page = doc[0]

# Simple text
page.insert_text(fitz.Point(100, 500), "Hello World", fontsize=12, color=(0, 0, 0))

# Text with different sizes and colors
page.insert_text(fitz.Point(100, 450), "Large Title", fontsize=24, color=(1, 0, 0))
page.insert_text(fitz.Point(100, 420), "Subtitle", fontsize=18, color=(0, 0, 1))
page.insert_text(fitz.Point(100, 390), "Regular text", fontsize=12, color=(0.5, 0.5, 0.5))

doc.save("output.pdf")
```

### Add Text Box

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Define text box rectangle
rect = fitz.Rect(50, 50, 500, 200)

# Insert text in box (with wrapping)
text = "This is a long text that will be automatically wrapped within the defined rectangle."
page.insert_textbox(rect, text, fontsize=12, color=(0, 0, 0))

doc.save("output.pdf")
```

### Add Freehand Text Annotation

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Add text annotation (sticky note style)
annot = page.add_text_annot(fitz.Point(100, 300), "This is an annotation!")

# Update annotation properties
annot.update(fontsize=10)

# Add multiline annotation
multiline_text = """Important Note:
- Check this section
- Review carefully
- Follow instructions"""
annot2 = page.add_text_annot(fitz.Point(100, 350), multiline_text)

doc.save("output.pdf")
```

## Annotations

### Add Highlight Annotation

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Highlight a specific area
rect = fitz.Rect(100, 400, 400, 450)
annot = page.add_highlight_annot(rect)
annot.set_colors(stroke=(1, 1, 0))  # Yellow highlight
annot.update()

doc.save("output.pdf")
```

### Add Other Annotation Types

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Underline annotation
rect1 = fitz.Rect(100, 300, 400, 320)
annot1 = page.add_underline_annot(rect1)
annot1.update()

# Strikeout annotation
rect2 = fitz.Rect(100, 250, 400, 270)
annot2 = page.add_strikeout_annot(rect2)
annot2.update()

doc.save("output.pdf")
```

## Page Manipulation

### Rotate Page

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Rotate page clockwise (90, 180, 270)
page.set_rotation(90)

doc.save("rotated.pdf")
```

### Crop Page

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Define crop rectangle (left, bottom, right, top)
crop_rect = fitz.Rect(50, 50, 550, 750)
page.set_cropbox(crop_rect)

doc.save("cropped.pdf")
```

### Merge PDFs

```python
# Create new document
merged_doc = fitz.open()

# Merge multiple PDFs
for pdf_file in ["doc1.pdf", "doc2.pdf", "doc3.pdf"]:
    doc = fitz.open(pdf_file)
    for page in doc:
        merged_doc.insert_pdf(doc)
    doc.close()

merged_doc.save("merged.pdf")
merged_doc.close()
```

### Split PDF

```python
doc = fitz.open("input.pdf")

# Split into individual pages
for i, page in enumerate(doc):
    new_doc = fitz.open()
    new_doc.insert_pdf(doc, from_page=i, to_page=i)
    new_doc.save(f"page_{i+1}.pdf")
    new_doc.close()

doc.close()
```

### Delete Pages

```python
doc = fitz.open("input.pdf")

# Delete page at index 2 (third page)
doc.delete_page(2)

# Delete multiple pages (delete pages 1 and 2)
doc.delete_pages([0, 1])

doc.save("deleted.pdf")
```

## Content Extraction

### Extract Text

```python
doc = fitz.open("document.pdf")

# Extract all text
for page in doc:
    text = page.get_text()
    print(text)

# Extract text from specific page
page = doc[0]
text = page.get_text("text")  # Plain text
text_with_layout = page.get_text("blocks")  # With layout
text_dict = page.get_text("dict")  # As dictionary

doc.close()
```

### Extract Images

```python
doc = fitz.open("document.pdf")

for page_num, page in enumerate(doc):
    image_list = page.get_images()
    print(f"Page {page_num + 1}: {len(image_list)} images")
    
    for img_index, img in enumerate(image_list):
        xref = img[0]
        base_image = doc.extract_image(xref)
        image_data = base_image["image"]
        image_ext = base_image["ext"]
        
        # Save image
        with open(f"image_page{page_num+1}_{img_index+1}.{image_ext}", "wb") as f:
            f.write(image_data)

doc.close()
```

### Extract Tables

```python
doc = fitz.open("document.pdf")

for page in doc:
    tables = page.find_tables()
    print(f"Found {len(tables.tables)} tables on this page")
    
    for i, table in enumerate(tables):
        print(f"Table {i+1}:")
        print(table.to_pandas())

doc.close()
```

## Advanced Operations

### Add Watermark

```python
import fitz

# Open documents
doc = fitz.open("document.pdf")
watermark = fitz.open("watermark.pdf")
watermark_page = watermark[0]

# Apply watermark to all pages
for page in doc:
    page.show_pdf_page(page.rect, watermark, 0)

doc.save("watermarked.pdf")
doc.close()
watermark.close()
```

### Add Stamp

```python
doc = fitz.open("input.pdf")
page = doc[0]

# Define stamp rectangle
stamp_rect = fitz.Rect(450, 750, 550, 800)

# Draw stamp background
page.draw_rect(stamp_rect, color=(1, 0.9, 0.9), fill=(1, 0.9, 0.9), width=2)

# Add stamp text
page.insert_text(
    fitz.Point(stamp_rect.x0 + 10, stamp_rect.y0 + 25),
    "APPROVED",
    fontsize=10,
    color=(0.8, 0, 0)
)

doc.save("stamped.pdf")
```

### Search and Highlight Text

```python
doc = fitz.open("document.pdf")

for page in doc:
    # Search for text
    text_instances = page.search_for("important")
    
    # Highlight all instances
    for rect in text_instances:
        annot = page.add_highlight_annot(rect)
        annot.update()

doc.save("highlighted.pdf")
```

### Extract Text by Coordinates

```python
doc = fitz.open("document.pdf")
page = doc[0]

# Define rectangle for text extraction
rect = fitz.Rect(100, 100, 500, 500)

# Extract text within rectangle
text = page.get_textbox(rect)
print(text)

doc.close()
```

## Color Management

PyMuPDF uses RGB color tuples with values 0-1:

```python
# Colors
red = (1, 0, 0)
green = (0, 1, 0)
blue = (0, 0, 1)
yellow = (1, 1, 0)
black = (0, 0, 0)
white = (1, 1, 1)
gray = (0.5, 0.5, 0.5)

# Custom colors
orange = (1, 0.5, 0)
purple = (0.5, 0, 0.5)

# Use in drawing
page.draw_rect(rect, color=blue, fill=(0.9, 0.9, 1))
page.insert_text(point, "Text", color=red)
```

## Common Patterns

### Batch Process Multiple PDFs

```python
import glob
import fitz

for pdf_file in glob.glob("*.pdf"):
    doc = fitz.open(pdf_file)
    
    # Process each page
    for page in doc:
        # Add watermark or stamp
        stamp_rect = fitz.Rect(page.rect.width - 100, page.rect.height - 50, 
                               page.rect.width - 10, page.rect.height - 10)
        page.draw_rect(stamp_rect, color=(1, 0.9, 0.9), fill=(1, 0.9, 0.9))
        page.insert_text(
            fitz.Point(stamp_rect.x0 + 10, stamp_rect.y0 + 25),
            "PROCESSED",
            fontsize=8,
            color=(0.8, 0, 0)
        )
    
    # Save processed file
    output_file = f"processed_{pdf_file}"
    doc.save(output_file)
    doc.close()
```

### Extract All Text to File

```python
import fitz

doc = fitz.open("document.pdf")

with open("extracted_text.txt", "w", encoding="utf-8") as f:
    for page_num, page in enumerate(doc, 1):
        f.write(f"--- Page {page_num} ---\n")
        f.write(page.get_text())
        f.write("\n\n")

doc.close()
```

### Create PDF from Images

```python
import fitz
import glob

doc = fitz.open()

# Add images as pages
for img_file in sorted(glob.glob("images/*.jpg")):
    img_doc = fitz.open(img_file)
    doc.insert_pdf(img_doc)
    img_doc.close()

doc.save("images_to_pdf.pdf")
doc.close()
```

## Error Handling

```python
import fitz

try:
    doc = fitz.open("document.pdf")
    
    # Process document
    for page in doc:
        text = page.get_text()
        # Process text...
    
    doc.save("output.pdf")
    
except fitz.FileNotFoundError:
    print("File not found")
except fitz.EmptyFileError:
    print("File is empty")
except Exception as e:
    print(f"Error: {e}")
finally:
    if 'doc' in locals():
        doc.close()
```

## Performance Tips

1. **Use garbage collection** when saving large files:
   ```python
   doc.save("output.pdf", garbage=4, deflate=True)
   ```

2. **Process pages in batches** for large documents:
   ```python
   batch_size = 10
   for i in range(0, len(doc), batch_size):
       # Process batch
       pass
   ```

3. **Use context managers** when possible:
   ```python
   with fitz.open("document.pdf") as doc:
       # Process document
       pass
   ```

## Best Practices

1. **Always close documents** to free resources
2. **Check file existence** before opening
3. **Validate page indices** to avoid index errors
4. **Use RGB colors correctly** (0-1 range, not 0-255)
5. **Save with compression** (`deflate=True`) for smaller files
6. **Test on copies** before modifying important documents
7. **Use try-except** blocks for error handling
8. **Process large files in chunks** to avoid memory issues

## Comparison with Other Libraries

| Feature | PyMuPDF | pypdf | pdfplumber |
|---------|---------|-------|------------|
| Draw shapes | ✅ | ❌ | ❌ |
| Add annotations | ✅ | ✅ (limited) | ❌ |
| Extract text | ✅ | ✅ | ✅ (better) |
| Extract tables | ✅ | ❌ | ✅ (better) |
| Merge/split | ✅ | ✅ | ❌ |
| Performance | ⚡ Fast | 🐢 Slow | 🐢 Slow |
| Memory usage | ⚡ Low | 🐢 High | 🐢 High |

## Use Cases

- **Document Review**: Add highlights, comments, and annotations
- **Document Processing**: Add watermarks, stamps, and metadata
- **Content Extraction**: Extract text, images, and tables
- **Document Conversion**: Convert images to PDF, split/merge documents
- **Document Editing**: Draw shapes, add text, modify layout
- **Batch Processing**: Apply same changes to multiple documents

## License

PyMuPDF is available under AGPL-3.0 license (or commercial license).
