# Chart and Graphics Generation Guide

This document covers comprehensive chart and graphics generation for PDF documents using Matplotlib, ReportLab, and Pandas.

## Table of Contents

1. [Matplotlib Charts](#matplotlib-charts)
2. [Pandas DataFrame Charts](#pandas-dataframe-charts)
3. [Custom Graphics with ReportLab](#custom-graphics-with-reportlab)
4. [External Image Processing](#external-image-processing)
5. [Complete Report Generation](#complete-report-generation)

---

## Matplotlib Charts

### Setup

```python
import matplotlib.pyplot as plt
import matplotlib
import tempfile
import os

# Set non-interactive backend (required for PDF generation)
matplotlib.use('Agg')
```

### Bar Chart

```python
def create_bar_chart(data, title, output_path, colors_list=None):
    """Create bar chart and save as image
    
    Args:
        data: dict {category: value}
        title: Chart title
        output_path: Path to save PNG
        colors_list: List of colors for bars (optional)
    """
    categories = list(data.keys())
    values = list(data.values())
    
    fig, ax = plt.subplots(figsize=(10, 5))
    
    bars = ax.bar(categories, values, color=colors_list or '#4472C4')
    ax.set_title(title, fontsize=14, fontweight='bold', pad=20)
    ax.set_ylabel('Value', fontsize=11)
    ax.set_xlabel('Category', fontsize=11)
    
    # Add value labels on bars
    for bar, value in zip(bars, values):
        height = bar.get_height()
        ax.text(bar.get_x() + bar.get_width()/2., height,
                f'{value:,.0f}',
                ha='center', va='bottom', fontsize=10, fontweight='bold')
    
    # Grid and style
    ax.grid(True, alpha=0.3, axis='y')
    ax.spines['top'].set_visible(False)
    ax.spines['right'].set_visible(False)
    
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches='tight', facecolor='white')
    plt.close()

# Example
bar_data = {"Q1": 100, "Q2": 120, "Q3": 110, "Q4": 130}
create_bar_chart(bar_data, "Quarterly Revenue", "bar_chart.png")
```

### Multi-Line Chart

```python
def create_line_chart(x_data, y_series, title, output_path):
    """Create multi-line chart
    
    Args:
        x_data: List of x-axis values
        y_series: dict {series_name: [values]}
        title: Chart title
        output_path: Path to save PNG
    """
    fig, ax = plt.subplots(figsize=(10, 5))
    
    colors_palette = ['#4472C4', '#FF6B6B', '#4ECDC4', '#FFE66D', '#95E1D3']
    markers = ['o', 's', '^', 'D', 'v']
    
    for i, (y_data, label) in enumerate(y_series.items()):
        ax.plot(x_data, y_data, marker=markers[i % len(markers)], 
                label=label, color=colors_palette[i % len(colors_palette)],
                linewidth=2, markersize=8, markeredgewidth=1.5)
    
    ax.set_title(title, fontsize=14, fontweight='bold', pad=20)
    ax.set_xlabel('Period', fontsize=11)
    ax.set_ylabel('Value', fontsize=11)
    ax.legend(loc='best', framealpha=0.9)
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches='tight', facecolor='white')
    plt.close()

# Example
months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]
line_data = {
    "Product A": [100, 110, 105, 120, 115, 130],
    "Product B": [80, 85, 90, 95, 100, 105]
}
create_line_chart(months, line_data, "Product Performance", "line_chart.png")
```

### Pie Chart

```python
def create_pie_chart(data, title, output_path, explode=None):
    """Create pie chart with optional exploded slice
    
    Args:
        data: dict {label: value}
        title: Chart title
        output_path: Path to save PNG
        explode: List of explosion values (optional)
    """
    fig, ax = plt.subplots(figsize=(8, 8))
    
    labels = list(data.keys())
    sizes = list(data.values())
    colors_palette = ['#4472C4', '#FF6B6B', '#4ECDC4', '#FFE66D', '#95E1D3']
    
    wedges, texts, autotexts = ax.pie(
        sizes, labels=labels, colors=colors_palette[:len(labels)],
        autopct='%1.1f%%', startangle=90, explode=explode,
        textprops={'fontsize': 10, 'fontweight': 'bold'}
    )
    
    # Style percentage text
    for autotext in autotexts:
        autotext.set_color('white')
        autotext.set_fontweight('bold')
        autotext.set_fontsize(11)
    
    ax.set_title(title, fontsize=14, fontweight='bold', y=1.05)
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches='tight', facecolor='white')
    plt.close()

# Example
pie_data = {"Product A": 35, "Product B": 25, "Product C": 20, "Product D": 20}
create_pie_chart(pie_data, "Market Share", "pie_chart.png", explode=[0.1, 0, 0, 0])
```

### Stacked Area Chart

```python
def create_area_chart(x_data, y_series, title, output_path):
    """Create stacked area chart
    
    Args:
        x_data: List of x-axis values
        y_series: dict {series_name: [values]}
        title: Chart title
        output_path: Path to save PNG
    """
    fig, ax = plt.subplots(figsize=(10, 5))
    
    colors_palette = ['#4472C4', '#4ECDC4', '#95E1D3']
    
    # Stacked area plot
    ax.stackplot(x_data, *[y_series[key] for key in y_series.keys()],
                 labels=y_series.keys(), colors=colors_palette, alpha=0.7)
    
    ax.set_title(title, fontsize=14, fontweight='bold', pad=20)
    ax.set_xlabel('Period', fontsize=11)
    ax.set_ylabel('Value', fontsize=11)
    ax.legend(loc='upper left', framealpha=0.9)
    ax.grid(True, alpha=0.3, axis='y')
    
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches='tight', facecolor='white')
    plt.close()

# Example
area_data = {
    "Sales": [50, 60, 55, 70, 65, 80],
    "Revenue": [40, 50, 45, 60, 55, 70]
}
create_area_chart(months, area_data, "Sales vs Revenue", "area_chart.png")
```

### Scatter Plot

```python
def create_scatter_plot(x_data, y_data, labels, title, output_path):
    """Create scatter plot with point labels
    
    Args:
        x_data: List of x values
        y_data: List of y values
        labels: List of labels for each point
        title: Chart title
        output_path: Path to save PNG
    """
    fig, ax = plt.subplots(figsize=(10, 6))
    
    scatter = ax.scatter(x_data, y_data, s=100, alpha=0.6,
                        c=range(len(x_data)), cmap='viridis')
    
    # Add labels for each point
    for i, (x, y, label) in enumerate(zip(x_data, y_data, labels)):
        ax.annotate(label, (x, y), fontsize=9, ha='center', va='bottom')
    
    ax.set_title(title, fontsize=14, fontweight='bold', pad=20)
    ax.set_xlabel('X Axis', fontsize=11)
    ax.set_ylabel('Y Axis', fontsize=11)
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches='tight', facecolor='white')
    plt.close()

# Example
scatter_x = [10, 20, 30, 40, 50]
scatter_y = [15, 35, 25, 45, 30]
scatter_labels = ["A", "B", "C", "D", "E"]
create_scatter_plot(scatter_x, scatter_y, scatter_labels, "Data Points", "scatter.png")
```

### Heatmap

```python
def create_heatmap(data, title, output_path):
    """Create heatmap with value annotations
    
    Args:
        data: 2D array or list of lists
        title: Chart title
        output_path: Path to save PNG
    """
    import numpy as np
    
    fig, ax = plt.subplots(figsize=(10, 6))
    
    im = ax.imshow(data, cmap='YlOrRd', aspect='auto')
    
    # Add colorbar
    cbar = plt.colorbar(im, ax=ax)
    cbar.set_label('Value', rotation=270, labelpad=15)
    
    # Set ticks
    ax.set_xticks(np.arange(len(data[0])))
    ax.set_yticks(np.arange(len(data)))
    ax.set_xticklabels([f'Col {i}' for i in range(len(data[0]))])
    ax.set_yticklabels([f'Row {i}' for i in range(len(data))])
    
    # Add value annotations
    for i in range(len(data)):
        for j in range(len(data[0])):
            text = ax.text(j, i, data[i][j],
                         ha="center", va="center", color="black", fontsize=9)
    
    ax.set_title(title, fontsize=14, fontweight='bold', pad=20)
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches='tight', facecolor='white')
    plt.close()

# Example
import numpy as np
heatmap_data = np.random.rand(5, 5) * 100
create_heatmap(heatmap_data, "Performance Matrix", "heatmap.png")
```

---

## Pandas DataFrame Charts

### DataFrame to Chart

```python
import pandas as pd

def create_dataframe_chart(df, x_col, y_cols, chart_type='bar', title='', output_path='chart.png'):
    """Create chart from pandas DataFrame
    
    Args:
        df: pandas DataFrame
        x_col: Column name for x-axis
        y_cols: List of column names for y-axis
        chart_type: 'bar', 'line', 'area', or 'pie'
        title: Chart title
        output_path: Path to save PNG
    """
    fig, ax = plt.subplots(figsize=(10, 5))
    
    if chart_type == 'bar':
        df.plot(kind='bar', x=x_col, y=y_cols, ax=ax, width=0.8)
    elif chart_type == 'line':
        df.plot(kind='line', x=x_col, y=y_cols, ax=ax, marker='o', markersize=6)
    elif chart_type == 'area':
        df.plot(kind='area', x=x_col, y=y_cols, ax=ax, alpha=0.7)
    elif chart_type == 'pie':
        df.set_index(x_col)[y_cols[0]].plot(kind='pie', ax=ax, autopct='%1.1f%%')
    
    ax.set_title(title, fontsize=14, fontweight='bold', pad=20)
    ax.legend(loc='best', framealpha=0.9)
    ax.grid(True, alpha=0.3, axis='y' if chart_type != 'pie' else False)
    
    plt.xticks(rotation=45, ha='right')
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches='tight', facecolor='white')
    plt.close()

# Example
df = pd.DataFrame({
    'Month': ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
    'Product A': [100, 120, 110, 130, 125, 140],
    'Product B': [80, 85, 90, 95, 100, 105],
    'Product C': [60, 70, 75, 80, 85, 90]
})

create_dataframe_chart(df, 'Month', ['Product A', 'Product B', 'Product C'], 
                   chart_type='line', title='Product Trends', output_path='df_chart.png')
```

---

## Custom Graphics with ReportLab

### Flowchart Diagram

```python
from reportlab.lib.shapes import Drawing, Rect, Circle, Line, String
from reportlab.lib import colors

def create_flowchart():
    """Create simple flowchart diagram using ReportLab shapes"""
    d = Drawing(400, 300)
    
    # Define colors
    box_color = colors.HexColor("#4472C4")
    start_color = colors.HexColor("#6CC24A")
    end_color = colors.HexColor("#FF6B6B")
    text_color = colors.white
    
    # Add boxes
    d.add(Rect(50, 200, 100, 50, fillColor=start_color, strokeColor=colors.black))
    d.add(String(100, 225, "Start", fontSize=12, fillColor=text_color, textAnchor='middle'))
    
    d.add(Rect(150, 200, 100, 50, fillColor=box_color, strokeColor=colors.black))
    d.add(String(200, 225, "Process", fontSize=12, fillColor=text_color, textAnchor='middle'))
    
    d.add(Rect(250, 200, 100, 50, fillColor=box_color, strokeColor=colors.black))
    d.add(String(300, 225, "Decision", fontSize=12, fillColor=text_color, textAnchor='middle'))
    
    d.add(Rect(150, 100, 100, 50, fillColor=box_color, strokeColor=colors.black))
    d.add(String(200, 125, "Action", fontSize=12, fillColor=text_color, textAnchor='middle'))
    
    d.add(Rect(150, 20, 100, 50, fillColor=end_color, strokeColor=colors.black))
    d.add(String(200, 45, "End", fontSize=12, fillColor=text_color, textAnchor='middle'))
    
    # Add arrows (lines)
    d.add(Line(100, 200, 100, 200, strokeColor=colors.black, strokeWidth=2))
    d.add(Line(250, 200, 250, 200, strokeColor=colors.black, strokeWidth=2))
    d.add(Line(200, 200, 200, 150, strokeColor=colors.black, strokeWidth=2))
    d.add(Line(200, 100, 200, 70, strokeColor=colors.black, strokeWidth=2))
    
    return d
```

### Organizational Chart

```python
def create_organizational_chart():
    """Create org chart structure"""
    d = Drawing(500, 300)
    
    box_width = 120
    box_height = 50
    box_color = colors.HexColor("#4472C4")
    
    # CEO (top center)
    d.add(Rect(190, 220, box_width, box_height, fillColor=box_color, strokeColor=colors.black, radius=5))
    d.add(String(250, 245, "CEO", fontSize=12, fillColor=colors.white, textAnchor='middle'))
    
    # Department heads (middle row)
    positions = [(50, 140, "HR"), (190, 140, "Finance"), (330, 140, "Tech")]
    for x, y, label in positions:
        d.add(Rect(x, y, box_width, box_height, fillColor=colors.HexColor("#6CC24A"), 
                  strokeColor=colors.black, radius=5))
        d.add(String(x + box_width/2, y + box_height/2, label, 
                   fontSize=11, fillColor=colors.white, textAnchor='middle'))
    
    # Team members (bottom row)
    team_positions = [(20, 60, "Recruiter"), (80, 60, "Training"),
                    (160, 60, "Accounting"), (220, 60, "Audit"),
                    (300, 60, "Dev"), (360, 60, "QA")]
    for x, y, label in team_positions:
        d.add(Rect(x, y, box_width/2, box_height/2, 
                  fillColor=colors.HexColor("#4ECDC4"), strokeColor=colors.black, radius=3))
        d.add(String(x + box_width/4, y + box_height/4, label, 
                   fontSize=9, fillColor=colors.white, textAnchor='middle'))
    
    # Connecting lines
    d.add(Line(250, 220, 250, 190, strokeColor=colors.black, strokeWidth=2))
    d.add(Line(110, 190, 390, 190, strokeColor=colors.black, strokeWidth=2))
    
    for dept_x, _, _ in positions:
        d.add(Line(dept_x, 190, dept_x, 140, strokeColor=colors.black, strokeWidth=2))
        d.add(Line(dept_x, 140, dept_x, 110, strokeColor=colors.black, strokeWidth=1))
    
    return d
```

### Custom Logo

```python
def create_custom_logo():
    """Create a simple logo using shapes"""
    d = Drawing(200, 200)
    
    # Background circle
    d.add(Circle(100, 100, 90, fillColor=colors.HexColor("#4472C4"), 
               strokeColor=colors.HexColor("#2F5597"), strokeWidth=3))
    
    # Inner shapes
    d.add(Rect(70, 70, 60, 60, fillColor=colors.white, strokeColor=colors.HexColor("#2F5597"), strokeWidth=2))
    
    # Triangle
    from reportlab.lib.shapes import Polygon
    triangle = Polygon([100, 75, 80, 115, 120, 115], 
                      fillColor=colors.HexColor("#FF6B6B"), strokeColor=colors.black, strokeWidth=1)
    d.add(triangle)
    
    # Text
    d.add(String(100, 50, "COMPANY", fontSize=14, fontName='Helvetica-Bold', 
               fillColor=colors.HexColor("#2F5597"), textAnchor='middle'))
    
    return d
```

---

## External Image Processing

### Optimized Image Insertion

```python
from reportlab.lib.utils import ImageReader
from reportlab.platypus import Image as PlatypusImage

def insert_image_optimized(canvas_obj, image_path, x, y, width, height, maintain_aspect=True):
    """Insert image with aspect ratio preservation and optimization
    
    Args:
        canvas_obj: ReportLab canvas object
        image_path: Path to image file
        x, y: Position
        width, height: Dimensions
        maintain_aspect: Whether to preserve aspect ratio
    """
    reader = ImageReader(image_path)
    img_width, img_height = reader.getSize()
    
    if maintain_aspect:
        # Calculate aspect ratio
        aspect_ratio = img_width / img_height
        
        if width is None and height is not None:
            width = height * aspect_ratio
        elif height is None and width is not None:
            height = width / aspect_ratio
    
    # Draw image
    canvas_obj.drawImage(
        image_path, x, y, width=width, height=height,
        preserveAspectRatio=True,
        mask='auto'
    )
```

### Image Grid Layout

```python
def insert_image_grid(canvas_obj, images_dir, cols=3, start_x=50, start_y=750):
    """Insert multiple images in a grid layout
    
    Args:
        canvas_obj: ReportLab canvas object
        images_dir: Directory containing images
        cols: Number of columns
        start_x, start_y: Starting position
    """
    import glob
    
    image_files = glob.glob(os.path.join(images_dir, "*.jpg")) + glob.glob(os.path.join(images_dir, "*.png"))
    
    img_width = 150
    img_height = 100
    gap_x = 20
    gap_y = 20
    
    x_pos = start_x
    y_pos = start_y
    col_count = 0
    
    for img_file in image_files[:12]:  # Limit to 12 images
        # Draw image with frame
        canvas_obj.setStrokeColor(colors.black)
        canvas_obj.setLineWidth(1)
        canvas_obj.rect(x_pos, y_pos, img_width, img_height)
        
        canvas_obj.drawImage(img_file, x_pos + 2, y_pos + 2, 
                          width=img_width - 4, height=img_height - 4)
        
        # Update position
        col_count += 1
        if col_count >= cols:
            x_pos = start_x
            y_pos -= (img_height + gap_y)
            col_count = 0
        else:
            x_pos += (img_width + gap_x)
```

---

## Complete Report Generation

### Comprehensive Charts Report

```python
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, PageBreak
from reportlab.lib.styles import getSampleStyleSheet

def generate_charts_report(output_path):
    """Generate complete report with multiple chart types
    
    Args:
        output_path: Path to save PDF
    """
    doc = SimpleDocTemplate(output_path, pagesize=letter)
    story = []
    styles = getSampleStyleSheet()
    
    # Title
    story.append(Paragraph("Comprehensive Charts Report", styles["Title"]))
    story.append(Paragraph("Generated with Matplotlib and ReportLab", styles["Normal"]))
    story.append(Spacer(1, 0.5*inch))
    
    chart_files = []
    
    # 1. Bar Chart
    bar_data = {"Q1": 100, "Q2": 120, "Q3": 110, "Q4": 130}
    bar_file = tempfile.NamedTemporaryFile(suffix='.png', delete=False)
    create_bar_chart(bar_data, "Quarterly Revenue", bar_file.name)
    chart_files.append(bar_file.name)
    story.append(Paragraph("Revenue by Quarter", styles["Heading2"]))
    story.append(PlatypusImage(bar_file.name, width=6*inch, height=3.5*inch))
    story.append(Spacer(1, 0.3*inch))
    
    # 2. Line Chart
    months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]
    line_data = {
        "Product A": [100, 110, 105, 120, 115, 130],
        "Product B": [80, 85, 90, 95, 100, 105]
    }
    line_file = tempfile.NamedTemporaryFile(suffix='.png', delete=False)
    create_line_chart(months, line_data, "Product Performance", line_file.name)
    chart_files.append(line_file.name)
    story.append(Paragraph("Product Performance Trend", styles["Heading2"]))
    story.append(PlatypusImage(line_file.name, width=6*inch, height=3.5*inch))
    story.append(Spacer(1, 0.3*inch))
    
    # 3. Pie Chart
    pie_data = {"Product A": 35, "Product B": 25, "Product C": 20, "Product D": 20}
    pie_file = tempfile.NamedTemporaryFile(suffix='.png', delete=False)
    create_pie_chart(pie_data, "Market Share", pie_file.name)
    chart_files.append(pie_file.name)
    story.append(Paragraph("Market Share Distribution", styles["Heading2"]))
    story.append(PlatypusImage(pie_file.name, width=5*inch, height=5*inch))
    story.append(PageBreak())
    
    # 4. Area Chart
    area_data = {
        "Sales": [50, 60, 55, 70, 65, 80],
        "Revenue": [40, 50, 45, 60, 55, 70]
    }
    area_file = tempfile.NamedTemporaryFile(suffix='.png', delete=False)
    create_area_chart(months, area_data, "Sales vs Revenue", area_file.name)
    chart_files.append(area_file.name)
    story.append(Paragraph("Sales and Revenue Trends", styles["Heading2"]))
    story.append(PlatypusImage(area_file.name, width=6*inch, height=3.5*inch))
    story.append(Spacer(1, 0.3*inch))
    
    # 5. Custom Diagram
    flowchart = create_flowchart()
    story.append(Paragraph("Process Flowchart", styles["Heading2"]))
    story.append(Spacer(1, 0.2*inch))
    story.append(flowchart)
    story.append(PageBreak())
    
    # 6. Heatmap
    import numpy as np
    heatmap_data = np.random.rand(5, 5) * 100
    heatmap_file = tempfile.NamedTemporaryFile(suffix='.png', delete=False)
    create_heatmap(heatmap_data, "Performance Matrix", heatmap_file.name)
    chart_files.append(heatmap_file.name)
    story.append(Paragraph("Performance Heatmap", styles["Heading2"]))
    story.append(PlatypusImage(heatmap_file.name, width=6*inch, height=3.5*inch))
    
    doc.build(story)
    
    # Cleanup temporary files
    for f in chart_files:
        try:
            os.unlink(f)
        except:
            pass
    
    return chart_files

# Generate report
# generate_charts_report("charts_report.pdf")
```

---

## Best Practices

1. **Always use non-interactive backend**: `matplotlib.use('Agg')` before importing pyplot
2. **Set DPI for quality**: Use at least 150 DPI for print-quality charts
3. **Clean up temp files**: Always remove temporary chart images after building PDF
4. **Use consistent colors**: Define a color palette for professional look
5. **Add grid and labels**: Improve readability with grid lines and value labels
6. **Tight layout**: Use `bbox_inches='tight'` to avoid white space
7. **White background**: Set `facecolor='white'` for transparent backgrounds
8. **Optimize image sizes**: Use appropriate chart dimensions for PDF layout
9. **Set aspect ratio**: Preserve aspect ratio when inserting images
10. **Use vector formats**: For simple diagrams, consider SVG instead of PNG

## See Also

- [reportlab-guide.md](reportlab-guide.md) - PDF creation and document structure
- [SCENARIOS.md](SCENARIOS.md) - Complete report automation examples
- [REFERENCE.md](REFERENCE.md) - Colors and styling reference
