# CUHKthesis 格式调整用户指南

> 版本：2026-03-18 | 编译器：XeLaTeX | 主文件：`thesis.tex`

---

## 一、项目文件速查

```
CUHKthesis/
├── thesis.tex              ← 入口文件：元数据 + 章节顺序
├── preamble/
│   ├── settings.tex        ← 宏包、字体、间距参数（改样式改这里）
│   └── cmds.tex            ← 封面/Abstract/摘要排版命令
├── pages/
│   ├── abstract.tex        ← 英文摘要正文
│   ├── abstract-zh.tex     ← 中文摘要正文
│   └── acknowledgement.tex ← 致谢正文
├── chapters/chpX-*.tex     ← 各章节
├── appendices/appx-*.tex   ← 附录
└── refs.bib                ← 参考文献库
```

**记住这张表，想改什么找对文件：**

| 想改的内容 | 对应文件 |
|-----------|---------|
| 作者/标题/日期/学位 | `thesis.tex` 第 18–29 行 |
| 字体/行距/间距/宏包 | `preamble/settings.tex` |
| Abstract 排版布局 | `preamble/cmds.tex` 第 56–72 行 |
| 摘要正文文字 | `pages/abstract.tex` / `abstract-zh.tex` |
| 章节顺序 | `thesis.tex` 的 `\include` 列表 |
| 参考文献 | `refs.bib` + `settings.tex` 里的 `\bibliographystyle` |

---

## 二、常用调整操作

### 修改封面/Abstract 头部信息

编辑 `thesis.tex`：

```latex
\title{The miRNA Regulatory Nexus: ...}   % 封面英文标题
\titlezh{miRNA调控枢纽：...}               % 摘要页中文标题
\author{XU, Jiatong}
\degree{Doctor of Philosophy}
\institution{The Chinese University of Hong Kong, Shenzhen}
\date{January 2026}
```

> ⚠️ Abstract 页的英文大写标题单独硬编码在 `cmds.tex` 第 63 行，需要手动同步修改。

### 修改摘要正文

直接编辑 `pages/abstract.tex` 或 `pages/abstract-zh.tex`，段落间空一行，不要用 `\\` 强制换行。

### 调整章节标题上方间距

方法 A（推荐）：在 `cmds.tex` 每个 `\chapter*{...}` 后加一行：

```latex
\chapter*{Abstract}
\vspace*{-15pt}   % 负值=收缩，正值=加大，自行调整数字
```

方法 B：在 `settings.tex` 用 titlesec（注意只对普通 `\chapter` 有效，不影响 `\chapter*`）：

```latex
\titlespacing*{\chapter}{0pt}{10pt}{20pt}
```

### 调整正文行距

```latex
% 在 settings.tex 设置全局行距
\setstretch{1.5}

% 或在具体段落局部控制
\begin{spacing}{1.2}
  ...
\end{spacing}
```

### 修改参考文献样式

```latex
% settings.tex 里
\bibliographystyle{apalike}   % APA 风格
\bibliographystyle{ieeetr}    % IEEE 风格
\bibliographystyle{plain}     % 字母排序
```

---

## 三、编译说明

### VSCode + LaTeX Workshop（推荐本地方案）

1. 安装插件：**LaTeX Workshop**
2. 打开 `thesis.tex`
3. `Ctrl+Alt+B` 编译，`Ctrl+Alt+V` 预览 PDF（支持同步跳转）
4. Recipe 设置为 `xelatex → bibtex → xelatex × 2`

`.vscode/settings.json` 参考配置：

```json
{
  "latex-workshop.latex.tools": [
    {
      "name": "xelatex",
      "command": "xelatex",
      "args": ["-interaction=nonstopmode", "-synctex=1", "%DOC%"]
    },
    {
      "name": "bibtex",
      "command": "bibtex",
      "args": ["%DOCFILE%"]
    }
  ],
  "latex-workshop.latex.recipes": [
    {
      "name": "xelatex x2",
      "tools": ["xelatex", "xelatex"]
    },
    {
      "name": "xelatex + bibtex + xelatex x2",
      "tools": ["xelatex", "bibtex", "xelatex", "xelatex"]
    }
  ],
  "latex-workshop.latex.recipe.default": "xelatex x2"
}
```

### 命令行编译

```bash
cd CUHKthesis
xelatex -interaction=nonstopmode thesis.tex
bibtex thesis                             # 有新引用时需要
xelatex -interaction=nonstopmode thesis.tex
xelatex -interaction=nonstopmode thesis.tex  # 第三遍确保交叉引用正确
```

### Overleaf（云端备选）

- 将 `CUHKthesis/` 文件夹打包上传
- Compiler 必须选 **XeLaTeX**（不能用 pdfLaTeX）
- 已装 Overleaf Workshop 插件可在 VSCode 直接同步

> ⚠️ **必须用 XeLaTeX**：中文 ctex 和 fontspec 字体设置都依赖 XeLaTeX，pdfLaTeX 会直接报错。

---

## 四、常见问题排查

| 现象 | 原因 | 修复 |
|------|------|------|
| 中文乱码 | 文件编码不是 UTF-8 | VSCode 右下角 → 点编码 → 转为 UTF-8 保存 |
| Abstract 英文标题超出右边界 | `\underline` 不换行 | 改用 `\CJKunderline`（需 `xeCJKfntef` 宏包） |
| `\CJKunderline` 下划线不显示 | ctex+fontspec 配置冲突 | 短标题改用 `\underline{...}` |
| 续行没有缩进 | 默认左边距为 0 | 在命令前加 `{\setlength{\leftskip}{2em}\CJKunderline{...}\par}` |
| 目录出现多余条目 | `\addcontentsline` 位置错 | 在前面加 `\phantomsection` |
| References 在 Appendices 后面 | `thesis.tex` 中顺序错误 | `\bibliography{refs}` 必须在 `\begin{appendices}` 之前 |
| TOC 页面不更新 | LaTeX 需两次编译才更新目录 | 再编译一次 |
| 编译找不到字体 | Times New Roman 未安装 | 系统安装该字体，或改为 `TeX Gyre Termes` |
| `\@titlezh undefined` | `thesis.tex` 缺少 `\titlezh{}` | 在元数据区加该行 |

---

## 五、与 AI 协作高效方式

### 提供信息的优先级

1. **截图 + 描述现象**（最高效）：直接截 PDF 截图，说"这里XXX不对"
2. **提供 Word 文件**：摘要/章节内容用 `.docx` 提供，避免 PDF 复制带来的乱码和断行
3. **贴编译报错**：把 terminal 报错或 `thesis.log` 末尾 30 行直接贴出来

### 描述问题的方式

```
✅ "Abstract 顶部空白太大，能缩小一半吗？" + 截图
✅ "英文标题超出页面右边界" + 截图
✅ 附上 Word 文件让 AI 提取正文内容
❌ "帮我把第三个参数改成 10pt"（除非你确定那个参数）
```

### 节省时间的习惯

- 每次改动后立即编译验证，不要攒多个改动
- 用 LaTeX Workshop 的"保存时自动编译"功能（`latex-workshop.latex.autoBuild.run: "onSave"`）
- 改动前告诉 AI"只改 X，不动 Y"，减少误改

---

## 六、已安装宏包（`preamble/settings.tex`）

| 宏包 | 作用 |
|------|------|
| `fontspec` | 设置英文字体（Times New Roman） |
| `ctex` | 中文支持（XeLaTeX 专用） |
| `xeCJKfntef` | 中文下划线 `\CJKunderline` |
| `ulem` | 英文可换行下划线 `\uline` |
| `tocloft` | 目录样式定制 |
| `titlesec` | 章节标题间距 `\titlespacing` |
| `setspace` | 行距控制 `\begin{spacing}{}` |
| `caption` | 图表 caption 样式（`labelfont=bf`） |
| `appendix` | `appendices` 环境 |
| `natbib` | 参考文献引用格式 |
| `geometry` | 页面边距设置 |
