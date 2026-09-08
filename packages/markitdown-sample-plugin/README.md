> 🌐 本文档由 [microsoft/markitdown](https://github.com/microsoft/markitdown) 翻译,英文原版见原项目。

# MarkItDown 示例插件

[![PyPI](https://img.shields.io/pypi/v/markitdown-sample-plugin.svg)](https://pypi.org/project/markitdown-sample-plugin/)
![PyPI - Downloads](https://img.shields.io/pypi/dd/markitdown-sample-plugin)
[![Built by AutoGen Team](https://img.shields.io/badge/Built%20by-AutoGen%20Team-blue)](https://github.com/microsoft/autogen)


本项目演示如何为 MarkItDown 编写一个示例插件。最关键的部分如下:

首先,实现你自定义的 DocumentConverter:

```python
from typing import BinaryIO, Any
from markitdown import MarkItDown, DocumentConverter, DocumentConverterResult, StreamInfo, PRIORITY_SPECIFIC_FILE_FORMAT

class RtfConverter(DocumentConverter):

    def __init__(
        self, priority: float = PRIORITY_SPECIFIC_FILE_FORMAT
    ):
        super().__init__(priority=priority)

    def accepts(
        self,
        file_stream: BinaryIO,
        stream_info: StreamInfo,
        **kwargs: Any,
    ) -> bool:

        # 在这里实现判断文件流是否为 RTF 文件的逻辑
        # ...
        raise NotImplementedError()


    def convert(
        self,
        file_stream: BinaryIO,
        stream_info: StreamInfo,
        **kwargs: Any,
    ) -> DocumentConverterResult:

        # 在这里实现把文件流转换为 Markdown 的逻辑
        # ...
        raise NotImplementedError()
```

接着,确保你的包实现并导出以下内容:

```python
# The version of the plugin interface that this plugin uses.
# The only supported version is 1 for now.
__plugin_interface_version__ = 1

# The main entrypoint for the plugin. This is called each time MarkItDown instances are created.
def register_converters(markitdown: MarkItDown, **kwargs):
    """
    Called during construction of MarkItDown instances to register converters provided by plugins.
    """

    # Simply create and attach an RtfConverter instance
    markitdown.register_converter(RtfConverter())
```


最后,在 `pyproject.toml` 文件中创建入口点:

```toml
[project.entry-points."markitdown.plugin"]
sample_plugin = "markitdown_sample_plugin"
```

其中 `sample_plugin` 的键名可以任意取,但最好用插件的名字。值是实现插件的包的完整限定名。


## 安装

要在 MarkItDown 中使用插件,必须先安装它。从当前目录安装插件:

```bash
pip install -e .
```

插件包装好之后,运行以下命令验证 MarkItDown 能发现它:

```bash
markitdown --list-plugins
```

转换时使用 `--use-plugins` 标志启用插件。例如转换一个 RTF 文件:

```bash
markitdown --use-plugins path-to-file.rtf
```

在 Python 中可以这样启用插件:

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=True)
result = md.convert("path-to-file.rtf")
print(result.text_content)
```

## 商标声明

本项目可能包含相关项目、产品或服务的商标或徽标。微软商标和徽标的授权使用须遵守
[微软商标与品牌准则](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general)。
在本项目的修改版本中使用微软商标或徽标,不得造成混淆或暗示微软的赞助。
任何第三方商标或徽标的使用须遵守相应第三方的政策。
