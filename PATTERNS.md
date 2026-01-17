# Core Patterns - Reportlab & Pdfplumber
## 1. Basic PDF Structure

```python
from reportlab.lib.pagesizes import letter, A4
from reportlab.pdfgen import canvas
from reportlab.lib.units import inch, cm, pt
from reportlab.lib.colors import HexColor

# Initialize canvas (US Letter)
c = canvas.Canvas("document.pdf", pagesize=letter)

# Or A4 (common internationally)
c = canvas.Canvas("document.pdf", pagesize=A4)

# Get page dimensions
width, height = letter

# Always save at the end
c.save()
```
### 1.1 Chinese Font Support Pattern

### Option A: Lightweight Chinese Font (Recommended - 4.4MB)

```python
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
from fontTools.ttLib import TTFont as FontTool
import subprocess
import os

def setup_chinese_font_lightweight():
    """
    Setup lightweight Chinese font using WQY Microhei (4.4MB).
    Returns: (regular_font_name, bold_font_name)
    """
    font_cache = '/home/user/.fonts/wqy-microhei.ttf'
    
    # Check cache first
    if os.path.exists(font_cache):
        pdfmetrics.registerFont(TTFont('CJK-Light', font_cache))
        pdfmetrics.registerFont(TTFont('CJK-Bold-Light', font_cache))
        return 'CJK-Light', 'CJK-Bold-Light'
    
    try:
        # Install WQY Microhei (lightweight, only 4.4MB)
        subprocess.run(['apt-get', 'update', '-qq'], capture_output=True)
        subprocess.run(['apt-get', 'install', '-y', 'fonts-wqy-microhei'], 
                      capture_output=True)
        
        # Extract from TTC
        ttc_path = '/usr/share/fonts/truetype/wqy/wqy-microhei.ttc'
        font = FontTool(ttc_path, fontNumber=0)
        font.save(font_cache)
        
        print(f"✓ Lightweight Chinese font ready (4.4MB)")
        
        # Register fonts
        pdfmetrics.registerFont(TTFont('CJK-Light', font_cache))
        pdfmetrics.registerFont(TTFont('CJK-Bold-Light', font_cache))
        
        return 'CJK-Light', 'CJK-Bold-Light'
    except Exception as e:
        print(f"Warning: Font setup failed: {e}")
        return 'Helvetica', 'Helvetica-Bold'

# Usage - Simple and fast!
cjk_font, cjk_bold = setup_chinese_font_lightweight()
c.setFont(cjk_font, 12)
c.drawString(x, y, "中文内容")
```

### Option B: Full Chinese Font (Complete support - 100MB+)

```python
def setup_chinese_font_full():
    """
    Setup full Chinese font using Noto Sans CJK (100MB+).
    Best for professional documents with complete character support.
    Returns: (regular_font_name, bold_font_name)
    """
    try:
        # Install Noto Sans CJK (full CJK support, 100MB+)
        subprocess.run(['apt-get', 'update', '-qq'], capture_output=True)
        subprocess.run(['apt-get', 'install', '-y', 'fonts-noto-cjk-extra'], 
                      capture_output=True)
        
        # Extract from TTC (subfontIndex 2 = Simplified Chinese)
        ttc_path = '/usr/share/fonts/opentype/noto/NotoSansCJK-Regular.ttc'
        output_path = '/home/user/.fonts/NotoSansSC-Regular.ttf'
        font = FontTool(ttc_path, fontNumber=2)
        font.save(output_path)
        
        # Bold font
        bold_ttc = '/usr/share/fonts/opentype/noto/NotoSansCJK-Bold.ttc'
        bold_output = '/home/user/.fonts/NotoSansSC-Bold.ttf'
        font_bold = FontTool(bold_ttc, fontNumber=2)
        font_bold.save(bold_output)
        
        print(f"✓ Full Chinese font ready (100MB+)")
        
        # Register fonts
        pdfmetrics.registerFont(TTFont('CJK-Full', output_path))
        pdfmetrics.registerFont(TTFont('CJK-Bold-Full', bold_output))
        
        return 'CJK-Full', 'CJK-Bold-Full'
    except Exception as e:
        print(f"Warning: Font setup failed: {e}")
        return 'Helvetica', 'Helvetica-Bold'

# Usage - For professional documents
cjk_font, cjk_bold = setup_chinese_font_full()
c.setFont(cjk_font, 12)
c.drawString(x, y, "中文内容")
```

### Font Comparison

| Font | Size | Speed | Character Support | Use Case |
|------|------|-------|------------------|-----------|
| **WQY Microhei** | 4.4MB | ⚡ Fast | Basic Chinese | Quick docs, testing |
| **Noto Sans CJK** | 100MB+ | 🐢 Slow | Complete CJK | Professional, full support |
| **Noto Sans SC** | ~15MB | 🚀 Fast | Simplified CN | Chinese-only docs |

### Quick Recommendation

```python
# For most use cases, use lightweight
cjk, cjk_bold = setup_chinese_font_lightweight()

# Only use full font when needed:
# - Professional documents
# - Need complete CJK character support
# - Mixed Chinese/Japanese/Korean text
```



## 2. Adding Text

```python
def draw_text(c, text, x, y, font_name="Helvetica", font_size=12, 
              color=HexColor("#000000"), alignment="left"):
    """Draw text with styling"""
    c.setFont(font_name, font_size)
    c.setFillColor(color)
    
    if alignment == "center":
        text_width = c.stringWidth(text, font_name, font_size)
        c.drawString(x - text_width/2, y, text)
    elif alignment == "right":
        text_width = c.stringWidth(text, font_name, font_size)
        c.drawString(x - text_width, y, text)
    else:
        c.drawString(x, y, text)
```

## 3. Title Page Pattern

```python
def create_title_page(c, title, subtitle=None):
    """Create title page centered"""
    width, height = letter
    
    # Main title
    c.setFont("Helvetica-Bold", 36)
    c.setFillColor(HexColor("#1F4E79"))
    title_width = c.stringWidth(title, "Helvetica-Bold", 36)
    c.drawString((width - title_width) / 2, height / 2, title)
    
    # Subtitle (optional)
    if subtitle:
        c.setFont("Helvetica", 18)
        c.setFillColor(HexColor("#595959"))
        sub_width = c.stringWidth(subtitle, "Helvetica", 18)
        c.drawString((width - sub_width) / 2, height / 2 - 50, subtitle)
    
    c.showPage()
```

## 4. Header and Footer Pattern

```python
def add_header(c, text, page_num, total_pages):
    """Add header with page numbers"""
    width, height = letter
    
    # Line
    c.setStrokeColor(HexColor("#CCCCCC"))
    c.setLineWidth(1)
    c.line(50, height - 50, width - 50, height - 50)
    
    # Text
    c.setFont("Helvetica", 10)
    c.setFillColor(HexColor("#666666"))
    c.drawString(50, height - 65, text)
    
    # Page number
    c.drawRightString(width - 50, height - 65, f"Page {page_num} of {total_pages}")

def add_footer(c, text):
    """Add footer text"""
    width = letter[0]
    
    c.setStrokeColor(HexColor("#CCCCCC"))
    c.setLineWidth(1)
    c.line(50, 50, width - 50, 50)
    
    c.setFont("Helvetica", 9)
    c.setFillColor(HexColor("#999999"))
    c.drawString(50, 35, text)
```

## 5. Data-Driven Content Pattern

```python
def create_bullet_list(c, items, x, y, font_size=12, spacing=15, 
                       bullet="•", color=HexColor("#595959")):
    """Create bullet list from data"""
    c.setFont("Helvetica", font_size)
    c.setFillColor(color)
    
    for item in items:
        c.drawString(x, y, f"{bullet} {item}")
        y -= spacing
    
    return y  # Return new Y position
```

## 6. Multi-Column Layout

```python
def create_two_column(c, left_items, right_items, y_start, col_width=250, spacing=20):
    """Create two-column layout"""
    # Left column
    y_left = y_start
    c.setFont("Helvetica", 10)
    for item in left_items:
        c.drawString(50, y_left, item)
        y_left -= 15
    
    # Right column
    y_right = y_start
    for item in right_items:
        c.drawString(50 + col_width + spacing, y_right, item)
        y_right -= 15
    
    return min(y_left, y_right)
```

## 7. Table Pattern (Simple)

```python
def draw_simple_table(c, data, x, y, col_widths, row_height=20, 
                      header_bg=HexColor("#4472C4"), header_text=HexColor("#FFFFFF")):
    """Draw simple table with header"""
    # Header row
    c.setFillColor(header_bg)
    c.rect(x, y, sum(col_widths), row_height, fill=1, stroke=0)
    
    c.setFont("Helvetica-Bold", 10)
    c.setFillColor(header_text)
    
    col_x = x
    for header, width in zip(data[0], col_widths):
        c.drawString(col_x + 5, y + 5, header)
        col_x += width
    
    # Data rows
    c.setFont("Helvetica", 9)
    c.setFillColor(HexColor("#000000"))
    
    y -= row_height
    for row in data[1:]:
        col_x = x
        # Alternating row background
        if data.index(row) % 2 == 0:
            c.setFillColor(HexColor("#F2F2F2"))
            c.rect(x, y, sum(col_widths), row_height, fill=1, stroke=0)
        
        for cell, width in zip(row, col_widths):
            c.setFillColor(HexColor("#000000"))
            c.drawString(col_x + 5, y + 5, str(cell))
            col_x += width
        
        y -= row_height
    
    # Border
    c.setStrokeColor(HexColor("#CCCCCC"))
    c.rect(x, y + (len(data)-1) * row_height, sum(col_widths), len(data) * row_height)
```

## 8. Image Pattern

```python
def add_image(c, image_path, x, y, width=None, height=None, caption=None):
    """Add image with optional caption"""
    from reportlab.lib.utils import ImageReader
    img = ImageReader(image_path)
    img_width, img_height = img.getSize()
    
    # Calculate aspect ratio if only one dimension specified
    if width and not height:
        ratio = img_height / img_width
        height = width * ratio
    elif height and not width:
        ratio = img_width / img_height
        width = height * ratio
    elif not width and not height:
        width = 2 * inch  # Default
        height = width * (img_height / img_width)
    
    c.drawImage(image_path, x, y, width=width, height=height)
    
    # Caption
    if caption:
        c.setFont("Helvetica-Italic", 9)
        c.setFillColor(HexColor("#666666"))
        caption_width = c.stringWidth(caption, "Helvetica-Italic", 9)
        c.drawString(x + (width - caption_width) / 2, y - 15, caption)
```

## 9. Section Divider Pattern

```python
def create_section_divider(c, section_title, bg_color=HexColor("#4472C4")):
    """Create colored section divider page"""
    width, height = letter
    
    # Background
    c.setFillColor(bg_color)
    c.rect(0, 0, width, height, fill=1, stroke=0)
    
    # Title
    c.setFont("Helvetica-Bold", 48)
    c.setFillColor(HexColor("#FFFFFF"))
    title_width = c.stringWidth(section_title, "Helvetica-Bold", 48)
    c.drawString((width - title_width) / 2, height / 2 - 24, section_title)
    
    c.showPage()
```

## 10. PDF Reading Pattern (pdfplumber)

```python
import pdfplumber

def extract_text_from_pdf(pdf_path):
    """Extract all text from PDF"""
    text = ""
    with pdfplumber.open(pdf_path) as pdf:
        for page in pdf.pages:
            text += page.extract_text() + "\n"
    return text

def extract_tables_from_pdf(pdf_path):
    """Extract all tables from PDF"""
    tables = []
    with pdfplumber.open(pdf_path) as pdf:
        for page in pdf.pages:
            tables.append(page.extract_table())
    return tables

def extract_text_by_page(pdf_path, page_num):
    """Extract text from specific page (1-indexed)"""
    with pdfplumber.open(pdf_path) as pdf:
        if page_num <= len(pdf.pages):
            return pdf.pages[page_num - 1].extract_text()
    return None
```

## 11. Multi-Page Document Pattern

```python
def create_multi_page_report(data):
    """Create multi-page report from data"""
    c = canvas.Canvas("report.pdf", pagesize=letter)
    width, height = letter
    
    # Title page
    create_title_page(c, data["title"], data.get("subtitle"))
    
    # Content pages
    page_num = 1
    total_pages = len(data["sections"]) + 1
    
    for section in data["sections"]:
        # Header
        add_header(c, data["title"], page_num, total_pages)
        
        # Section title
        c.setFont("Helvetica-Bold", 24)
        c.setFillColor(HexColor("#1F4E79"))
        c.drawString(50, height - 120, section["title"])
        
        # Content
        y = height - 160
        y = create_bullet_list(c, section["content"], 50, y)
        
        # Footer
        add_footer(c, "Generated by Reportlab")
        
        c.showPage()
        page_num += 1
    
    c.save()
```