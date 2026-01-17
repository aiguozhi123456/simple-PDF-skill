# Reportlab API Guide

## Canvas Overview

Reportlab's `Canvas` is the primary object for creating PDFs. It provides methods for drawing text, shapes, and images.

```python
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter

c = canvas.Canvas("output.pdf", pagesize=letter)
# ... draw operations ...
c.save()
```

## Constructor Parameters

```python
canvas.Canvas(
    filename,              # Output PDF filename
    pagesize=letter,        # Page size (default: letter)
    bottomup=1,             # Coordinate system (0,0 at bottom-left)
    pageCompression=1,      # Enable compression
    verbosity=0,            # Debug verbosity
    encrypt=None            # Encryption settings
)
```

## Text Methods

### Basic Text Drawing

```python
# Draw text at position (x, y)
c.drawString(x, y, "text")

# Draw right-aligned text
c.drawRightString(x, y, "text")

# Draw centered text (use manually calculated offset)
text_width = c.stringWidth("text", font_name, font_size)
c.drawString(x - text_width/2, y, "text")

# Get text width before drawing
width = c.stringWidth("text", font_name, font_size)
```

### Text Formatting

```python
# Set font
c.setFont("Helvetica-Bold", 12)

# Set text color
c.setFillColor(HexColor("#FF0000"))

# Set text transparency (0=transparent, 1=opaque)
c.setFillAlpha(0.5)
```

### Advanced Text (PLATYPUS)

For complex text layout, use the higher-level PLATYPUS framework:

```python
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib.units import inch

doc = SimpleDocTemplate("document.pdf")
story = []
styles = getSampleStyleSheet()

# Add paragraph
p = Paragraph("This is a paragraph", styles["Normal"])
story.append(p)
story.append(Spacer(1, inch/4))

# Build document
doc.build(story)
```

## Drawing Methods

### Lines

```python
# Set stroke properties
c.setStrokeColor(HexColor("#000000"))
c.setLineWidth(2)
c.setLineCap(0)  # 0=butt, 1=round, 2=square
c.setLineJoin(0)  # 0=miter, 1=round, 2=bevel
c.setDash(array=[], phase=0)  # Solid line

# Draw line
c.line(x1, y1, x2, y2)

# Dashed line
c.setDash([3, 2], phase=0)  # 3pt on, 2pt off
```

### Rectangles

```python
# Draw rectangle
c.rect(x, y, width, height, stroke=1, fill=0)

# Filled rectangle
c.setFillColor(HexColor("#FF0000"))
c.rect(x, y, width, height, stroke=0, fill=1)

# Rounded rectangle (not directly available, use custom path)
```

### Circles and Ellipses

```python
# Draw ellipse/circle
c.ellipse(x, y, x+width, y+height, stroke=1, fill=0)

# Circle (equal width/height)
radius = 50
c.ellipse(x, y, x+radius*2, y+radius*2)
```

### Polygons and Paths

```python
from reportlab.lib.pathobject import PathObject

# Draw polygon
path = PathObject()
path.moveTo(x1, y1)
path.lineTo(x2, y2)
path.lineTo(x3, y3)
path.closePath()
c.drawPath(path, stroke=1, fill=0)
```

### Shapes

```python
from reportlab.lib.shapes import Drawing, Rect, Circle, String

# Create drawing
d = Drawing(width, height)

# Add shape
d.add(Rect(x, y, width, height, fillColor=HexColor("#FF0000")))
d.add(Circle(center_x, center_y, radius, fillColor=HexColor("#00FF00")))

# Render to canvas
d.drawOn(c, 0, 0)
```

## Images

```python
# Draw image
c.drawImage(
    image_path,           # File path
    x, y,                 # Position
    width=None,           # Optional width
    height=None,          # Optional height
    mask='auto',           # Mask for transparency
    preserveAspectRatio=True
)

# Get image dimensions
from reportlab.lib.utils import ImageReader
img = ImageReader(image_path)
img_width, img_height = img.getSize()
```

## Grid and Table

```python
from reportlab.platypus import Table, TableStyle

# Create table
data = [
    ["Header 1", "Header 2", "Header 3"],
    ["Row 1 Col 1", "Row 1 Col 2", "Row 1 Col 3"],
    ["Row 2 Col 1", "Row 2 Col 2", "Row 2 Col 3"]
]

table = Table(data, colWidths=[100, 100, 100], rowHeights=30)

# Add style
style = TableStyle([
    ('BACKGROUND', (0, 0), (-1, 0), HexColor("#4472C4")),
    ('TEXTCOLOR', (0, 0), (-1, 0), HexColor("#FFFFFF")),
    ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
    ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
    ('FONTSIZE', (0, 0), (-1, 0), 12),
    ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
    ('GRID', (0, 0), (-1, -1), 1, HexColor("#CCCCCC"))
])
table.setStyle(style)

# Draw on canvas
table.wrapOn(c, width, height)
table.drawOn(c, x, y)
```

## Page Management

```python
# Add new page
c.showPage()

# Get current page number
page_num = c.getPageNumber()

# Set page size dynamically
c.setPageSize((width, height))

# Rotate page
c.rotate(angle)  # in degrees
```

## Colors

```python
from reportlab.lib.colors import (
    HexColor,      # Hex strings
    CMYKColor,     # CMYK color
    PCMYKColor,    # Percentage CMYK
    black, white,  # Predefined colors
    red, green, blue,
    grey, lightgrey
)

# Hex color
color = HexColor("#1F4E79")

# RGB color
from reportlab.lib.colors import Color
color = Color(0.12, 0.31, 0.47, alpha=1.0)

# CMYK color
cmyk = CMYKColor(0.8, 0.6, 0, 0.1)
```
## Fonts

```python
# Standard fonts (PostScript)
c.setFont("Helvetica", 12)
c.setFont("Helvetica-Bold", 12)
c.setFont("Helvetica-Oblique", 12)
c.setFont("Times-Roman", 12)
c.setFont("Times-Bold", 12)
c.setFont("Courier", 12)

# Register TrueType font
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont

pdfmetrics.registerFont(TTFont('CustomFont', 'custom_font.ttf'))
c.setFont("CustomFont", 12)
### Chinese Font Support

**Problem**: When creating PDFs with Chinese characters, text appears as black boxes if Chinese fonts are not properly registered.

**Issue Details**:
1. Reportlab's standard fonts don't support Chinese characters
2. TTC (TrueType Collection) fonts need extraction
3. Large font files (100MB+) slow down PDF generation

**Solution**: Use Lightweight Chinese Fonts

#### Option A: WQY Microhei (Recommended - 4.4MB)

```python
import subprocess
import os
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
from fontTools.ttLib import TTFont as FontTool

def setup_chinese_font_lightweight():
    """
    Setup lightweight Chinese font (4.4MB).
    Fast and efficient for most use cases.
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

#### Option B: Noto Sans CJK (Complete Support - 100MB+)

```python
def setup_chinese_font_full():
    """
    Setup full Chinese font using Noto Sans CJK (100MB+).
    Best for professional documents with complete CJK character support.
    """
    try:
        # Install Noto Sans CJK
        subprocess.run(['apt-get', 'update', '-qq'], capture_output=True)
        subprocess.run(['apt-get', 'install', '-y', 'fonts-noto-cjk-extra'], 
                      capture_output=True)
        
        # Extract from TTC (subfontIndex 2 = Simplified Chinese)
        from fontTools.ttLib import TTFont
        
        ttc_path = '/usr/share/fonts/opentype/noto/NotoSansCJK-Regular.ttc'
        output_path = '/home/user/.fonts/NotoSansSC-Regular.ttf'
        font = TTFont(ttc_path, fontNumber=2)
        font.save(output_path)
        
        # Bold font
        bold_ttc = '/usr/share/fonts/opentype/noto/NotoSansCJK-Bold.ttc'
        bold_output = '/home/user/.fonts/NotoSansSC-Bold.ttf'
        font_bold = TTFont(bold_ttc, fontNumber=2)
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

**Font Comparison**:

| Font | Size | Speed | Character Support | Use Case |
|------|------|-------|------------------|-----------|
| **WQY Microhei** | 4.4MB | ⚡ Fast | Basic Chinese | Quick docs, testing, 90% of use cases |
| **Noto Sans CJK** | 100MB+ | 🐢 Slow | Complete CJK | Professional, mixed CJK text |
| **Noto Sans SC** | ~15MB | 🚀 Fast | Simplified CN | Chinese-only professional docs |

**Important Notes**:
- **TTC files**: Always use `fontNumber` parameter when loading TTC files with `fontTools`
- **Subfont indexes**: Common indexes for CJK fonts:
  - 0-1: Japanese
  - 2: Simplified Chinese
  - 3: Traditional Chinese
  - 4-5: Korean
- **Font compatibility**: WQY Microhei is most reliable and lightweight
- **Caching**: Always cache extracted fonts to avoid repeated extraction

**Best Practices**:
1. **Start with lightweight**: Use WQY Microhei for most cases (4.4MB vs 100MB+)
2. **Cache fonts**: Check cache before extracting to speed up subsequent runs
3. **Test first**: Always test with a simple example before full document
4. **Handle errors**: Use try-except and fall back to Helvetica
5. **Upgrade when needed**: Only use full Noto Sans CJK for:
   - Professional documents requiring complete character support
   - Mixed Chinese/Japanese/Korean text
   - Rare characters not in WQY Microhei

## Page Sizes

```python
from reportlab.lib.pagesizes import (
    letter, A4, legal, tabloid,
    landscape, portrait
)

# Common sizes
letter    # 612 x 792 (8.5 x 11 inches)
A4        # 595 x 842 (210 x 297 mm)
legal     # 612 x 1008
landscape(letter)  # Rotated version

# Custom size
from reportlab.lib.pagesizes import custom_size
custom_size(8*72, 6*72)  # 8 x 6 inches in points
```

## Units

```python
from reportlab.lib.units import inch, cm, mm, pt

# Convert
points = inch      # 72
points = cm        # 28.35
points = mm        # 2.83
points = pt(1)     # 1

# Usage
c.drawString(1*inch, 1*cm, "text")
```

## State Management

```python
# Save current state
c.saveState()

# ... transformations ...

# Restore state
c.restoreState()

# Translate coordinate system
c.translate(dx, dy)

# Scale
c.scale(sx, sy)

# Rotate
c.rotate(angle)  # degrees
```

## Metadata

```python
# Set document metadata
c.setTitle("Document Title")
c.setAuthor("Author Name")
c.setSubject("Document Subject")
c.setCreator("Application Name")
c.setProducer("Producer Name")
```

## Encryption

```python
from reportlab.lib.pdfencrypt import StandardEncryption

# Create encryption
enc = StandardEncryption("password", 
                        canPrint=1, 
                        canCopy=1, 
                        canAnnotate=0)

c = canvas.Canvas("secure.pdf", encrypt=enc)
```

## Page Labels

```python
# Set page labels (requires advanced usage)
from reportlab.pdfgen import canvas

c = canvas.Canvas("labeled.pdf")
# Page 1
c.showPage()

# Page 2 (with label)
c.setPageLabel(format="%d", start=1)
c.showPage()
```

## Bookmarks (Outlines)

```python
# Add bookmark (requires PLATYPUS)
from reportlab.platypus import SimpleDocTemplate, Paragraph

doc = SimpleDocTemplate("bookmarked.pdf")
key = "section1"
doc.addKeyEntry(key, title="Section 1", level=0)
```

## Links and Actions

```python
# Add internal link
c.bookmarkPage(key="section1", fit="Fit")

# Add external link
c.linkURL(url="https://example.com", rect=(x1, y1, x2, y2))
```

## Barcodes

```python
from reportlab.graphics.barcode import code39, code128
from reportlab.lib.units import mm

# Create barcode
barcode = code39.Standard39("12345", barWidth=0.5*mm)

# Draw on canvas
barcode.drawOn(c, x, y)
```

## Charts

```python
from reportlab.graphics.charts.barcharts import VerticalBarChart
from reportlab.graphics.shapes import Drawing

# Create chart
chart = VerticalBarChart()
chart.x = 50
chart.y = 50
chart.width = 300
chart.height = 200
chart.data = [[10, 20, 30, 40, 50]]

# Add to drawing
d = Drawing(400, 300)
d.add(chart)

# Draw on canvas
d.drawOn(c, 0, 0)
```
---

## Advanced Multi-Page Document Generation

### Creating Documents with Headers and Footers

```python
from reportlab.lib.pagesizes import letter
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.platypus import (
    SimpleDocTemplate, PageTemplate, BaseDocTemplate,
    Paragraph, Frame, PageBreak
)
from reportlab.lib.units import inch

class HeaderFooterTemplate(BaseDocTemplate):
    """Document template with headers and footers on every page"""
    
    def __init__(self, filename, **kwargs):
        self.header_text = kwargs.pop('header_text', '')
        self.footer_text = kwargs.pop('footer_text', '')
        BaseDocTemplate.__init__(self, filename, **kwargs)
    
    def onPage(self, canvas, doc):
        """Draw header and footer on each page"""
        # Header
        canvas.saveState()
        canvas.setFont('Helvetica-Bold', 10)
        canvas.setFillColorRGB(0.2, 0.2, 0.2)
        canvas.drawString(0.5*inch, letter[1] - 0.5*inch, self.header_text)
        canvas.line(0.5*inch, letter[1] - 0.6*inch, letter[0] - 0.5*inch, letter[1] - 0.6*inch)
        canvas.restoreState()
        
        # Footer
        canvas.saveState()
        canvas.setFont('Helvetica', 9)
        canvas.setFillColorRGB(0.5, 0.5, 0.5)
        canvas.drawString(0.5*inch, 0.5*inch, self.footer_text)
        canvas.drawCentredString(letter[0]/2, 0.5*inch, f"Page {doc.page}")
        canvas.drawRightString(letter[0] - 0.5*inch, 0.5*inch, "Confidential")
        canvas.line(0.5*inch, 0.6*inch, letter[0] - 0.5*inch, 0.6*inch)
        canvas.restoreState()

def create_document_with_headers(filename):
    """Create document with headers and footers"""
    doc = HeaderFooterTemplate(
        filename,
        pagesize=letter,
        header_text="Annual Report 2024",
        footer_text="ABC Corporation - Internal Use Only"
    )
    
    story = []
    styles = getSampleStyleSheet()
    
    # Add content pages
    for i in range(3):
        story.append(Paragraph(f"Section {i+1}", styles["Heading1"]))
        story.append(Paragraph("Content goes here..." * 10, styles["Normal"]))
        story.append(PageBreak())
    
    doc.build(story)
```

### Table of Contents Generation

```python
from reportlab.platypus import TableOfContents
from reportlab.lib.styles import ParagraphStyle
from reportlab.lib.enums import TA_CENTER

def create_document_with_toc(filename):
    """Create document with table of contents"""
    doc = SimpleDocTemplate(filename, pagesize=letter)
    story = []
    styles = getSampleStyleSheet()
    
    # Define heading styles with TOC links
    h1 = ParagraphStyle(
        'Heading1',
        parent=styles['Heading1'],
        fontSize=18,
        leading=22,
        spaceAfter=12,
        textColor='navy'
    )
    
    h2 = ParagraphStyle(
        'Heading2',
        parent=styles['Heading2'],
        fontSize=14,
        leading=18,
        spaceAfter=10,
        textColor='darkblue'
    )
    
    # Create TOC
    toc = TableOfContents()
    toc.levelStyles = [
        ParagraphStyle(
            'TOCHeading1',
            parent=styles['Heading2'],
            fontSize=14,
            leading=16
        ),
        ParagraphStyle(
            'TOCHeading2',
            parent=styles['Normal'],
            fontSize=12,
            leading=14,
            leftIndent=20
        )
    ]
    
    story.append(Paragraph("Table of Contents", styles["Title"]))
    story.append(toc)
    story.append(PageBreak())
    
    # Add content with TOC markers
    story.append(Paragraph("Introduction", h1))
    story.append(Paragraph("This is introduction..." * 5, styles["Normal"]))
    story.append(Paragraph("Background", h2))
    story.append(Paragraph("Background information..." * 3, styles["Normal"]))
    
    story.append(Paragraph("Methodology", h1))
    story.append(Paragraph("Methodology description..." * 5, styles["Normal"]))
    
    story.append(Paragraph("Results", h1))
    story.append(Paragraph("Results analysis..." * 5, styles["Normal"]))
    story.append(Paragraph("Data Analysis", h2))
    story.append(Paragraph("Data analysis details..." * 3, styles["Normal"]))
    
    doc.build(story)
### Complex Tables with Merged Cells

```python
def create_complex_table():
    """Create table with merged cells"""
    data = [
        ["Q1 2024", "Revenue", "Expenses", "Profit"],
        ["January", "$50,000", "$30,000", "$20,000"],
        ["February", "$55,000", "$32,000", "$23,000"],
        ["March", "$60,000", "$35,000", "$25,000"],
        ["Total", "$165,000", "$97,000", "$68,000"]
    ]
    
    table = Table(data, colWidths=[1.5*inch, 2*inch, 2*inch, 2*inch])
    
    style = TableStyle([
        ('BACKGROUND', (0, 0), (-1, 0), colors.HexColor("#4472C4")),
        ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
        ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
        ('BACKGROUND', (0, -1), (-1, -1), colors.HexColor("#E7E6E6")),
        ('GRID', (0, 0), (-1, -1), 1, colors.grey),
    ])
    
    table.setStyle(style)
    return table
```
## Best Practices

1. **Always call `save()`** at the end to finalize the PDF
2. **Use units** (`inch`, `cm`) instead of raw numbers for positioning
3. **Check text width** before drawing to avoid overflow
4. **Group operations** that require same font/style together
5. **Use `showPage()`** to create new pages before drawing new content
6. **Remember Y starts from bottom-left** (0,0), not top-left
7. **Test on different page sizes** to ensure compatibility
8. **Use PLATYPUS** for complex text layout instead of Canvas
9. **Handle errors** gracefully when loading external resources (images, fonts)
10. **Compress images** before adding to reduce file size
11. **For Chinese documents**: Always install and register Chinese fonts before use
12. **Extract TTC fonts** to TTF format to avoid compatibility issues
13. **Test Chinese font rendering** with a small sample before creating full document
14. **Use PageBreak** for clean page transitions in multi-page documents
15. **Create custom templates** for consistent headers/footers across pages