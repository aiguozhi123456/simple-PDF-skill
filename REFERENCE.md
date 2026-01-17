# Quick Reference - Reportlab & Pdfplumber

## Page Sizes

```python
from reportlab.lib.pagesizes import letter, A4, legal, tabloid

# US sizes
LETTER = letter          # 612 x 792 points (8.5 x 11 inches)
LEGAL = legal            # 612 x 1008 points
TABLOID = tabloid        # 792 x 1224 points

# International
A4 = A4                  # 595 x 842 points (210 x 297 mm)

# Landscape
from reportlab.lib.pagesizes import landscape
LANDSCAPE_LETTER = landscape(letter)
LANDSCAPE_A4 = landscape(A4)
```

## Color Palette

```python
from reportlab.lib.colors import HexColor, black, white, red, green, blue

# Using Hex strings
BLUE = HexColor("#1F4E79")
LIGHT_BLUE = HexColor("#4472C4")
GREEN = HexColor("#4F6228")
RED = HexColor("#C00000")
ORANGE = HexColor("#FF9933")
GRAY = HexColor("#595959")
LIGHT_GRAY = HexColor("#F2F2F2")
GOLD = HexColor("#D4AF37")
```

## Font Sizes (Points)

```python
from reportlab.lib.units import pt

TITLE = 36
SUBTITLE = 24
HEADING = 18
BODY = 12
SMALL = 10
CAPTION = 9
```

## Standard Fonts

```python
# Sans-serif
HELVETICA = "Helvetica"
HELVETICA_BOLD = "Helvetica-Bold"
HELVETICA_ITALIC = "Helvetica-Oblique"

# Serif
TIMES = "Times-Roman"
TIMES_BOLD = "Times-Bold"
TIMES_ITALIC = "Times-Italic"

# Monospace
COURIER = "Courier"
COURIER_BOLD = "Courier-Bold"
```

## Units of Measurement

```python
from reportlab.lib.units import inch, cm, mm, pt

# 1 inch = 2.54 cm = 72 points
# 1 cm = 28.35 points
# 1 mm = 2.83 points

# Common conversions
ONE_INCH = inch
HALF_INCH = 0.5 * inch
ONE_CM = cm
ONE_MM = mm

# Font sizes
PT_10 = pt(10)
PT_12 = pt(12)
PT_24 = pt(24)
```

## Page Margins (Letter)

```python
from reportlab.lib.pagesizes import letter

width, height = letter

# Standard margins
MARGIN_LEFT = 50
MARGIN_RIGHT = width - 50
MARGIN_TOP = height - 50
MARGIN_BOTTOM = 50

# Content area
CONTENT_WIDTH = width - 100
CONTENT_HEIGHT = height - 100

# Common positions
TITLE_Y = height - 100
HEADER_Y = height - 60
FOOTER_Y = 50
CENTER_X = width / 2
CENTER_Y = height / 2
```

## Text Alignment Methods

```python
# Left-aligned
c.drawString(x, y, "Text")

# Right-aligned
c.drawRightString(x, y, "Text")

# Center-aligned (calculate manually)
text_width = c.stringWidth("Text", font_name, font_size)
c.drawString(x - text_width/2, y, "Text")
```

## Canvas Coordinates

```python
# PDF coordinate system: (0,0) is bottom-left
# X increases to the right
# Y increases upward

width, height = letter  # Get page dimensions

# Example positions:
# Top-left:     (50, height - 50)
# Top-right:    (width - 50, height - 50)
# Bottom-left:  (50, 50)
# Bottom-right: (width - 50, 50)
# Center:       (width/2, height/2)
```

## Drawing Methods

```python
# Lines
c.setStrokeColor(HexColor("#000000"))
c.setLineWidth(1)
c.line(x1, y1, x2, y2)

# Rectangles
c.rect(x, y, width, height, stroke=1, fill=0)
c.rect(x, y, width, height, stroke=0, fill=1)  # Filled only

# Circles/Ovals
c.ellipse(x, y, x + width, y + height, stroke=1, fill=0)

# Images
c.drawImage(image_path, x, y, width=None, height=None)
```

## Text Formatting

```python
# Set font
c.setFont("Helvetica-Bold", 12)

# Set color (both fill and stroke)
c.setFillColor(HexColor("#000000"))
c.setStrokeColor(HexColor("#000000"))

# Get text width
text_width = c.stringWidth("Text", "Helvetica", 12)
```

## Page Management

```python
# New page
c.showPage()

# Save document (required)
c.save()

# Get page count (after creation)
num_pages = c.getPageNumber()
```

## Common Patterns

### Title Box
```python
c.setFont("Helvetica-Bold", 36)
c.setFillColor(HexColor("#1F4E79"))
title_width = c.stringWidth("Title", "Helvetica-Bold", 36)
c.drawString((width - title_width) / 2, height / 2, "Title")
```

### Header Line
```python
c.setStrokeColor(HexColor("#CCCCCC"))
c.setLineWidth(1)
c.line(50, height - 50, width - 50, height - 50)
```

### Bullet Point
```python
c.setFont("Helvetica", 12)
c.drawString(70, y, "• Bullet text")
```

### Two-Column Layout
```python
# Left column: x=50 to x=275
# Right column: x=295 to x=520
# Spacing: 20 points
```

## Pdfplumber Quick Reference

```python
import pdfplumber

# Open PDF
with pdfplumber.open("file.pdf") as pdf:
    # Get info
    num_pages = len(pdf.pages)
    metadata = pdf.metadata
    
    # Iterate pages
    for page in pdf.pages:
        # Extract text
        text = page.extract_text()
        
        # Extract table
        table = page.extract_table()
        
        # Extract tables
        tables = page.extract_tables()
        
        # Get page dimensions
        width, height = page.width, page.height
        
        # Get bounding box
        bbox = page.bbox  # (x0, top, x1, bottom)
```

## Conversion Reference

```
1 inch = 2.54 cm = 25.4 mm = 72 points
1 cm = 0.394 inches = 28.35 points
1 mm = 0.0394 inches = 2.83 points
1 point = 1/72 inch = 0.0139 inches

Font sizes in points:
9 pt  = captions/small text
10 pt = footers/notes
12 pt = body text (standard)
14 pt = larger body text
18 pt = headings
24 pt = subtitles
36 pt = titles
48 pt = large titles
```

## Common Pitfalls

✅ **DO** - Use `HexColor("#RRGGBB")` for colors
❌ **DON'T** - Use tuples or color names only

✅ **DO** - Use units (inch, cm, mm) for positioning
❌ **DON'T** - Use raw numbers without context

✅ **DO** - Remember Y starts from bottom (0,0)
❌ **DON'T** - Assume Y starts from top

✅ **DO** - Call `save()` at the end
❌ **DON'T** - Forget to save the document

✅ **DO** - Use `showPage()` to create new pages
❌ **DON'T** - Try to draw beyond page boundaries

✅ **DO** - Check text width before positioning
❌ **DON'T** - Assume text fits in given space

✅ **DO** - Use `pdfplumber` for reading PDFs
❌ **DON'T** - Use `reportlab` to read existing PDFs (it's write-only)
