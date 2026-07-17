# Word 格式提取方法论

从 `.docx` 参考模板中系统提取格式规范的方法。

## 工具链

- `python-docx`：段落/字体/缩进/对齐读取
- `lxml` + OOXML 命名空间：直接读取 XML 属性（行距、字距、孤行控制等）

## 提取流程

### 1. 页面设置

```python
for section in doc.sections:
    w = section.page_width / 360000   # cm
    h = section.page_height / 360000
    top = section.top_margin / 360000
```

### 2. 字体/字号/加粗

```python
for para in doc.paragraphs:
    for r in para.runs:
        font_name = r.font.name      # None = 继承样式
        font_size = r.font.size      # EMU, /12700 = pt
        is_bold = r.font.bold
```

**关键**：`font.name = None` 表示继承段落/样式默认字体。`font.size = None` 同理。需同时检查 Normal 样式基准。

### 3. 段落格式

```python
pf = para.paragraph_format
first_indent = pf.first_line_indent / 12700  # pt
left_indent = pf.left_indent / 12700
space_before = pf.space_before / 12700
space_after = pf.space_after / 12700
alignment = para.alignment  # 0=LEFT, 1=CENTER, 2=RIGHT, 3=JUSTIFY
```

### 4. 行距（XML 层）

python-docx 的 `line_spacing` 对继承值返回 `None`。需读 XML：

```python
from docx.oxml.ns import qn
pPr = para._element.find(qn('w:pPr'))
spacing = pPr.find(qn('w:spacing'))
line = spacing.get(qn('w:line'))        # 行距值
lineRule = spacing.get(qn('w:lineRule'))  # 规则
```

### 5. 缩进值精度

Word 中 21.0pt 和 21.1pt 本质相同（0.1pt 是浮点舍入误差），都对应 2 个中文字符的缩进量。

### 6. Normal 样式基准

```python
normal = doc.styles['Normal']
n_pPr = normal.element.find(qn('w:pPr'))
# 检查 widowControl, jc (对齐), spacing (行距)
n_rPr = normal.element.find(qn('w:rPr'))
# 检查 sz (字号 half-points), rFonts, kern
```

字号 half-points 换算：`sz=21` = 10.5pt (五号)，`sz=28` = 14pt (四号)。

## 输出

生成 `FORMAT-SPEC.md`，包含：
- Normal 样式基准
- 页面设置表格
- 每个区域的字体/字号/加粗/对齐/缩进/段距表格
- 字号对照表（pt ↔ ctex 字号命令）
- 对齐方式汇总
