# LaTeX → Word 转换（pandoc + ctex）

将 ctex 中文数学论文转为 Word 文档的方法与陷阱。

## 核心问题

pandoc 不理解 ctex 专有命令（`\songti`, `\heiti`, `\kaishu`, `\zihao`, `\fontfamily`, `\selectfont`），也不解析 `\input` 子文件。直接转换会导致：
- 数学公式丢失
- ctex 命令被跳过
- `\input` 内容缺失

## 正确流程

### 1. 合并所有子文件

```bash
cat zh-intro.tex zh-prelim.tex zh-sec2.tex ... > body-combined.tex
```

或 Python 递归解析 `\input` 命令。

### 2. 手动嵌入摘要/关键词

`main.tex` 中的摘要和关键词通常在正文里（非子文件），需提取并拼入合并文件最前面。

### 3. 创建最简 preamble

```latex
\documentclass{ctexart}
\usepackage{amsmath,amssymb,amsthm}
% 最少的定理环境定义
\begin{document}
...body...
\end{document}
```

**不要**包含 `newtxtext`, `newtxmath`, `microtype`, `geometry`, `\ctexset` 等包。pandoc 不需要它们进行公式转换。

### 4. 清理 ctex 专有命令

如果已嵌入在 body 中，需用正则删除：
```python
body = re.sub(r'\\(?:songti|heiti|kaishu|fangsong)\b', '', body)
body = re.sub(r'\\zihao\{[^}]*\}', '', body)
body = re.sub(r'\\fontfamily\{[^}]*\}', '', body)
body = re.sub(r'\\selectfont', '', body)
```

### 5. 转换

```bash
pandoc paper-flat.tex -o paper.docx --from=latex --to=docx
```

## 已知限制

| 问题 | 原因 | 解决 |
|------|------|------|
| 公式不显示 | ctex 命令干扰 | 清理 ctex 命令后重转 |
| 摘要/关键词缺失 | 在 main.tex 中非子文件 | 手动提取拼接 |
| 节编号错位 | pandoc 不完全支持 `[section]` 计数器 | Word 中手动修复 |
| 证明显示 "Proof." | amsthm 默认标签 | Word 中手动改 |
| 数学显示为空白 | 公式在 OMML 层，`.text` 提取看不到 | Word 打开即可见 |
| `\cite` 显示 `[key]` | 无 bib 文件 | Word 中手动替换 |

## 数学保留验证

```python
from docx import Document
from lxml import etree
ns = {'m': 'http://schemas.openxmlformats.org/officeDocument/2006/math'}
count = len(doc.element.findall('.//m:oMath', ns))
```

OMML 公式数 > 0 即表示数学成功保留（Word 可渲染）。
