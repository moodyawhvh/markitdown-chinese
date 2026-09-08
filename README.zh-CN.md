<div align="center">

# markitdown 中文文档

[![原项目](https://img.shields.io/badge/原项目-microsoft--markitdown-blue?style=flat-square&logo=github)](https://github.com/microsoft/markitdown)
[![PyPI](https://img.shields.io/badge/PyPI-markitdown-blue?style=flat-square&logo=pypi)](https://pypi.org/project/markitdown/)
[![微信联系](https://img.shields.io/badge/微信-uaycar-brightgreen?style=flat-square&logo=wechat)](#)

</div>

---

> 本文档是 [microsoft/markitdown](https://github.com/microsoft/markitdown) 官方 README 的中文翻译版本。
> 完整源代码请访问原项目:https://github.com/microsoft/markitdown

**代部署 / 定制服务 / 技术咨询 请添加微信:uaycar**

---

## ⚠️ 重要安全提示

MarkItDown 以当前进程的权限执行 I/O 操作。与 `open()` 或 `requests.get()` 类似,它会访问进程本身可访问的任何资源。在不可信环境中请务必对输入做安全过滤,并尽量调用范围最窄的转换函数(例如 `convert_stream()` 或 `convert_local()`)。

## 📖 项目简介

MarkItDown 是一个轻量级 Python 工具,用于将各种文件转换为 Markdown,主要服务于大语言模型(LLM)及相关文本分析管道。同类工具中它最接近 [textract](https://github.com/deanmalmgren/textract),但更注重以 Markdown 形式保留文档的重要结构与内容(包括标题、列表、表格、链接等)。虽然其输出通常也相当整洁、对人类可读,但它设计上是给文本分析工具消费的,未必是面向人类的高保真文档转换的最佳选择。

当前支持从以下格式转换:

- PDF
- PowerPoint
- Word
- Excel
- 图片(EXIF 元数据与 OCR)
- 音频(EXIF 元数据与语音转录)
- HTML
- 文本类格式(CSV、JSON、XML)
- ZIP 文件(遍历内部内容)
- YouTube 链接
- EPUB 电子书
- ……以及更多!

## 🤔 为什么选择 Markdown?

Markdown 极其接近纯文本,标记和格式极少,却仍能表达文档的重要结构。主流大模型(如 OpenAI 的 GPT-4o)原生"会说"Markdown,甚至经常在回复中主动使用 Markdown。这表明它们在大量 Markdown 格式文本上训练过,对它的理解非常好。附带的好处是,Markdown 的书写约定也非常节省 token。

## 📋 环境要求

MarkItDown 需要 Python 3.10 或更高版本。建议使用虚拟环境以避免依赖冲突。

标准 Python 安装方式创建并激活虚拟环境:

```bash
python -m venv .venv
source .venv/bin/activate
```

如果使用 `uv`:

```bash
uv venv --python=3.12 .venv
source .venv/bin/activate
# 注意:在此虚拟环境中安装包请使用 'uv pip install' 而不是直接 'pip install'
```

如果使用 Anaconda:

```bash
conda create -n markitdown python=3.12
conda activate markitdown
```

## 📦 安装

使用 pip 安装:`pip install 'markitdown[all]'`。也可以从源码安装:

```bash
git clone git@github.com:microsoft/markitdown.git
cd markitdown
pip install -e 'packages/markitdown[all]'
```

## 🚀 使用方法

### 命令行

```bash
markitdown path-to-file.pdf > document.md
```

或使用 `-o` 指定输出文件:

```bash
markitdown path-to-file.pdf -o document.md
```

也可以通过管道输入内容:

```bash
cat path-to-file.pdf | markitdown
```

### 可选依赖

MarkItDown 为不同文件格式提供可选依赖。前文用 `[all]` 一次性安装了全部可选依赖,你也可以按需单独安装以获得更精细的控制。例如:

```bash
pip install 'markitdown[pdf, docx, pptx]'
```

将只安装 PDF、DOCX 和 PPTX 文件所需的依赖。

当前可用的可选依赖包括:

- `[all]` — 安装全部可选依赖
- `[pptx]` — PowerPoint 文件支持
- `[docx]` — Word 文件支持
- `[xlsx]` — Excel 文件支持
- `[xls]` — 旧版 Excel 文件支持
- `[pdf]` — PDF 文件支持
- `[outlook]` — Outlook 邮件消息支持
- `[az-doc-intel]` — Azure Document Intelligence 支持
- `[az-content-understanding]` — Azure Content Understanding 支持
- `[audio-transcription]` — wav 与 mp3 音频转录支持
- `[youtube-transcription]` — YouTube 视频转录获取支持

### 插件

MarkItDown 支持第三方插件,插件默认禁用。列出已安装插件:

```bash
markitdown --list-plugins
```

启用插件:

```bash
markitdown --use-plugins path-to-file.pdf
```

在 GitHub 搜索话题标签 `#markitdown-plugin` 可以发现可用插件;插件开发可参考 `packages/markitdown-sample-plugin`。

#### markitdown-ocr 插件

`markitdown-ocr` 插件为 PDF、DOCX、PPTX、XLSX 转换器增加 OCR 能力,使用 LLM 视觉模型从内嵌图片中提取文字——与 MarkItDown 图片描述功能使用的 `llm_client` / `llm_model` 模式相同,无需引入新的机器学习库或二进制依赖。

```bash
pip install markitdown-ocr
pip install openai  # 或任意 OpenAI 兼容客户端
```

传入与图片描述相同的 `llm_client` 和 `llm_model`:

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)
result = md.convert("document_with_images.pdf")
print(result.markdown)
```

若未提供 `llm_client`,插件仍会加载,但会静默跳过 OCR,改用内置标准转换器。

### Azure Content Understanding

[Azure Content Understanding](https://learn.microsoft.com/azure/ai-services/content-understanding/) 提供更高质量的转换:结构化字段抽取(YAML front matter)、多模态支持(文档、图片、音频、视频)以及可配置的分析器。

安装:`pip install 'markitdown[az-content-understanding]'`

适合使用 Content Understanding 的场景:

- **音视频文件** — 视频只能靠它,音频的云端高质量方案也是它;内置转换器不支持视频,音频转录也较基础。
- **结构化字段抽取** — 预置或自定义分析器可抽取领域字段(发票金额、收据日期、合同条款),并以 YAML front matter 序列化输出。
- **更高质量的文档抽取** — 云端版面分析与 OCR,适合扫描版 PDF、复杂表格和多页文档。
- **单一 API 覆盖所有模态** — 一个 `cu_endpoint` 处理文档、图片、音频、视频,自动路由分析器。

命令行用法:

```bash
markitdown path-to-file.pdf --use-cu --cu-endpoint "<content_understanding_endpoint>"
```

也可通过环境变量一次性配置端点:

```bash
export MARKITDOWN_CU_ENDPOINT="<content_understanding_endpoint>"
markitdown path-to-file.pdf --use-cu
```

Python API:

```python
from markitdown import MarkItDown

# 零配置 — 按文件类型自动选择分析器
md = MarkItDown(cu_endpoint="<content_understanding_endpoint>")
result = md.convert("report.pdf")   # 文档 → prebuilt-documentSearch
result = md.convert("meeting.mp4")  # 视频 → prebuilt-videoSearch
result = md.convert("call.wav")     # 音频 → prebuilt-audioSearch
print(result.markdown)
```

注意:CU 路由格式的每次 `convert()` 调用都是计费的 Azure API 调用,可用 `cu_file_types` 限制哪些格式走 CU。

### Azure Document Intelligence

使用微软 Document Intelligence 进行转换:

```bash
markitdown path-to-file.pdf -o document.md -d -e "<document_intelligence_endpoint>"
```

端点同样可通过环境变量一次性设置:

```bash
export MARKITDOWN_DOCINTEL_ENDPOINT="<document_intelligence_endpoint>"
markitdown path-to-file.pdf -o document.md -d
```

### Python API

基础用法:

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=False) # 设为 True 启用插件
result = md.convert("test.xlsx")
print(result.markdown)
```

使用 Document Intelligence 转换:

```python
from markitdown import MarkItDown

md = MarkItDown(docintel_endpoint="<document_intelligence_endpoint>")
result = md.convert("test.pdf")
print(result.markdown)
```

使用大模型生成图片描述(目前仅支持 pptx 和图片文件),需提供 `llm_client` 和 `llm_model`:

```python
from markitdown import MarkItDown
from openai import OpenAI

client = OpenAI()
md = MarkItDown(llm_client=client, llm_model="gpt-4o", llm_prompt="可选的自定义提示词")
result = md.convert("example.jpg")
print(result.markdown)
```

### Docker

```sh
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < ~/your-file.pdf > output.md
```

## 🤝 参与贡献

本项目欢迎贡献与建议,多数贡献需要签署贡献者许可协议(CLA)。本项目遵循 [Microsoft 开源行为准则](https://opensource.microsoft.com/codeofconduct/)。

贡献范围说明:本仓库定位于提供可集成到其他系统的 Python 库,而非构建在其上的终端用户应用。现有转换器的保真度改进、Bug 修复、性能与安全修复、`markitdown` 命令行界面、`markitdown-mcp` 包、测试与文档等均在范围内;Web 服务器、REST/HTTP API、托管转换服务、网页前端、桌面与移动应用等不在接受范围内——如有兴趣,请以依赖 PyPI 上 `markitdown` 的独立项目形式维护。

新格式支持可通过第三方插件独立发布安装,参考 `packages/markitdown-sample-plugin` 入门,并给你的仓库打上 `#markitdown-plugin` 标签。

运行测试与检查:

```sh
cd packages/markitdown
pip install hatch
hatch shell
hatch test
```

提交 PR 前运行 pre-commit 检查:`pre-commit run --all-files`

## 🔒 安全注意事项

MarkItDown 以当前进程权限执行 I/O 操作,会访问进程自身可访问的资源。

**净化你的输入:** 不要将不可信输入直接传给 MarkItDown。若输入的任何部分可能被不可信用户或系统控制(例如托管或服务端应用),调用前必须进行验证和限制,包括限制文件路径、限制 URI 协议与网络目标、阻止对私有地址、回环地址、链路本地地址及元数据服务的访问。

**只调用所需的转换方法:** 优先选择范围最窄的转换 API。`convert()` 方法设计上较为宽松,可处理本地文件、远程 URI 和字节流;若只需读取本地文件,请改用 `convert_local()`;若需自行控制 URI 获取,可自己调用 `requests.get()` 后将响应对象传给 `convert_response()`;需要最大控制权时,打开输入流并调用 `convert_stream()`。

## 📄 商标声明

本项目可能包含相关项目、产品或服务的商标或徽标。微软商标与徽标的授权使用须遵循 [Microsoft 商标与品牌指南](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general),在本项目修改版中的使用不得引起混淆或暗示微软的赞助。第三方商标与徽标的使用受相应第三方政策约束。

---

> 本文档为 [microsoft/markitdown](https://github.com/microsoft/markitdown) 官方 README 的中文翻译,所有代码与内容版权归原项目作者所有,遵循其原始许可证(MIT)。
> 翻译仅供学习参考,如有出入请以原项目英文文档为准。

**代部署 / 定制服务 / 技术咨询 请添加微信:uaycar**

**如果觉得有用,请给原项目点个 Star!** ⭐
