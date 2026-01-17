---
name: simple-pdf-skill
description: PDF处理技能知识包：提供创建、编辑、提取、合并PDF的代码示例、最佳实践和工作流，需集成到Python代码中使用。
license: Apache 2.0. LICENSE.txt has complete terms
---

## Overview

This guide covers essential PDF processing operations using Python libraries and command-line tools.

**Core Libraries:**
- **reportlab** - Create new PDFs with text, shapes, and layouts
- **PyMuPDF (fitz)** - 首选编辑库：支持页面编辑、内容注入、合并、拆分及批注等修改任务
- **pypdf** - PDF manipulation: merge, split, rotate, encrypt
- **pdfplumber** - Extract text and tables from PDFs
- **pypdfium2** - Render PDF pages to images

### 场景识别指南

本技能包不仅用于静态读取，更侧重于对文档内容的主动变更。当AI分析用户需求时，若发现以下意图特征，应优先考虑调用修改类库（如 PyMuPDF/fitz）：
- **变更意图**：涉及在现有文档基础上添加、删除、替换或移动内容（如添加水印、合并文件、删除页面、插入图片）。
- **交互操作**：需要创建表单域、添加批注或设置超链接。
- **非纯读取**：需求不仅仅是“查看”或“提取文本”，而是要求产出一份与原文件不同的新文档。

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

## 🔴 中文支持（必读）

本包默认字体不支持中文，输出中文必须注册字体，否则必乱码。

**Required**: 使用reportlab创建包含中文的PDF前，必须：
1. 注册中文字体：`pdfmetrics.registerFont(TTFont(...))`
2. 使用注册的字体名：`setFont()`

**Recommended Fonts**:
- **WQY Microhei** (4.4MB) - 轻量快速，适合90%场景
- **Noto Sans SC** (~15MB) - 专业简体中文
- **Noto Sans CJK** (100MB+) - 完整CJK多语言支持

详细代码见 `reportlab-guide.md` (Fonts → Chinese Font Support)

## When to Apply

**Use when**:
- Creating automated reports from data
- Editing existing PDFs with highlights/annotations
- Drawing shapes and text on PDF pages
- Invoices/receipts generation
- Text-heavy documents required
- Simple layouts with basic graphics
- Python-only environment constraints
- Batch document processing
- PDF text/table extraction needed
- Merging/splitting PDFs
- Password protection or encryption

**Do NOT use when**:
- Complex HTML/CSS styling needed (use WeasyPrint or pdfkit)
- Rich media/interactive PDFs required (use commercial libraries)
- High-performance vector graphics needed
- Existing PDF templates with complex forms
## Quick Reference
| Task | Best Tool | Guide |
|------|-----------|-------|
| Create PDFs | reportlab | [reportlab-guide.md](reportlab-guide.md) |
| Create PDFs with **Chinese** text | reportlab + TTFont | [reportlab-guide.md](reportlab-guide.md) → Chinese Font Support |
| Edit existing PDFs | PyMuPDF (fitz) | [pymupdf-guide.md](pymupdf-guide.md) |
| Draw shapes/annotate | PyMuPDF (fitz) | [pymupdf-guide.md](pymupdf-guide.md) |
| Add highlights/annotations | PyMuPDF (fitz) | [pymupdf-guide.md](pymupdf-guide.md) |
| Merge PDFs | PyMuPDF or pypdf | [pymupdf-guide.md](pymupdf-guide.md) or [pypdf-guide.md](pypdf-guide.md) |
| Split PDFs | PyMuPDF or pypdf | [pymupdf-guide.md](pymupdf-guide.md) or [pypdf-guide.md](pypdf-guide.md) |
| Extract text | PyMuPDF or pdfplumber | [pymupdf-guide.md](pymupdf-guide.md) or [pdfplumber-guide.md](pdfplumber-guide.md) |
| Extract tables | PyMuPDF or pdfplumber | [pymupdf-guide.md](pymupdf-guide.md) or [pdfplumber-guide.md](pdfplumber-guide.md) |
| Extract images | PyMuPDF | [pymupdf-guide.md](pymupdf-guide.md) |
| Add watermarks | PyMuPDF or pypdf | [pymupdf-guide.md](pymupdf-guide.md) or [pypdf-guide.md](pypdf-guide.md) |
| Render to images | pypdfium2 | [pypdfium2-guide.md](pypdfium2-guide.md) |
| Password protection | pypdf | [pypdf-guide.md](pypdf-guide.md) |
| Rotate/crop pages | PyMuPDF or pypdf | [pymupdf-guide.md](pymupdf-guide.md) or [pypdf-guide.md](pypdf-guide.md) |
| Patterns | - | [PATTERNS.md](PATTERNS.md) |
| Examples | - | [EXAMPLES.md](EXAMPLES.md) 
## 𝘋𝘦𝘱𝘦𝘯𝘥𝘦𝘯𝘤𝘪𝘦𝘴
### 𝘙𝘦𝘲𝘶𝘪𝘳𝘦𝘥 𝘗𝘺𝘵𝘩𝘰𝘯 𝘓𝘪𝘣𝘳𝘢𝘳𝘪𝘦𝘴
```𝘣𝘢𝘴𝘩
𝘱𝘪𝘱 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘳𝘦𝘱𝘰𝘳𝘵𝘭𝘢𝘣
𝘱𝘪𝘱 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘱𝘺𝘮𝘶𝘱𝘥𝘧
𝘱𝘪𝘱 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘱𝘥𝘧𝘱𝘭𝘶𝘮𝘣𝘦𝘳
𝘱𝘪𝘱 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘱𝘺𝘱𝘥𝘧
𝘱𝘪𝘱 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘱𝘺𝘱𝘥𝘧𝘪𝘶𝘮2
𝘱𝘪𝘱 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘧𝘰𝘯𝘵𝘵𝘰𝘰𝘭𝘴  # 𝘍𝘰𝘳 𝘛𝘛𝘊 𝘧𝘰𝘯𝘵 𝘦𝘹𝘵𝘳𝘢𝘤𝘵𝘪𝘰𝘯
```

### 𝘖𝘱𝘵𝘪𝘰𝘯𝘢𝘭 𝘧𝘰𝘳 𝘈𝘥𝘷𝘢𝘯𝘤𝘦𝘥 𝘍𝘦𝘢𝘵𝘶𝘳𝘦𝘴
```𝘣𝘢𝘴𝘩
# 𝘍𝘰𝘳 𝘖𝘊𝘙 (𝘴𝘤𝘢𝘯𝘯𝘦𝘥 𝘗𝘋𝘍𝘴)
𝘱𝘪𝘱 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘱𝘺𝘵𝘦𝘴𝘴𝘦𝘳𝘢𝘤𝘵 𝘱𝘥𝘧2𝘪𝘮𝘢𝘨𝘦
𝘴𝘶𝘥𝘰 𝘢𝘱𝘵-𝘨𝘦𝘵 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘵𝘦𝘴𝘴𝘦𝘳𝘢𝘤𝘵-𝘰𝘤𝘳

# 𝘍𝘰𝘳 𝘤𝘰𝘮𝘮𝘢𝘯𝘥-𝘭𝘪𝘯𝘦 𝘵𝘰𝘰𝘭𝘴
𝘴𝘶𝘥𝘰 𝘢𝘱𝘵-𝘨𝘦𝘵 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘱𝘰𝘱𝘱𝘭𝘦𝘳-𝘶𝘵𝘪𝘭𝘴
𝘴𝘶𝘥𝘰 𝘢𝘱𝘵-𝘨𝘦𝘵 𝘪𝘯𝘴𝘵𝘢𝘭𝘭 𝘲𝘱𝘥𝘧
```
## 𝘒𝘦𝘺 𝘙𝘶𝘭𝘦𝘴

✅ 𝘜𝘴𝘦 `𝘊𝘢𝘯𝘷𝘢𝘴` 𝘤𝘰𝘰𝘳𝘥𝘪𝘯𝘢𝘵𝘦 𝘴𝘺𝘴𝘵𝘦𝘮 (0,0 𝘢𝘵 𝘣𝘰𝘵𝘵𝘰𝘮-𝘭𝘦𝘧𝘵) 𝘧𝘰𝘳 𝘳𝘦𝘱𝘰𝘳𝘵𝘭𝘢𝘣
✅ 𝘜𝘴𝘦 `𝘙𝘎𝘉𝘊𝘰𝘭𝘰𝘳(𝘳,𝘨,𝘣)` 𝘰𝘳 𝘩𝘦𝘹 𝘴𝘵𝘳𝘪𝘯𝘨𝘴 𝘧𝘰𝘳 𝘳𝘦𝘱𝘰𝘳𝘵𝘭𝘢𝘣
✅ 𝘜𝘴𝘦 `𝘗𝘵()` 𝘧𝘰𝘳 𝘧𝘰𝘯𝘵 𝘴𝘪𝘻𝘦𝘴, `𝘐𝘯𝘤𝘩()` 𝘧𝘰𝘳 𝘱𝘰𝘴𝘪𝘵𝘪𝘰𝘯𝘪𝘯𝘨 𝘪𝘯 𝘳𝘦𝘱𝘰𝘳𝘵𝘭𝘢𝘣
✅ 𝘊𝘩𝘦𝘤𝘬 𝘴𝘵𝘳𝘪𝘯𝘨 𝘭𝘦𝘯𝘨𝘵𝘩 𝘧𝘰𝘳 𝘰𝘷𝘦𝘳𝘧𝘭𝘰𝘸
✅ 𝘜𝘴𝘦 `𝘴𝘩𝘰𝘸𝘗𝘢𝘨𝘦()` 𝘵𝘰 𝘤𝘳𝘦𝘢𝘵𝘦 𝘯𝘦𝘸 𝘱𝘢𝘨𝘦𝘴
✅ 𝘈𝘭𝘸𝘢𝘺𝘴 𝘤𝘢𝘭𝘭 `𝘴𝘢𝘷𝘦()` 𝘵𝘰 𝘧𝘪𝘯𝘢𝘭𝘪𝘻𝘦 𝘥𝘰𝘤𝘶𝘮𝘦𝘯𝘵
✅ 𝘏𝘢𝘯𝘥𝘭𝘦 𝘦𝘯𝘤𝘳𝘺𝘱𝘵𝘦𝘥 𝘗𝘋𝘍𝘴 𝘨𝘳𝘢𝘤𝘦𝘧𝘶𝘭𝘭𝘺
✅ 𝘗𝘳𝘰𝘤𝘦𝘴𝘴 𝘭𝘢𝘳𝘨𝘦 𝘗𝘋𝘍𝘴 𝘪𝘯 𝘤𝘩𝘶𝘯𝘬𝘴
✅ 𝘗𝘺𝘔𝘶𝘗𝘋𝘍 𝘶𝘴𝘦𝘴 𝘙𝘎𝘉 𝘵𝘶𝘱𝘭𝘦𝘴 (0-1 𝘳𝘢𝘯𝘨𝘦), 𝘯𝘰𝘵 0-255
✅ 𝘈𝘭𝘸𝘢𝘺𝘴 𝘤𝘭𝘰𝘴𝘦 𝘗𝘺𝘔𝘶𝘗𝘋𝘍 𝘥𝘰𝘤𝘶𝘮𝘦𝘯𝘵𝘴 𝘵𝘰 𝘧𝘳𝘦𝘦 𝘳𝘦𝘴𝘰𝘶𝘳𝘤𝘦𝘴
✅ **𝘈𝘓𝘞𝘈𝘠𝘚 𝘳𝘦𝘨𝘪𝘴𝘵𝘦𝘳 𝘊𝘩𝘪𝘯𝘦𝘴𝘦 𝘧𝘰𝘯𝘵𝘴 𝘣𝘦𝘧𝘰𝘳𝘦 𝘶𝘴𝘪𝘯𝘨 𝘊𝘩𝘪𝘯𝘦𝘴𝘦 𝘵𝘦𝘹𝘵 𝘪𝘯 𝘳𝘦𝘱𝘰𝘳𝘵𝘭𝘢𝘣**
## 𝘈𝘥𝘥𝘪𝘵𝘪𝘰𝘯𝘢𝘭 𝘍𝘪𝘭𝘦𝘴

- [𝘗𝘈𝘛𝘛𝘌𝘙𝘕𝘚.𝘮𝘥](𝘗𝘈𝘛𝘛𝘌𝘙𝘕𝘚.𝘮𝘥) - 𝘊𝘰𝘳𝘦 𝘤𝘰𝘥𝘦 𝘱𝘢𝘵𝘵𝘦𝘳𝘯𝘴 𝘢𝘯𝘥 𝘧𝘶𝘯𝘤𝘵𝘪𝘰𝘯𝘴
- [𝘌𝘟𝘈𝘔𝘗𝘓𝘌𝘚.𝘮𝘥](𝘌𝘟𝘈𝘔𝘗𝘓𝘌𝘚.𝘮𝘥) - 𝘊𝘰𝘮𝘱𝘭𝘦𝘵𝘦 𝘸𝘰𝘳𝘬𝘪𝘯𝘨 𝘦𝘹𝘢𝘮𝘱𝘭𝘦𝘴
- [𝘙𝘌𝘍𝘌𝘙𝘌𝘕𝘊𝘌.𝘮𝘥](𝘙𝘌𝘍𝘌𝘙𝘌𝘕𝘊𝘌.𝘮𝘥) - 𝘊𝘰𝘭𝘰𝘳𝘴, 𝘧𝘰𝘯𝘵𝘴, 𝘲𝘶𝘪𝘤𝘬 𝘳𝘦𝘧𝘦𝘳𝘦𝘯𝘤𝘦
- [𝘚𝘊𝘌𝘕𝘈𝘙𝘐𝘖𝘚.𝘮𝘥](𝘚𝘊𝘌𝘕𝘈𝘙𝘐𝘖𝘚.𝘮𝘥) - 𝘙𝘦𝘢𝘭-𝘸𝘰𝘳𝘭𝘥 𝘣𝘶𝘴𝘪𝘯𝘦𝘴𝘴 𝘶𝘴𝘦 𝘤𝘢𝘴𝘦𝘴 𝘢𝘯𝘥 𝘤𝘰𝘮𝘱𝘭𝘦𝘵𝘦 𝘴𝘰𝘭𝘶𝘵𝘪𝘰𝘯𝘴
- [𝘞𝘖𝘙𝘒𝘍𝘓𝘖𝘞𝘚.𝘮𝘥](𝘞𝘖𝘙𝘒𝘍𝘓𝘖𝘞𝘚.𝘮𝘥) - 𝘓𝘪𝘣𝘳𝘢𝘳𝘺 𝘴𝘦𝘭𝘦𝘤𝘵𝘪𝘰𝘯, 𝘱𝘦𝘳𝘧𝘰𝘳𝘮𝘢𝘯𝘤𝘦 𝘣𝘦𝘯𝘤𝘩𝘮𝘢𝘳𝘬𝘴, 𝘢𝘯𝘥 𝘤𝘳𝘰𝘴𝘴-𝘭𝘪𝘣𝘳𝘢𝘳𝘺 𝘸𝘰𝘳𝘬𝘧𝘭𝘰𝘸𝘴
 [reportlab-guide.md](reportlab-guide.md) - Full API documentation for PDF creation (incl. multi-page docs, headers, TOC) |
| [chart-guide.md](chart-guide.md) - Chart and graphics generation using Matplotlib and ReportLab |
- [pymupdf-guide.md](pymupdf-guide.md) - Advanced PDF editing and manipulation with PyMuPDF
- [pypdf-guide.md](pypdf-guide.md) - PDF merging, splitting, bookmarks, forms, and metadata
- [pdfplumber-guide.md](pdfplumber-guide.md) - Reading PDFs documentation
- [pypdfium2-guide.md](pypdfium2-guide.md) - PDF to image rendering

important:Please read the document carefully and understand the meaning of each item to avoid encountering unnecessary errors during implementation.

Read PATTERNS.md for implementation patterns, EXAMPLES.md for usage demonstrations, and REFERENCE.md for styling references.