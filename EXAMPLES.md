# Complete Examples - Reportlab & Pdfplumber

## Example 1: Basic PDF Document

```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
from reportlab.lib.colors import HexColor

c = canvas.Canvas("basic.pdf", pagesize=letter)
width, height = letter

# Title
c.setFont("Helvetica-Bold", 24)
c.setFillColor(HexColor("#1F4E79"))
c.drawString(100, height - 100, "My First PDF")

# Content
c.setFont("Helvetica", 12)
c.setFillColor(HexColor("#000000"))
c.drawString(100, height - 150, "This is a simple PDF document.")
c.drawString(100, height - 170, "Created with Reportlab.")

c.save()
```

## Example 2: Invoice Generation

```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
from reportlab.lib.colors import HexColor
from reportlab.lib.units import inch

def create_invoice(invoice_data):
    c = canvas.Canvas(f"invoice_{invoice_data['invoice_num']}.pdf", pagesize=letter)
    width, height = letter
    
    # Company header
    c.setFont("Helvetica-Bold", 16)
    c.setFillColor(HexColor("#1F4E79"))
    c.drawString(50, height - 50, "ACME Corporation")
    
    c.setFont("Helvetica", 10)
    c.drawString(50, height - 70, "123 Business Ave, Suite 100")
    c.drawString(50, height - 85, "San Francisco, CA 94102")
    
    # Invoice info (right-aligned)
    c.setFont("Helvetica-Bold", 14)
    c.setFillColor(HexColor("#000000"))
    invoice_title = f"INVOICE #{invoice_data['invoice_num']}"
    c.drawRightString(width - 50, height - 50, invoice_title)
    
    c.setFont("Helvetica", 10)
    c.drawRightString(width - 50, height - 70, f"Date: {invoice_data['date']}")
    c.drawRightString(width - 50, height - 85, f"Due: {invoice_data['due_date']}")
    
    # Client info
    c.setFont("Helvetica-Bold", 12)
    c.drawString(50, height - 130, "Bill To:")
    c.setFont("Helvetica", 10)
    c.drawString(50, height - 145, invoice_data['client']['name'])
    c.drawString(50, height - 160, invoice_data['client']['address'])
    
    # Line
    c.setStrokeColor(HexColor("#CCCCCC"))
    c.setLineWidth(1)
    c.line(50, height - 190, width - 50, height - 190)
    
    # Table header
    c.setFillColor(HexColor("#4472C4"))
    c.rect(50, height - 220, width - 100, 25, fill=1, stroke=0)
    
    c.setFont("Helvetica-Bold", 10)
    c.setFillColor(HexColor("#FFFFFF"))
    c.drawString(55, height - 207, "Description")
    c.drawString(300, height - 207, "Qty")
    c.drawString(380, height - 207, "Price")
    c.drawRightString(width - 55, height - 207, "Total")
    
    # Line items
    c.setFont("Helvetica", 10)
    c.setFillColor(HexColor("#000000"))
    y = height - 245
    total = 0
    
    for item in invoice_data['items']:
        c.drawString(55, y, item['description'])
        c.drawString(300, y, str(item['qty']))
        c.drawString(380, y, f"${item['price']:.2f}")
        line_total = item['qty'] * item['price']
        c.drawRightString(width - 55, y, f"${line_total:.2f}")
        total += line_total
        y -= 25
    
    # Total
    c.setFont("Helvetica-Bold", 12)
    c.drawRightString(width - 55, y - 20, f"Total: ${total:.2f}")
    
    c.save()

# Usage
invoice_data = {
    "invoice_num": "INV-2024-001",
    "date": "2024-01-15",
    "due_date": "2024-02-15",
    "client": {
        "name": "John Smith",
        "address": "456 Client St, New York, NY 10001"
    },
    "items": [
        {"description": "Web Development Service", "qty": 1, "price": 2500.00},
        {"description": "Hosting (12 months)", "qty": 1, "price": 120.00},
        {"description": "Domain Registration", "qty": 2, "price": 15.00}
    ]
}

create_invoice(invoice_data)
```

## Example 3: Quarterly Report

```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
from reportlab.lib.colors import HexColor
from reportlab.lib.units import inch

def create_quarterly_report(report_data):
    c = canvas.Canvas("quarterly_report.pdf", pagesize=letter)
    width, height = letter
    
    # Page 1: Title
    c.setFont("Helvetica-Bold", 36)
    c.setFillColor(HexColor("#1F4E79"))
    title_width = c.stringWidth(report_data['title'], "Helvetica-Bold", 36)
    c.drawString((width - title_width) / 2, height / 2, report_data['title'])
    
    c.setFont("Helvetica", 18)
    c.setFillColor(HexColor("#595959"))
    sub_width = c.stringWidth(report_data['period'], "Helvetica", 18)
    c.drawString((width - sub_width) / 2, height / 2 - 50, report_data['period'])
    
    c.showPage()
    
    # Page 2: Executive Summary
    c.setFont("Helvetica-Bold", 24)
    c.setFillColor(HexColor("#1F4E79"))
    c.drawString(50, height - 100, "Executive Summary")
    
    c.setFont("Helvetica", 12)
    c.setFillColor(HexColor("#000000"))
    y = height - 140
    for line in report_data['summary'].split('. '):
        if line.strip():
            c.drawString(50, y, line.strip() + '.')
            y -= 20
    
    # Key metrics box
    c.setStrokeColor(HexColor("#4472C4"))
    c.setLineWidth(2)
    c.rect(50, y - 80, width - 100, 100)
    
    c.setFont("Helvetica-Bold", 14)
    c.setFillColor(HexColor("#4472C4"))
    c.drawString(60, y - 70, "Key Metrics")
    
    c.setFont("Helvetica", 11)
    c.setFillColor(HexColor("#000000"))
    metric_y = y - 95
    for metric, value in report_data['metrics'].items():
        c.drawString(60, metric_y, f"• {metric}: {value}")
        metric_y -= 20
    
    c.save()

# Usage
report_data = {
    "title": "Q3 2024 Quarterly Report",
    "period": "July - September 2024",
    "summary": "Our company achieved strong growth in Q3 2024. Revenue increased by 15% compared to the previous quarter. Customer satisfaction scores improved significantly. We successfully launched two new products and expanded into the European market.",
    "metrics": {
        "Revenue": "$2.5M (+15% YoY)",
        "New Customers": "500",
        "Retention Rate": "85%",
        "NPS Score": "72"
    }
}

create_quarterly_report(report_data)
```

## Example 4: Data-Driven Document

```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
from reportlab.lib.colors import HexColor

def create_data_document(data_dict):
    c = canvas.Canvas("data_document.pdf", pagesize=letter)
    width, height = letter
    
    # Title
    c.setFont("Helvetica-Bold", 24)
    c.setFillColor(HexColor("#1F4E79"))
    c.drawString(50, height - 50, "Data Analysis Report")
    c.showPage()
    
    # Create a section for each category
    for category, items in data_dict.items():
        # Section divider
        c.setFillColor(HexColor("#4472C4"))
        c.rect(0, 0, width, height, fill=1, stroke=0)
        
        c.setFont("Helvetica-Bold", 48)
        c.setFillColor(HexColor("#FFFFFF"))
        title_width = c.stringWidth(category, "Helvetica-Bold", 48)
        c.drawString((width - title_width) / 2, height / 2 - 24, category)
        c.showPage()
        
        # Content page
        c.setFont("Helvetica-Bold", 20)
        c.setFillColor(HexColor("#1F4E79"))
        c.drawString(50, height - 50, category)
        
        y = height - 90
        c.setFont("Helvetica", 12)
        c.setFillColor(HexColor("#000000"))
        
        for item in items:
            c.drawString(70, y, f"• {item}")
            y -= 20
            
            # Check if we need a new page
            if y < 100:
                c.showPage()
                y = height - 50
        
        c.showPage()
    
    c.save()

# Usage
data = {
    "Sales Overview": [
        "Total Sales: $4.2M",
        "Growth Rate: 12%",
        "Top Product: Widget Pro",
        "Best Region: North America"
    ],
    "Customer Analysis": [
        "Total Customers: 15,000",
        "New Acquisitions: 2,500",
        "Churn Rate: 3.2%",
        "Customer Satisfaction: 4.5/5"
    ],
    "Financial Health": [
        "Profit Margin: 18%",
        "Operating Expenses: $800K",
        "Cash Flow: Positive",
        "Debt-to-Equity: 0.4"
    ]
}

create_data_document(data)
```

## Example 5: Reading PDF with pdfplumber

```python
import pdfplumber

def analyze_pdf(pdf_path):
    """Extract and analyze PDF content"""
    with pdfplumber.open(pdf_path) as pdf:
        print(f"Total pages: {len(pdf.pages)}")
        print(f"Metadata: {pdf.metadata}")
        
        # Extract text from all pages
        full_text = ""
        for i, page in enumerate(pdf.pages, 1):
            text = page.extract_text()
            if text:
                print(f"\n--- Page {i} ---")
                print(text)
                full_text += text
        
        # Extract tables if any
        tables = []
        for page in pdf.pages:
            table = page.extract_table()
            if table:
                tables.append(table)
        
        if tables:
            print(f"\nFound {len(tables)} table(s)")
            for i, table in enumerate(tables, 1):
                print(f"\n--- Table {i} ---")
                for row in table:
                    print(row)
        
        return full_text, tables

# Usage
text, tables = analyze_pdf("document.pdf")
```
## Example 6: Certificate Generation

```python
from reportlab.lib.pagesizes import landscape, letter
from reportlab.pdfgen import canvas
from reportlab.lib.colors import HexColor
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont

def create_certificate(name, course, date):
    # Use landscape for certificate
    c = canvas.Canvas(f"certificate_{name.replace(' ', '_')}.pdf", 
                      pagesize=landscape(letter))
    width, height = landscape(letter)
    
    # Border
    c.setStrokeColor(HexColor("#D4AF37"))  # Gold
    c.setLineWidth(3)
    c.rect(20, 20, width - 40, height - 40)
    
    # Inner border
    c.setLineWidth(1)
    c.rect(30, 30, width - 60, height - 60)
    
    # Title
    c.setFont("Times-Bold", 48)
    c.setFillColor(HexColor("#1F4E79"))
    title_width = c.stringWidth("Certificate of Completion", "Times-Bold", 48)
    c.drawString((width - title_width) / 2, height - 150, "Certificate of Completion")
    
    # Present to
    c.setFont("Times-Italic", 24)
    c.setFillColor(HexColor("#000000"))
    present_width = c.stringWidth("This is to certify that", "Times-Italic", 24)
    c.drawString((width - present_width) / 2, height - 220, "This is to certify that")
    
    # Name
    c.setFont("Times-Bold", 36)
    c.setFillColor(HexColor("#1F4E79"))
    name_width = c.stringWidth(name, "Times-Bold", 36)
    c.drawString((width - name_width) / 2, height - 280, name)
    
    # Course
    c.setFont("Times-Italic", 24)
    c.setFillColor(HexColor("#000000"))
    has_completed = "has successfully completed the course"
    completed_width = c.stringWidth(has_completed, "Times-Italic", 24)
    c.drawString((width - completed_width) / 2, height - 330, has_completed)
    
    c.setFont("Times-Bold", 28)
    c.setFillColor(HexColor("#4472C4"))
    course_width = c.stringWidth(course, "Times-Bold", 28)
    c.drawString((width - course_width) / 2, height - 380, course)
    
    # Date and signature area
    c.setFont("Times", 14)
    c.setFillColor(HexColor("#595959"))
    c.drawString(150, 150, f"Date: {date}")
    
    c.setFont("Times-Italic", 12)
    c.drawString(width - 250, 150, "Director")
    c.setFont("Times-Bold", 12)
    c.drawString(width - 250, 135, "[Signature]")
    
    c.save()

# Usage
create_certificate(
    name="John Doe",
    course="Advanced Python Programming",
    date="January 10, 2024"
)
```

## Example 7: Chinese Document Generation

```python
from reportlab.lib.pagesizes import A4
from reportlab.pdfgen import canvas
from reportlab.lib.colors import HexColor, blue, black, gray
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
from fontTools.ttLib import TTFont as FontTool
import subprocess

def setup_chinese_font():
    """Setup Chinese font for PDF creation"""
    try:
        # Install WQY Microhei
        subprocess.run(['apt-get', 'update', '-qq'], capture_output=True)
        subprocess.run(['apt-get', 'install', '-y', 'fonts-wqy-microhei'], 
                      capture_output=True)
        
        # Extract from TTC
        ttc_path = '/usr/share/fonts/truetype/wqy/wqy-microhei.ttc'
        output_path = '/home/user/.fonts/wqy-microhei.ttf'
        font = FontTool(ttc_path, fontNumber=0)
        font.save(output_path)
        
        # Register fonts
        pdfmetrics.registerFont(TTFont('CJK', output_path))
        pdfmetrics.registerFont(TTFont('CJK-Bold', output_path))
        
        return 'CJK', 'CJK-Bold'
    except Exception as e:
        print(f"Warning: {e}")
        return 'Helvetica', 'Helvetica-Bold'

def create_chinese_document(title, content_list, author):
    """Create a Chinese language PDF document"""
    c = canvas.Canvas("chinese_document.pdf", pagesize=A4)
    width, height = A4
    
    # Setup Chinese fonts
    regular_font, bold_font = setup_chinese_font()
    
    # Title
    c.setFont(bold_font, 24)
    c.setFillColor(HexColor("#1F4E79"))
    title_width = c.stringWidth(title, bold_font, 24)
    c.drawString((width - title_width) / 2, height - 80, title)
    
    # Author
    c.setFont(regular_font, 12)
    c.setFillColor(HexColor("#666666"))
    c.drawString((width - c.stringWidth(author, regular_font, 12)) / 2, 
                height - 120, f"作者: {author}")
    
    # Separator line
    c.setStrokeColor(HexColor("#CCCCCC"))
    c.setLineWidth(1)
    c.line(50, height - 140, width - 50, height - 140)
    
    # Content
    y = height - 180
    c.setFont(regular_font, 12)
    c.setFillColor(black)
    
    for line in content_list:
        c.drawString(70, y, line)
        y -= 20
        
        # Check for page break
        if y < 100:
            c.showPage()
            y = height - 80
            c.setFont(regular_font, 12)
    
    # Footer
    c.setFont(regular_font, 10)
    c.setFillColor(gray)
    c.drawString(50, 50, "生成时间: 2024年1月")
    c.drawString(width - 50, 50, "页码: 1", 
                mode=2)  # 2 = drawRightString
    
    c.save()

# Usage
title = "人工智能与未来工作"
author = "张三"
content = [
    "人工智能正在改变我们的工作方式。",
    "",
    "随着技术的不断发展，AI在各个领域都发挥着重要作用。",
    "从医疗诊断到自动驾驶，从客服机器人到内容创作，",
    "AI的应用场景越来越广泛。",
    "",
    "但是，我们也面临着新的挑战：",
    "• 如何确保AI的决策公平性？",
    "• 如何处理AI带来的就业变化？",
    "• 如何保护个人隐私和数据安全？",
    "",
    "这些问题需要我们共同努力去解决。",
    "只有通过技术创新和制度完善，",
    "我们才能更好地迎接AI时代的到来。"
]

create_chinese_document(title, content, author)
```