> 🌐 本文档由 [microsoft/markitdown](https://github.com/microsoft/markitdown) 翻译,英文原版见原项目。

# MarkItDown OCR 插件

面向 MarkItDown 的 LLM 视觉插件,可从 PDF、DOCX、PPTX、XLSX 文件内嵌的图片中提取文字。

它复用 MarkItDown 原生支持的 `llm_client` / `llm_model` 模式(与图片描述功能相同)——无需引入新的机器学习库或二进制依赖。

## 功能特性

- **增强 PDF 转换器**:提取 PDF 内图片中的文字,扫描件还能整页 OCR 兜底
- **增强 DOCX 转换器**:对 Word 文档中的图片做 OCR
- **增强 PPTX 转换器**:对 PowerPoint 演示文稿中的图片做 OCR
- **增强 XLSX 转换器**:对 Excel 表格中的图片做 OCR
- **上下文保持**:插入提取文字时保留文档原有结构与阅读顺序

## 安装

```bash
pip install markitdown-ocr
```

插件使用你已有的任意 OpenAI 兼容客户端。还没有的话先装一个:

```bash
pip install openai
```

## 用法

### 命令行

```bash
markitdown document.pdf --use-plugins --llm-client openai --llm-model gpt-4o
```

### Python API

像图片描述功能一样,把 `llm_client` 和 `llm_model` 传给 `MarkItDown()` 即可:

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)

result = md.convert("document_with_images.pdf")
print(result.text_content)
```

如果不提供 `llm_client`,插件照样加载,但会静默跳过 OCR——退回标准内置转换器。

### 自定义提示词

针对特殊文档,可以覆盖默认的提取提示词:

```python
md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
    llm_prompt="Extract all text from this image, preserving table structure.",
)
```

### 任意 OpenAI 兼容客户端

任何遵循 OpenAI API 的客户端都可以使用:

```python
from openai import AzureOpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=AzureOpenAI(
        api_key="...",
        azure_endpoint="https://your-resource.openai.azure.com/",
        api_version="2024-02-01",
    ),
    llm_model="gpt-4o",
)
```

## 工作原理

当调用 `MarkItDown(enable_plugins=True, llm_client=..., llm_model=...)` 时:

1. MarkItDown 通过 `markitdown.plugin` 入口点组发现插件
2. 调用 `register_converters()`,并转发包括 `llm_client` 和 `llm_model` 在内的全部 kwargs
3. 插件用这些 kwargs 创建 `LLMVisionOCRService`
4. 四个 OCR 增强转换器以 **-1.0 优先级**注册——排在优先级 0.0 的内置转换器之前

转换文件时:

1. OCR 转换器接手该文件
2. 从文档中提取内嵌图片
3. 每张图片连同提取提示词一起发给 LLM
4. 返回的文字按原位内联插入,保持文档结构
5. 若 LLM 调用失败,转换继续进行,只是缺少该图片的文字

## 支持的文件格式

### PDF

- 内嵌图片按位置提取(通过 `page.images` / 页面 XObject),OCR 文字按纵向阅读顺序与上下文文本交错插入。
- **扫描版 PDF**(页面无可提取文本)会被自动识别:每页按 300 DPI 渲染成整页图片发给 LLM。
- pdfplumber/pdfminer 打不开的**损坏 PDF**(如 EOF 截断)会改用 PyMuPDF 页面渲染重试,内容仍可恢复。

### DOCX

- 图片通过文档部件关系(`doc.part.rels`)提取。
- OCR 在 DOCX→HTML→Markdown 管道执行之前完成:占位标记先注入 HTML,避免 markdown 转换器转义 OCR 标记,转换结束后再把占位标记替换为格式化的 `*[Image OCR]...[End OCR]*` 块。
- OCR 块周围的文档流(标题、段落、表格)完整保留。

### PPTX

- 图片形状、带图片的占位符形状、以及组合内部的图片都支持。
- 每张幻灯片内的形状按从上到下、从左到右的阅读顺序处理。
- 如果配置了 `llm_client`,会先向 LLM 请求图片描述;没有返回描述时才回退到 OCR。

### XLSX

- 各工作表内嵌的图片(`sheet._images`)按表逐一提取。
- 单元格位置由图片锚点坐标换算(列/行 → Excel 字母标记)。
- 图片列在表格数据之后的 `### Images in this sheet:` 小节中——不会插入表格行内。

### 输出格式

每个 OCR 提取块都包裹为:

```text
*[Image OCR]
<extracted text>
[End OCR]*
```

## 故障排查

### 输出中缺少 OCR 文字

最可能的原因是没传 `llm_client` 或 `llm_model`。请检查:

```python
from openai import OpenAI
from markitdown import MarkItDown

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),   # 必填
    llm_model="gpt-4o",    # 必填
)
```

### 插件未加载

确认插件已安装且能被发现:

```bash
markitdown --list-plugins   # 应显示: ocr
```

### API 报错

插件会把 LLM API 错误作为警告抛出并继续转换。请检查 API 密钥、配额,以及所选模型是否支持视觉输入。

## 开发

### 运行测试

```bash
cd packages/markitdown-ocr
pytest tests/ -v
```

### 从源码构建

```bash
git clone https://github.com/microsoft/markitdown.git
cd markitdown/packages/markitdown-ocr
pip install -e .
```

## 参与贡献

欢迎贡献!贡献指南见 [MarkItDown 仓库](https://github.com/microsoft/markitdown)。

## 许可证

MIT —— 见 [LICENSE](LICENSE)。

## 更新日志

### 0.1.0(首发版本)

- 面向 PDF、DOCX、PPTX、XLSX 的 LLM 视觉 OCR
- 扫描版 PDF 的整页 OCR 兜底
- 感知上下文的内联文字插入
- 基于优先级的转换器替换(无需改动任何代码)
