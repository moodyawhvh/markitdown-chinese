> 🌐 本文档由 [microsoft/markitdown](https://github.com/microsoft/markitdown) 翻译,英文原版见原项目。

# MarkItDown

> [!TIP]
> MarkItDown 是一个 Python 包和命令行工具,用于把各类文件转换为 Markdown(例如用于索引、文本分析等场景)。
>
> 更多信息和完整文档请参见 GitHub 上的项目 [README.md](https://github.com/microsoft/markitdown)。

> [!IMPORTANT]
> MarkItDown 以当前进程的权限执行 I/O 操作。与 `open()` 或 `requests.get()` 类似,它会访问进程本身可访问的任何资源。在不可信环境中请务必对输入做安全过滤,并尽量调用范围最窄的 `convert_*` 函数(例如 `convert_stream()` 或 `convert_local()`)。更多信息参见文档中的[安全注意事项](https://github.com/microsoft/markitdown#security-considerations)一节。

## 安装

从 PyPI 安装:

```bash
pip install 'markitdown[all]'
```

从源码安装:

```bash
git clone git@github.com:microsoft/markitdown.git
cd markitdown
pip install -e 'packages/markitdown[all]'
```

## 用法

### 命令行

```bash
markitdown path-to-file.pdf > document.md
```

### Python API

```python
from markitdown import MarkItDown

md = MarkItDown()
result = md.convert("test.xlsx")
print(result.markdown)
```

### 更多信息

更多信息和完整文档请参见 GitHub 上的项目 [README.md](https://github.com/microsoft/markitdown)。

## 商标声明

本项目可能包含相关项目、产品或服务的商标或徽标。微软商标和徽标的授权使用须遵守
[微软商标与品牌准则](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general)。
在本项目的修改版本中使用微软商标或徽标,不得造成混淆或暗示微软的赞助。
任何第三方商标或徽标的使用须遵守相应第三方的政策。
