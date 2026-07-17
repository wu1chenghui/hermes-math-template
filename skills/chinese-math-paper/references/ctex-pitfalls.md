# ctex 排版陷阱

> 日期：2026-07-12

## 1. 楷体加粗不生效

**问题**：`\kaishu\bfseries` 在 ctex 中不产生可见的粗体效果。

**原因**：ctex 使用的楷体字体（KaiTi/楷体）通常没有原生粗体变体。
Word 通过合成粗体（synthetic bold）渲染，但 LaTeX 默认不会。

**修复**：在 preamble 中添加 AutoFakeBold：

```latex
\setCJKfamilyfont{kaishu}{KaiTi}[AutoFakeBold={2.5}]
```

参数 `2.5` 控制伪粗体的粗细程度，可调整。

**验证**：编译后检查 PDF，楷体文本应有明显加粗。如果不可见，增大 AutoFakeBold 值。

## 2. 节编号与标题间的空格

**问题**：默认 ctex 节标题显示为 "1  预备知识"（编号后有空格），
但 Word 模板常要求 "1预备知识"（紧贴）。

**修复**：

```latex
\ctexset{
  section = {
    aftername = \hspace{0pt},  % 去掉默认间距
    name = {},                  % 去掉 "第" "节"
    number = \arabic{section},  % 阿拉伯数字
  }
}
```

## 3. 页眉自动显示节标题

**问题**：ctexart 默认在每页顶部显示当前节标题作为页眉。
Word 模板通常无页眉。

**修复**：

```latex
\begin{document}
\pagestyle{empty}
```

## 4. \DeclareMathOperator 与 ctex 的交互

ctex 不改变 `\DeclareMathOperator` 的行为。在正文中使用 `\im`, `\id`, `\ad` 等自定义运算符与标准 LaTeX 一致。

## 5. newtxtext/newtxmath 与 ctex 中文字体共存

`newtxtext` 将西文字体改为 Times，不影响 ctex 管理的中文字体（宋体/黑体/楷体）。
但 `\zihao` 命令同时对中西文生效，字号一致。
