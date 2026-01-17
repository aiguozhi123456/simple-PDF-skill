# PDF Processing Scenarios - Real-World Use Cases

This document provides complete, production-ready solutions for common PDF processing tasks.

## Table of Contents

1. [Invoice Processing](#invoice-processing)
2. [Contract Generation and Signing](#contract-generation-and-signing)
3. [Report Automation](#report-automation)
4. [Document Archiving and OCR](#document-archiving-and-ocr)
5. [Batch Form Processing](#batch-form-processing)
6. [PDF to E-book Conversion](#pdf-to-ebook-conversion)
7. [Legal Document Review](#legal-document-review)

---

## Invoice Processing

### Scenario: Extract structured data from scanned PDF invoices and generate summary reports

```python
import pdfplumber
import re
from reportlab.lib.pagesizes import letter
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph, Spacer
from reportlab.lib import colors

def extract_invoice_data(pdf_path):
    """Extract structured data from invoice PDF"""
    data = {
        "invoice_number": "",
        "date": "",
        "total": 0.0,
        "line_items": []
    }
    
    with pdfplumber.open(pdf_path) as pdf:
        page = pdf.pages[0]
        text = page.extract_text()
        
        # Extract invoice number (e.g., "INV-2024-001")
        inv_match = re.search(r'INV-\d{4}-\d{3}', text)
        if inv_match:
            data["invoice_number"] = inv_match.group()
        
        # Extract date (e.g., "Date: 2024-01-15")
        date_match = re.search(r'Date:\s*(\d{4}-\d{2}-\d{2})', text)
        if date_match:
            data["date"] = date_match.group(1)
        
        # Extract total amount (e.g., "Total: $1,234.56")
        total_match = re.search(r'Total:\s*\$?([\d,]+\.?\d*)', text)
        if total_match:
            data["total"] = float(total_match.group(1).replace(',', ''))
        
        # Extract line items from table
        tables = page.extract_tables()
        if tables:
            for table in tables:
                data["line_items"] = [row for row in table if row]
    
    return data

def generate_invoice_report(invoices_data, output_path):
    """Generate summary report from multiple invoices"""
    doc = SimpleDocTemplate(output_path, pagesize=letter)
    story = []
    styles = getSampleStyleSheet()
    
    # Title
    title = Paragraph("Invoice Summary Report", styles["Title"])
    story.append(title)
    story.append(Spacer(1, 12))
    
    # Summary table
    table_data = [["Invoice #", "Date", "Total"]]
    total_sum = 0.0
    
    for inv in invoices_data:
        table_data.append([
            inv["invoice_number"],
            inv["date"],
            f"${inv['total']:.2f}"
        ])
        total_sum += inv["total"]
    
    table_data.append(["", "Grand Total:", f"${total_sum:.2f}"])
    
    table = Table(table_data, colWidths=[100, 100, 100])
    table.setStyle(TableStyle([
        ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
        ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
        ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
        ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
        ('FONTSIZE', (0, 0), (-1, 0), 12),
        ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
        ('BACKGROUND', (0, 1), (-1, -1), colors.beige),
        ('GRID', (0, 0), (-1, -1), 1, colors.black),
        ('BACKGROUND', (0, -1), (-1, -1), colors.lightgrey),
        ('FONTNAME', (0, -1), (-1, -1), 'Helvetica-Bold'),
    ]))
    
    story.append(table)
    doc.build(story)

# Usage
invoices = [extract_invoice_data(f"invoice_{i}.pdf") for i in range(1, 6)]
generate_invoice_report(invoices, "invoice_summary.pdf")
```

---

## Contract Generation and Signing

### Scenario: Generate legal contracts with signature placeholders and process signed versions

```python
import fitz  # PyMuPDF
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
from reportlab.lib.units import inch
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
import os

# Setup Chinese font (use from previous guide)
def setup_chinese_font():
    font_path = "/storage/emulated/0/Download/Operit/fonts/wqy-microhei.ttf"
    if os.path.exists(font_path):
        pdfmetrics.registerFont(TTFont('CJK', font_path))
        return 'CJK'
    return 'Helvetica'

def create_contract_template(output_path, company_name, contract_type="Service"):
    """Generate contract with signature placeholders"""
    c = canvas.Canvas(output_path, pagesize=letter)
    font = setup_chinese_font()
    
    # Company header
    c.setFont(font, 18)
    c.drawString(72, 750, company_name)
    
    # Contract title
    c.setFont(font, 14)
    c.drawString(72, 720, f"{contract_type} Agreement")
    
    # Contract body
    c.setFont(font, 11)
    body_text = """
    This agreement is entered into between the parties as of the date of signing.
    
    1. Services: The service provider agrees to provide services as specified.
    2. Payment: The client agrees to pay the specified amount upon completion.
    3. Term: This agreement shall remain in effect until all obligations are fulfilled.
    4. Termination: Either party may terminate with 30 days written notice.
    """
    
    y_pos = 680
    for line in body_text.strip().split('\n'):
        c.drawString(72, y_pos, line.strip())
        y_pos -= 15
    
    # Signature area
    c.line(72, 150, 250, 150)
    c.setFont(font, 10)
    c.drawString(72, 140, "Provider Signature")
    
    c.line(350, 150, 528, 150)
    c.drawString(350, 140, "Client Signature")
    
    # Date fields
    c.line(72, 110, 200, 110)
    c.drawString(72, 100, "Date")
    
    c.line(350, 110, 478, 110)
    c.drawString(350, 100, "Date")
    
    c.save()

def verify_signature_areas(pdf_path):
    """Check if signature areas are properly filled"""
    doc = fitz.open(pdf_path)
    
    # Define signature areas (based on template)
    provider_sig_area = fitz.Rect(72, 140, 250, 150)
    client_sig_area = fitz.Rect(350, 140, 528, 150)
    
    # Check for content in signature areas
    page = doc[0]
    
    provider_filled = False
    client_filled = False
    
    # Check for text or drawings in signature areas
    words = page.extract_words()
    for word in words:
        x0, y0, x1, y1 = word["x0"], word["bottom"], word["x1"], word["top"]
        word_rect = fitz.Rect(x0, y0, x1, y1)
        
        if word_rect.intersects(provider_sig_area):
            provider_filled = True
        if word_rect.intersects(client_sig_area):
            client_filled = True
    
    doc.close()
    
    return {
        "provider_signed": provider_filled,
        "client_signed": client_filled,
        "fully_signed": provider_filled and client_filled
    }

# Usage
create_contract_template("contract_template.pdf", "ABC Company")
status = verify_signature_areas("signed_contract.pdf")
print(f"Contract status: {status}")
```

---

## Report Automation

### Scenario: Automated weekly report generation from data sources with charts

```python
import matplotlib.pyplot as plt
from reportlab.lib.pagesizes import letter, A4
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph, Spacer, Image
from reportlab.lib import colors
from reportlab.lib.units import inch
import tempfile
import os

# Set matplotlib backend to Agg for non-interactive use
import matplotlib
matplotlib.use('Agg')

def create_charts(data):
    """Generate charts and save as temporary images"""
    chart_files = []
    
    # Sales chart
    fig, ax = plt.subplots(figsize=(8, 4))
    months = data["months"]
    sales = data["sales"]
    
    ax.bar(months, sales, color='steelblue')
    ax.set_title('Monthly Sales')
    ax.set_ylabel('Amount ($)')
    plt.tight_layout()
    
    sales_chart = tempfile.NamedTemporaryFile(suffix='.png', delete=False)
    plt.savefig(sales_chart.name, dpi=150, bbox_inches='tight')
    chart_files.append(sales_chart.name)
    plt.close()
    
    # Performance chart
    fig, ax = plt.subplots(figsize=(8, 4))
    categories = data["categories"]
    values = data["performance"]
    
    ax.plot(categories, values, marker='o', linestyle='-', color='darkorange')
    ax.set_title('Performance Metrics')
    plt.xticks(rotation=45)
    plt.tight_layout()
    
    perf_chart = tempfile.NamedTemporaryFile(suffix='.png', delete=False)
    plt.savefig(perf_chart.name, dpi=150, bbox_inches='tight')
    chart_files.append(perf_chart.name)
    plt.close()
    
    return chart_files

def generate_weekly_report(data, output_path):
    """Generate comprehensive weekly report"""
    doc = SimpleDocTemplate(output_path, pagesize=A4)
    story = []
    styles = getSampleStyleSheet()
    
    # Cover section
    title = Paragraph(f"Weekly Report - Week {data['week_number']}", styles["Title"])
    story.append(title)
    
    date_para = Paragraph(f"Generated: {data['report_date']}", styles["Normal"])
    story.append(date_para)
    story.append(Spacer(1, 0.5*inch))
    
    # Executive summary
    story.append(Paragraph("Executive Summary", styles["Heading2"]))
    summary = Paragraph(data["executive_summary"], styles["Normal"])
    story.append(summary)
    story.append(Spacer(1, 0.3*inch))
    
    # KPI table
    story.append(Paragraph("Key Performance Indicators", styles["Heading2"]))
    
    kpi_data = [["Metric", "Value", "Change"]]
    for kpi in data["kpis"]:
        kpi_data.append([
            kpi["name"],
            f"{kpi['value']}",
            f"{kpi['change']:+.1f}%"
        ])
    
    kpi_table = Table(kpi_data, colWidths=[2*inch, 1.5*inch, 1*inch])
    kpi_table.setStyle(TableStyle([
        ('BACKGROUND', (0, 0), (-1, 0), colors.HexColor("#4472C4")),
        ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
        ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
        ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
        ('FONTSIZE', (0, 0), (-1, 0), 10),
        ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
        ('GRID', (0, 0), (-1, -1), 1, colors.grey),
    ]))
    story.append(kpi_table)
    story.append(Spacer(1, 0.5*inch))
    
    # Charts section
    story.append(Paragraph("Visual Analytics", styles["Heading2"]))
    
    chart_files = create_charts(data)
    for chart_file in chart_files:
        img = Image(chart_file, width=6*inch, height=3*inch)
        story.append(img)
        story.append(Spacer(1, 0.3*inch))
    
    # Build document
    doc.build(story)
    
    # Cleanup temporary chart files
    for f in chart_files:
        try:
            os.unlink(f)
        except:
            pass

# Sample data
sample_data = {
    "week_number": 52,
    "report_date": "2024-12-30",
    "executive_summary": "This week shows strong performance across all metrics with significant growth in sales and customer satisfaction.",
    "kpis": [
        {"name": "Revenue", "value": "$125,430", "change": 15.3},
        {"name": "New Customers", "value": "234", "change": 8.7},
        {"name": "Satisfaction", "value": "94%", "change": 2.1},
        {"name": "Retention", "value": "87%", "change": -1.2}
    ],
    "months": ["Oct", "Nov", "Dec", "Jan"],
    "sales": [85000, 92000, 110000, 125430],
    "categories": ["Speed", "Quality", "Support", "Price", "Reliability"],
    "performance": [85, 92, 88, 78, 95]
}

generate_weekly_report(sample_data, "weekly_report.pdf")
```

---

## Document Archiving and OCR

### Scenario: Process scanned documents, extract text using OCR, and organize archive
```python
import pdfplumber
import pypdfium2 as pdfium
from PIL import Image
import pytesseract
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
import os
import json
import fitz  # PyMuPDF

def check_if_scanned(pdf_path):
    """Check if PDF is scanned (contains no extractable text)"""
    with pdfplumber.open(pdf_path) as pdf:
        for page in pdf.pages:
            text = page.extract_text()
            if text and len(text.strip()) > 100:
                return False
    return True

def ocr_pdf_to_text(pdf_path, output_txt_path):
    """Extract text from scanned PDF using OCR"""
    pdf = pdfium.PdfDocument(pdf_path)
    full_text = ""
    
    for i, page in enumerate(pdf):
        # Render page to image
        bitmap = page.render(scale=2.0)
        img = bitmap.to_pil()
        
        # Perform OCR
        text = pytesseract.image_to_string(img, lang='eng+chi_sim')
        full_text += f"--- Page {i+1} ---\n{text}\n\n"
    
    with open(output_txt_path, 'w', encoding='utf-8') as f:
        f.write(full_text)
    
    return full_text

def create_searchable_pdf(original_pdf, ocr_text, output_pdf):
    """Create searchable PDF by layering OCR text over original"""
    doc = fitz.open(original_pdf)
    
    for i, page in enumerate(doc):
        # Split text by pages (simple split by page markers)
        page_text = ocr_text.split(f"--- Page {i+1} ---")[1].split("--- Page")[0] if f"--- Page {i+1} ---" in ocr_text else ""
        
        # In production, you would need to position text more precisely
        # This is a simplified example
        if page_text.strip():
            rect = fitz.Rect(50, 50, 550, 750)
            page.insert_textbox(rect, page_text, fontsize=8, color=(0.9, 0.9, 0.9))
    
    doc.save(output_pdf)
    doc.close()

def create_archive_index(documents_data, output_path):
    """Generate archive index PDF"""
    c = canvas.Canvas(output_path, pagesize=letter)
    
    c.setFont("Helvetica-Bold", 18)
    c.drawString(72, 750, "Document Archive Index")
    
    c.setFont("Helvetica", 12)
    y_pos = 720
    
    c.drawString(72, y_pos, "Generated: " + documents_data["date"])
    y_pos -= 30
    
    c.drawString(72, y_pos, "-" * 70)
    y_pos -= 20
    
    for doc in documents_data["documents"]:
        c.drawString(72, y_pos, f"Filename: {doc['filename']}")
        y_pos -= 15
        c.drawString(100, y_pos, f"Type: {doc['type']} | Status: {doc['status']} | Size: {doc['size']}")
        y_pos -= 15
        c.drawString(100, y_pos, f"Keywords: {doc['keywords']}")
        y_pos -= 25
    
    c.save()

# Usage workflow
def process_document_archive(input_dir, output_dir):
    """Complete document archiving workflow"""
    os.makedirs(output_dir, exist_ok=True)
    
    documents_data = {
        "date": "2024-12-30",
        "documents": []
    }
    
    for filename in os.listdir(input_dir):
        if filename.endswith('.pdf'):
            pdf_path = os.path.join(input_dir, filename)
            
            # Check if scanned
            is_scanned = check_if_scanned(pdf_path)
            
            doc_info = {
                "filename": filename,
                "type": "Scanned" if is_scanned else "Digital",
                "status": "Processed",
                "size": f"{os.path.getsize(pdf_path) / 1024:.1f} KB",
                "keywords": ""
            }
            
            if is_scanned:
                # OCR processing
                txt_path = os.path.join(output_dir, filename.replace('.pdf', '_ocr.txt'))
                ocr_text = ocr_pdf_to_text(pdf_path, txt_path)
                
                # Create searchable version
                searchable_path = os.path.join(output_dir, filename.replace('.pdf', '_searchable.pdf'))
                create_searchable_pdf(pdf_path, ocr_text, searchable_path)
                
                # Extract keywords from OCR text
                words = ocr_text.split()
                doc_info["keywords"] = ", ".join(words[:10])
            else:
                # Extract text directly
                with pdfplumber.open(pdf_path) as pdf:
                    text = pdf.pages[0].extract_text()
                    doc_info["keywords"] = ", ".join(text.split()[:10])
            
            documents_data["documents"].append(doc_info)
    
    # Generate index
    create_archive_index(documents_data, os.path.join(output_dir, "archive_index.pdf"))

# process_document_archive("input_documents/", "archive/")
```

---

## Batch Form Processing

### Scenario: Fill multiple PDF forms with data from CSV

```python
import fitz  # PyMuPDF
import csv
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter

def create_form_template(output_path):
    """Create a simple PDF form with text fields"""
    c = canvas.Canvas(output_path, pagesize=letter)
    
    # Form title
    c.setFont("Helvetica-Bold", 16)
    c.drawString(200, 750, "Employee Information Form")
    
    # Form fields (positions for filling)
    c.setFont("Helvetica", 11)
    labels = [
        ("Name:", 100, 680),
        ("Email:", 100, 640),
        ("Phone:", 100, 600),
        ("Department:", 100, 560),
        ("Position:", 100, 520)
    ]
    
    for label, x, y in labels:
        c.drawString(x, y, label)
        # Draw field box for reference
        c.rect(x + 100, y - 5, x + 300, y + 10, fill=0)
    
    # Footer
    c.setFont("Helvetica-Oblique", 9)
    c.drawString(100, 100, "Please review all information before submitting.")
    
    c.save()

def fill_pdf_form(template_path, data, output_path):
    """Fill PDF form fields with data"""
    doc = fitz.open(template_path)
    page = doc[0]
    
    # Field positions (must match template)
    field_positions = {
        "name": (200, 680),
        "email": (200, 640),
        "phone": (200, 600),
        "department": (200, 560),
        "position": (200, 520)
    }
    
    # Fill fields
    for field, value in data.items():
        if field in field_positions and value:
            x, y = field_positions[field]
            # Clear area first (draw white rectangle)
            page.draw_rect(fitz.Rect(x, y - 5, x + 300, y + 10), color=(1, 1, 1), fill=(1, 1, 1))
            # Insert text
            page.insert_text(fitz.Point(x, y), value, fontsize=11)
    
    doc.save(output_path)
    doc.close()

def batch_process_forms(csv_path, template_path, output_dir):
    """Process multiple forms from CSV data"""
    os.makedirs(output_dir, exist_ok=True)
    
    with open(csv_path, 'r', newline='', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        
        for i, row in enumerate(reader, 1):
            data = {
                "name": row.get("name", ""),
                "email": row.get("email", ""),
                "phone": row.get("phone", ""),
                "department": row.get("department", ""),
                "position": row.get("position", "")
            }
            
            output_path = os.path.join(output_dir, f"form_{i:03d}_{row['name'].replace(' ', '_')}.pdf")
            fill_pdf_form(template_path, data, output_path)
            print(f"Generated: {output_path}")

# Usage
# First create template
# create_form_template("form_template.pdf")
#
# Then process CSV
# batch_process_forms("employees.csv", "form_template.pdf", "filled_forms/")
```

---

## PDF to E-book Conversion

### Scenario: Convert PDF documents to reflowable e-book format (simplified)

```python
import pdfplumber
import re
from pathlib import Path

def extract_chapter_structure(pdf_path):
    """Analyze PDF to detect chapter structure"""
    chapters = []
    
    with pdfplumber.open(pdf_path) as pdf:
        for i, page in enumerate(pdf):
            text = page.extract_text()
            if text:
                # Look for chapter headings (e.g., "Chapter 1", "Chapter I")
                chapter_matches = re.finditer(r'(Chapter\s+\d+|CHAPTER\s+[IVX]+)\s*(.*?)\s*$', text, re.MULTILINE)
                
                for match in chapter_matches:
                    chapters.append({
                        "page": i + 1,
                        "title": match.group(),
                        "full_text": match.group()
                    })
    
    return chapters

def convert_to_markbook(pdf_path, output_path):
    """Convert PDF to Markdown book format"""
    with pdfplumber.open(pdf_path) as pdf:
        with open(output_path, 'w', encoding='utf-8') as md_file:
            md_file.write(f"# {Path(pdf_path).stem}\n\n")
            
            for i, page in enumerate(pdf):
                text = page.extract_text()
                if not text:
                    continue
                
                # Detect headings
                lines = text.split('\n')
                for line in lines:
                    line = line.strip()
                    if not line:
                        continue
                    
                    # Simple heading detection
                    if re.match(r'^CHAPTER\s+[IVX]+|^Chapter\s+\d+', line):
                        md_file.write(f"\n\n## {line}\n\n")
                    elif re.match(r'^[A-Z][A-Z\s]{5,}$', line):
                        md_file.write(f"\n### {line}\n\n")
                    else:
                        md_file.write(f"{line}\n")
                
                md_file.write("\n---\n\n")

def extract_images_for_ebook(pdf_path, output_dir):
    """Extract images from PDF for e-book"""
    import pypdfium2 as pdfium
    
    os.makedirs(output_dir, exist_ok=True)
    
    pdf = pdfium.PdfDocument(pdf_path)
    
    for i, page in enumerate(pdf):
        images = page.get_objects(filter=pdfium.FPDF_PAGEOBJ_IMAGE)
        
        for j, img in enumerate(images):
            try:
                bitmap = img.get_bitmap()
                pil_img = bitmap.to_pil()
                pil_img.save(f"{output_dir}/img_page{i+1}_{j+1}.png", "PNG")
            except:
                continue
    
    pdf.close()

# Usage
# convert_to_markbook("book.pdf", "book.md")
# extract_images_for_ebook("book.pdf", "book_images/")
```

---

## Legal Document Review

### Scenario: Highlight key legal terms and create annotated review document

```python
import fitz  # PyMuPDF
import re

# Key legal terms to highlight
LEGAL_TERMS = {
    "liability": colors.yellow,
    "indemnification": colors.red,
    "termination": colors.orange,
    "confidentiality": colors.green,
    "governing law": colors.blue,
    "force majeure": colors.purple,
    "intellectual property": colors.cyan
}

def highlight_legal_terms(pdf_path, output_path):
    """Highlight key legal terms in a contract"""
    doc = fitz.open(pdf_path)
    
    for page in doc:
        text = page.get_text()
        
        for term, color in LEGAL_TERMS.items():
            # Find all instances of the term (case-insensitive)
            instances = page.search_for(term, case_sensitive=False)
            
            for rect in instances:
                # Create highlight annotation
                annot = page.add_highlight_annot(rect)
                annot.set_colors(stroke=color)
                annot.update()
    
    doc.save(output_path)
    doc.close()

def extract_clause_summary(pdf_path):
    """Extract and summarize key clauses"""
    summary = {
        "liability": [],
        "termination": [],
        "governing_law": ""
    }
    
    with pdfplumber.open(pdf_path) as pdf:
        text = ""
        for page in pdf:
            text += page.extract_text() + "\n"
    
    # Extract liability clause
    liability_matches = re.findall(r'(?:liability|LIABILITY).{0,300}(?:\.|;)', text, re.IGNORECASE)
    summary["liability"] = liability_matches[:3]  # First 3 matches
    
    # Extract termination clause
    term_matches = re.findall(r'(?:termination|TERMINATION).{0,300}(?:\.|;)', text, re.IGNORECASE)
    summary["termination"] = term_matches[:3]
    
    # Extract governing law
    law_match = re.search(r'(?:governing\s*law|GOVERNING\s*LAW).{0,100}', text, re.IGNORECASE)
    if law_match:
        summary["governing_law"] = law_match.group()
    
    return summary

def generate_review_report(original_pdf, summary, output_path):
    """Generate review report with clause summaries"""
    from reportlab.lib.pagesizes import letter
    from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer
    from reportlab.lib.styles import getSampleStyleSheet
    from reportlab.lib import colors
    
    doc = SimpleDocTemplate(output_path, pagesize=letter)
    story = []
    styles = getSampleStyleSheet()
    
    # Title
    title = Paragraph("Legal Document Review", styles["Title"])
    story.append(title)
    story.append(Spacer(1, 0.5*inch))
    
    # Liability section
    story.append(Paragraph("Liability Clauses", styles["Heading2"]))
    for clause in summary["liability"]:
        story.append(Paragraph(clause, styles["Normal"]))
        story.append(Spacer(1, 0.1*inch))
    
    # Termination section
    story.append(Paragraph("Termination Clauses", styles["Heading2"]))
    for clause in summary["termination"]:
        story.append(Paragraph(clause, styles["Normal"]))
        story.append(Spacer(1, 0.1*inch))
    
    # Governing law
    story.append(Paragraph("Governing Law", styles["Heading2"]))
    story.append(Paragraph(summary["governing_law"], styles["Normal"]))
    
    doc.build(story)

# Complete workflow
def review_legal_document(input_pdf):
    """Complete legal document review workflow"""
    # Highlight legal terms
    highlight_legal_terms(input_pdf, "document_highlighted.pdf")
    
    # Extract clause summary
    summary = extract_clause_summary(input_pdf)
    
    # Generate review report
    generate_review_report(input_pdf, summary, "legal_review_report.pdf")
    
    print("Review complete. Files generated:")
    print("- document_highlighted.pdf (with highlighted terms)")
    print("- legal_review_report.pdf (clause summaries)")

# review_legal_document("contract.pdf")
```

---

## Best Practices for Scenario Implementation

1. **Error Handling**: Always wrap file operations in try-except blocks
2. **Resource Management**: Use context managers (`with` statements) when possible
3. **Memory Efficiency**: Process large PDFs page by page, not all at once
4. **Testing**: Test on sample files before batch processing
5. **Backup**: Always work on copies of important documents
6. **Logging**: Add logging for troubleshooting batch operations
7. **Validation**: Validate extracted data before using it
8. **Performance**: Use appropriate libraries for each task (see WORKFLOWS.md)

## See Also

- [WORKFLOWS.md](WORKFLOWS.md) - Cross-library workflows and decision trees
- [PATTERNS.md](PATTERNS.md) - Reusable code patterns
- [EXAMPLES.md](EXAMPLES.md) - Basic usage examples