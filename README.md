<div align="center">

# markitdown 中文翻译版

**[中文版] markitdown — 微软开源的轻量级文件转 Markdown 工具**

[![原项目](https://img.shields.io/badge/原项目-microsoft--markitdown-blue?style=flat-square&logo=github)](https://github.com/microsoft/markitdown)
[![中文文档](https://img.shields.io/badge/中文文档-README.zh--CN.md-orange?style=flat-square)](README.zh-CN.md)
[![GitHub Stars](https://img.shields.io/github/stars/microsoft/markitdown?style=flat-square&label=原项目Stars)](https://github.com/microsoft/markitdown/stargazers)
[![微信联系](https://img.shields.io/badge/微信-uaycar-brightgreen?style=flat-square&logo=wechat)](#)

</div>

---

> 这是 [microsoft/markitdown](https://github.com/microsoft/markitdown) 的中文翻译版本。
> 完整源代码请访问原项目:https://github.com/microsoft/markitdown

**代部署 / 定制服务 / 技术咨询 请添加微信:uaycar**

---

## 📖 项目简介

MarkItDown 是微软开源的一个轻量级 Python 工具,用于将各类文件和 Office 文档转换为 Markdown 格式,主要面向大语言模型(LLM)和文本分析管道场景。它在转换时注重保留文档的关键结构(标题、列表、表格、链接等),输出对文本分析工具非常友好。支持的输入格式包括 PDF、Word、Excel、PowerPoint、图片(OCR)、音频(转录)、HTML、CSV/JSON/XML、ZIP、YouTube 链接、EPUB 等。

## ✨ 主要特性

- 📄 多格式支持:PDF、Word、Excel、PowerPoint 一站式转换为 Markdown
- 🖼️ 图片处理:自动提取 EXIF 元数据,支持 OCR 文字识别
- 🎧 音频转录:支持 wav/mp3 音频的语音转文字
- 🌐 网页与在线内容:HTML 页面、YouTube 视频转录、EPUB 电子书
- 🤗 为 LLM 而生:Markdown 接近纯文本、省 token,主流大模型对其理解极佳
- 🔌 插件系统:支持第三方插件扩展新格式,可按需启用
- ☁️ Azure 集成:可选接入 Azure Document Intelligence 与 Azure Content Understanding 获得更高质量的云端转换
- 🧠 LLM 图片描述:接入 OpenAI 等客户端,用视觉模型为图片生成描述
- 🐳 提供 Docker 镜像,一条命令完成容器化转换
- 💻 同时提供命令行工具和 Python API 两种使用方式

## 📁 文件说明

| 文件 | 说明 |
|:-----|:-----|
| README.md | 本文件(中文简介) |
| README.zh-CN.md | 详细中文文档(完整汉化) |

## 🚀 快速开始

**1. 环境要求**:Python 3.10+,建议使用虚拟环境:

```bash
python -m venv .venv
source .venv/bin/activate
```

**2. 安装**(完整可选依赖):

```bash
pip install 'markitdown[all]'
```

**3. 命令行转换文件:**

```bash
markitdown path-to-file.pdf > document.md
# 或指定输出文件
markitdown path-to-file.pdf -o document.md
# 或通过管道输入
cat path-to-file.pdf | markitdown
```

**4. Python API 调用:**

```python
from markitdown import MarkItDown

md = MarkItDown()
result = md.convert("test.xlsx")
print(result.markdown)
```

**5. 按需安装部分格式依赖**(可选):

```bash
pip install 'markitdown[pdf, docx, pptx]'
```

**6. Docker 方式运行**(可选):

```sh
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < ~/your-file.pdf > output.md
```

**⚠️ 安全提示**:MarkItDown 以当前进程权限执行 I/O 操作,在不可信环境中请务必对输入做安全校验,并优先使用最窄范围的转换接口(如 `convert_local()`、`convert_stream()`)。

完整源代码与最新版本请访问原项目:https://github.com/microsoft/markitdown

## 📞 联系方式

**代部署 / 定制服务 / 技术咨询 请添加微信:uaycar**

---

本项目为 [microsoft/markitdown](https://github.com/microsoft/markitdown) 的中文翻译版本,所有代码版权归原项目作者所有,遵循其原始许可证。

**如果觉得有用,请给原项目点个 Star!** ⭐
