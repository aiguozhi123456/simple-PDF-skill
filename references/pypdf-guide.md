# pypdf Guide

## Overview

pypdf is a pure-Python library for reading, writing, and manipulating PDF documents. Ideal for merging, splitting, rotating, and encrypting PDFs.

```python
from pypdf import PdfReader, PdfWriter
```

## Installation

```bash
pip install pypdf
```

## Basic Operations

### Merge PDFs
```python
from pypdf import PdfWriter, PdfReader

writer = PdfWriter()
for pdf_file in ["doc1.pdf", "doc2.pdf", "doc3.pdf"]:
    reader = PdfReader(pdf_file)
    for page in reader.pages:
        writer.add_page(page)

with open("merged.pdf", "wb") as output:
    writer.write(output)
```

### Split PDF
```python
reader = PdfReader("input.pdf")
for i, page in enumerate(reader.pages):
    writer = PdfWriter()
    writer.add_page(page)
    with open(f"page_{i+1}.pdf", "wb") as output:
        writer.write(output)
```

### Rotate Pages
```python
reader = PdfReader("input.pdf")
writer = PdfWriter()

page = reader.pages[0]
page.rotate(90)  # Rotate 90 degrees clockwise
writer.add_page(page)

with open("rotated.pdf", "wb") as output:
    writer.write(output)
```

### Extract Metadata
```python
reader = PdfReader("document.pdf")
meta = reader.metadata
print(f"Title: {meta.title}")
print(f"Author: {meta.author}")
print(f"Subject: {meta.subject}")
print(f"Creator: {meta.creator}")
```

### Password Protection
```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

for page in reader.pages:
    writer.add_page(page)

# Add password
writer.encrypt("userpassword", "ownerpassword")

with open("encrypted.pdf", "wb") as output:
    writer.write(output)
```

### Decrypt PDF
```python
from pypdf import PdfReader

reader = PdfReader("encrypted.pdf")
if reader.is_encrypted:
    reader.decrypt("password")
    # Now you can access pages
    for page in reader.pages:
        print(page.extract_text())
```

## Advanced Operations

### Add Watermark
```python
from pypdf import PdfReader, PdfWriter

# Create watermark (or load existing)
watermark = PdfReader("watermark.pdf").pages[0]

# Apply to all pages
reader = PdfReader("document.pdf")
writer = PdfWriter()

for page in reader.pages:
    page.merge_page(watermark)
    writer.add_page(page)

with open("watermarked.pdf", "wb") as output:
    writer.write(output)
```

### Crop Pages
```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

# Crop page (left, bottom, right, top in points)
page = reader.pages[0]
page.mediabox.left = 50
page.mediabox.bottom = 50
page.mediabox.right = 550
page.mediabox.top = 750

writer.add_page(page)

with open("cropped.pdf", "wb") as output:
    writer.write(output)
```

### Extract Text
```python
from pypdf import PdfReader

reader = PdfReader("document.pdf")
text = ""

for page in reader.pages:
    text += page.extract_text()

print(text)
```

### Extract Images
```python
from pypdf import PdfReader

reader = PdfReader("document.pdf")
images = []

for page in reader.pages:
    if "/Images" in page["/Resources"]:
        for img in page["/Resources"]["/Images"].objects:
            images.append(img)

print(f"Found {len(images)} images")
```

## Page Manipulation

### Add Pages
```python
from pypdf import PdfWriter

writer = PdfWriter()
# Create blank page and add
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

new_page_canvas = canvas.Canvas("temp.pdf", pagesize=letter)
new_page_canvas.drawString(100, 500, "New Page")
new_page_canvas.save()

new_reader = PdfReader("temp.pdf")
writer.add_page(new_reader.pages[0])
```

### Delete Pages
```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

# Add all pages except index 5
for i, page in enumerate(reader.pages):
    if i != 5:
        writer.add_page(page)

with open("without_page_6.pdf", "wb") as output:
    writer.write(output)
```

### Reorder Pages
```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

# Reverse page order
for page in reversed(reader.pages):
    writer.add_page(page)

with open("reversed.pdf", "wb") as output:
    writer.write(output)
```

## Common Patterns

### Batch Merge Multiple Files
```python
import glob
from pypdf import PdfWriter, PdfReader

writer = PdfWriter()
for pdf_file in sorted(glob.glob("*.pdf")):
    reader = PdfReader(pdf_file)
    for page in reader.pages:
        writer.add_page(page)

with open("all_merged.pdf", "wb") as output:
    writer.write(output)
```

### Extract Specific Page Ranges
```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

# Extract pages 2, 4, 6, 8
for i in [1, 3, 5, 7]:
    if i < len(reader.pages):
        writer.add_page(reader.pages[i])

with open("selected_pages.pdf", "wb") as output:
    writer.write(output)
```

## Error Handling

```python
from pypdf import PdfReader, PdfWriter

try:
    reader = PdfReader("document.pdf")
    # Process PDF
    writer = PdfWriter()
    writer.add_page(reader.pages[0])
    
    with open("output.pdf", "wb") as output:
        writer.write(output)
        
except Exception as e:
    print(f"Error processing PDF: {e}")
```

### Add Bookmarks (Outlines)

```python
from pypdf import PdfReader, PdfWriter

def create_pdf_with_bookmarks(output_path):
    """Create PDF with hierarchical bookmarks"""
    reader = PdfReader("input.pdf")
    writer = PdfWriter()
    
    # Copy all pages
    for page in reader.pages:
        writer.add_page(page)
    
    # Add root-level bookmark
    writer.add_outline_item("Chapter 1: Introduction", 0, parent=None)
    
    # Add child bookmarks
    writer.add_outline_item("Section 1.1", 0, parent=0)
    writer.add_outline_item("Section 1.2", 1, parent=0)
    
    # Add another root bookmark
    writer.add_outline_item("Chapter 2: Methodology", 3, parent=None)
    
    # Add nested bookmarks
    parent_id = writer.add_outline_item("Chapter 3: Results", 5, parent=None)
    writer.add_outline_item("Section 3.1", 5, parent=parent_id)
    writer.add_outline_item("Section 3.2", 6, parent=parent_id)
    
    with open(output_path, "wb") as f:
        writer.write(f)

create_pdf_with_bookmarks("bookmarked.pdf")
```

### Manage Existing Bookmarks

```python
def extract_bookmarks(pdf_path):
    """Extract all bookmarks from PDF"""
    reader = PdfReader(pdf_path)
    
    def print_outline(outline, level=0):
        """Recursively print outline structure"""
        if isinstance(outline, list):
            for item in outline:
                print_outline(item, level)
        else:
            indent = "  " * level
            print(f"{indent}- {outline.title} (Page {outline.page + 1})")
            if outline.children:
                print_outline(outline.children, level + 1)
    
    print_outline(reader.outline)

def add_bookmark_to_existing(pdf_path, output_path, bookmark_title, page_num):
    """Add bookmark to existing PDF"""
    reader = PdfReader(pdf_path)
    writer = PdfWriter()
    
    # Clone all pages
    for page in reader.pages:
        writer.add_page(page)
    
    # Copy existing bookmarks
    if reader.outline:
        writer.clone_outline(reader)
    
    # Add new bookmark
    writer.add_outline_item(bookmark_title, page_num)
    
    with open(output_path, "wb") as f:
        writer.write(f)
```

### Add Links and Actions

```python
def add_internal_link(input_pdf, output_pdf):
    """Add internal links to jump between pages"""
    reader = PdfReader(input_pdf)
    writer = PdfWriter()
    
    # Clone pages
    for page in reader.pages:
        writer.add_page(page)
    
    # Add link from page 0 to page 5
    # Create link rectangle (left, bottom, right, top)
    link_rect = [100, 100, 300, 120]
    
    # Add link that jumps to page 5
    writer.add_link(
        page_number=0,
        pagenum=5,
        rect=link_rect,
        border=[0, 0, 1],  # Border: [h, v, width]
        fit="/FitV",  # Fit vertical
    )
    
    with open(output_pdf, "wb") as f:
        writer.write(f)

def add_external_link(input_pdf, output_pdf, url, rect_coords):
    """Add external link to URL"""
    reader = PdfReader(input_pdf)
    writer = PdfWriter()
    
    for page in reader.pages:
        writer.add_page(page)
    
    # Add link to URL on first page
    writer.add_uri(
        page_number=0,
        uri=url,
        rect=rect_coords,
        border=[0, 0, 1],
    )
    
    with open(output_pdf, "wb") as f:
        writer.write(f)
```

### Set Page Properties and Transitions

```python
def set_page_properties(input_pdf, output_pdf):
    """Set page rotation, labels, and transitions"""
    reader = PdfReader(input_pdf)
    writer = PdfWriter()
    
    for i, page in enumerate(reader.pages):
        writer.add_page(page)
        
        # Rotate specific page
        if i == 0:
            writer.pages[i].rotate(90)
        
        # Set page label
        if i == 1:
            writer.pages[i].mediabox.lower_left = (0, 0)
            writer.pages[i].mediabox.upper_right = (612, 792)
    
    with open(output_pdf, "wb") as f:
        writer.write(f)
```

### Modify Metadata

```python
def update_metadata(input_pdf, output_pdf, new_metadata):
    """Update PDF metadata"""
    reader = PdfReader(input_pdf)
    writer = PdfWriter()
    
    # Clone pages
    for page in reader.pages:
        writer.add_page(page)
    
    # Update metadata
    writer.add_metadata({
        "/Title": new_metadata.get("title", "Document Title"),
        "/Author": new_metadata.get("author", "Author Name"),
        "/Subject": new_metadata.get("subject", "Document Subject"),
        "/Creator": new_metadata.get("creator", "Application Name"),
        "/Keywords": new_metadata.get("keywords", "keyword1, keyword2"),
        "/CreationDate": new_metadata.get("date", "D:20241230120000Z"),
        "/ModDate": new_metadata.get("date", "D:20241230120000Z"),
    })
    
    with open(output_pdf, "wb") as f:
        writer.write(f)

# Usage
update_metadata("input.pdf", "updated.pdf", {
    "title": "Annual Report 2024",
    "author": "John Doe",
    "subject": "Financial Analysis",
    "creator": "PDF Generator",
    "keywords": "finance, report, 2024"
})
```

### Extract and Modify Form Fields

```python
def extract_form_fields(pdf_path):
    """Extract form field data from PDF"""
    reader = PdfReader(pdf_path)
    fields = reader.get_fields()
    
    if not fields:
        print("No form fields found")
        return
    
    for field_name, field in fields.items():
        print(f"Field: {field_name}")
        print(f"  Type: {field.get('/FT', 'Unknown')}")
        print(f"  Value: {field.get('/V', '')}")
        print(f"  Label: {field.get('/TU', '')}")
        print()

def fill_form_fields(input_pdf, output_pdf, field_data):
    """Fill form fields with data"""
    reader = PdfReader(input_pdf)
    writer = PdfWriter()
    
    # Clone pages
    for page in reader.pages:
        writer.add_page(page)
    
    # Get form fields
    fields = reader.get_fields()
    
    # Update field values
    if fields:
        for field_name, field_value in field_data.items():
            if field_name in fields:
                writer.update_page_form_field_values(
                    writer.pages[fields[field_name]['/P']], 
                    {field_name: field_value}
                )
                print(f"Filled field: {field_name} = {field_value}")
    
    with open(output_pdf, "wb") as f:
        writer.write(f)

# Usage
fill_form_fields("form.pdf", "filled.pdf", {
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "555-1234",
    "date": "2024-12-30"
})
```

### Split by Bookmarks

```python
def split_by_bookmarks(input_pdf, output_dir):
    """Split PDF into separate files based on bookmarks"""
    import os
    os.makedirs(output_dir, exist_ok=True)
    
    reader = PdfReader(input_pdf)
    
    if not reader.outline:
        print("No bookmarks found")
        return
    
    def extract_bookmark_ranges(outline):
        """Extract bookmark start and end page numbers"""
        ranges = []
        
        def process_item(item, parent_level=0):
            if isinstance(item, list):
                for sub_item in item:
                    process_item(sub_item, parent_level)
            else:
                start_page = item.page
                ranges.append((item.title, start_page))
        
        process_item(reader.outline)
        
        # Add end pages
        for i in range(len(ranges) - 1):
            ranges[i] = (ranges[i][0], ranges[i][1], ranges[i + 1][1])
        # Last item goes to end
        if ranges:
            ranges[-1] = (ranges[-1][0], ranges[-1][1], len(reader.pages))
        
        return ranges
    
    bookmark_ranges = extract_bookmark_ranges(reader.outline)
    
    # Create separate PDFs for each bookmark
    for title, start_page, end_page in bookmark_ranges:
        writer = PdfWriter()
        
        # Add pages in range
        for i in range(start_page, end_page):
            writer.add_page(reader.pages[i])
        
        # Sanitize filename
        safe_title = "".join(c if c.isalnum() or c in (' ', '-', '_') else '_' for c in title)
        output_path = os.path.join(output_dir, f"{safe_title}.pdf")
        
        with open(output_path, "wb") as f:
            writer.write(f)
        
        print(f"Created: {output_path} (pages {start_page + 1}-{end_page})")

# split_by_bookmarks("document.pdf", "split_output/")
```

### PDF/A Conversion

```python
def convert_to_pdfa(input_pdf, output_pdf):
    """Convert PDF to PDF/A-1b (ISO 19005-1:2005)"""
    reader = PdfReader(input_pdf)
    writer = PdfWriter()
    
    # Clone pages
    for page in reader.pages:
        writer.add_page(page)
    
    # Copy metadata and set PDF/A version
    if reader.metadata:
        writer.add_metadata(reader.metadata)
    
    # Set PDF/A conformance (simplified - actual PDF/A requires more steps)
    writer.add_metadata({
        "/GTS_PDFXVersion": "PDF/A-1b",
        "/GTS_PDFA1Version": "PDF/A-1b"
    })
    
    with open(output_pdf, "wb") as f:
        writer.write(f)
    print(f"Created PDF/A document: {output_pdf}")
```

### Add Attachments

```python
def add_attachment(input_pdf, output_pdf, attachment_path, attachment_name):
    """Add file attachment to PDF"""
    reader = PdfReader(input_pdf)
    writer = PdfWriter()
    
    # Clone pages
    for page in reader.pages:
        writer.add_page(page)
    
    # Read attachment file
    with open(attachment_path, "rb") as f:
        file_data = f.read()
    
    # Add attachment
    writer.add_attachment(
        filename=attachment_name,
        data=file_data
    )
    
    with open(output_pdf, "wb") as f:
        writer.write(f)

def extract_attachments(pdf_path, output_dir):
    """Extract all attachments from PDF"""
    import os
    os.makedirs(output_dir, exist_ok=True)
    
    reader = PdfReader(pdf_path)
    
    attachments = reader.attachments
    
    if not attachments:
        print("No attachments found")
        return
    
    for filename, file_data in attachments.items():
        output_path = os.path.join(output_dir, filename)
        with open(output_path, "wb") as f:
            f.write(file_data)
        print(f"Extracted: {output_path}")
```

## Best Practices

1. **Use context managers** - Always close PDF handles properly
2. **Handle encryption** - Check `is_encrypted` before accessing content
3. **Validate page indices** - Check bounds before accessing pages
4. **Process large files in chunks** - Avoid memory issues
5. **Use PdfWriter for output** - More flexible than direct file manipulation
6. **Back up before modifications** - PDF manipulation is destructive
7. **Test bookmark structure** - Complex outlines can have nested structures
8. **Validate form fields** - Field names can vary between PDF generators
9. **Preserve metadata** - Copy metadata when cloning documents
10. **Use appropriate PDF/A level** - Different requirements for different use cases

## License

pypdf is available under BSD 3-Clause license.