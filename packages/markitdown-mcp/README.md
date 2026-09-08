> 🌐 本文档由 [microsoft/markitdown](https://github.com/microsoft/markitdown) 翻译,英文原版见原项目。

# MarkItDown-MCP

> [!IMPORTANT]
> MarkItDown-MCP 包面向**本地使用**,请配合本地可信的智能体使用。特别注意:以 Streamable HTTP 或 SSE 方式运行 MCP 服务器时,它默认只绑定 `localhost`,不会暴露给网络中的其他机器或互联网。这种配置下,它可以作为 STDIO 传输方式的直接替代,在某些场景下更为方便。除非你清楚这样做的[安全影响](#安全注意事项),否则不要把服务器绑定到其他网络接口。


[![PyPI](https://img.shields.io/pypi/v/markitdown-mcp.svg)](https://pypi.org/project/markitdown-mcp/)
![PyPI - Downloads](https://img.shields.io/pypi/dd/markitdown-mcp)
[![Built by AutoGen Team](https://img.shields.io/badge/Built%20by-AutoGen%20Team-blue)](https://github.com/microsoft/autogen)

`markitdown-mcp` 包提供一个轻量级的 MCP 服务器,支持 STDIO、Streamable HTTP 和 SSE 三种传输方式,用于调用 MarkItDown。

它只暴露一个工具:`convert_to_markdown(uri)`,其中 uri 可以是任意 `http:`、`https:`、`file:` 或 `data:` URI。

## 安装

使用 pip 安装:

```bash
pip install markitdown-mcp
```

## 用法

以 STDIO 方式(默认)运行 MCP 服务器:

```bash
markitdown-mcp
```

以 Streamable HTTP 和 SSE 方式运行 MCP 服务器:

```bash
markitdown-mcp --http --host 127.0.0.1 --port 3001
```

## 在 Docker 中运行

要在 Docker 中运行 `markitdown-mcp`,先使用仓库自带的 Dockerfile 构建镜像:
```bash
docker build -t markitdown-mcp:latest .
```

然后运行:
```bash
docker run -it --rm markitdown-mcp:latest
```
这样就能处理远程 URI。如果需要访问本地文件,必须把本地目录挂载进容器。例如,要访问 `/home/user/data` 下的文件,可以这样运行:

```bash
docker run -it --rm -v /home/user/data:/workdir markitdown-mcp:latest
```

挂载完成后,`data` 目录下的所有文件都可以通过容器内的 `/workdir` 访问。例如 `/home/user/data` 下有一个 `example.txt`,在容器内就是 `/workdir/example.txt`。

## 从 Claude Desktop 访问

在为 Claude Desktop 运行 MCP 服务器时,建议使用 Docker 镜像。

按照[这些说明](https://modelcontextprotocol.io/quickstart/user#for-claude-desktop-users)找到 Claude 的 `claude_desktop_config.json` 文件。

编辑该文件,加入如下 JSON 配置:

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "markitdown-mcp:latest"
      ]
    }
  }
}
```

如果想挂载目录,相应调整即可:

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-v",
        "/home/user/data:/workdir",
        "markitdown-mcp:latest"
      ]
    }
  }
}
```

## 调试

可以使用 `MCP Inspector` 工具调试 MCP 服务器。

```bash
npx @modelcontextprotocol/inspector
```

然后通过指定的主机和端口(例如 `http://localhost:5173/`)连接到 Inspector。

如果使用 STDIO:
* 传输类型选择 `STDIO`,
* 命令填 `markitdown-mcp`,
* 点击 `Connect`。

如果使用 Streamable HTTP:
* 传输类型选择 `Streamable HTTP`,
* URL 填 `http://127.0.0.1:3001/mcp`,
* 点击 `Connect`。

如果使用 SSE:
* 传输类型选择 `SSE`,
* URL 填 `http://127.0.0.1:3001/sse`,
* 点击 `Connect`。

最后:
* 点击 `Tools` 标签页,
* 点击 `List Tools`,
* 点击 `convert_to_markdown`,
* 用任意合法 URI 运行该工具。

## 安全注意事项

该服务器不支持身份验证,并以运行它的用户权限执行。因此,在 SSE 或 Streamable HTTP 模式下,服务器默认只绑定 `localhost`。即便如此也要清楚:同一台本地机器上的任何进程或用户都可以访问该服务器,而且 `convert_to_markdown` 工具可以被用来读取服务器用户可访问的任意文件,或从网络获取任意数据。如果需要更高的安全性,建议在虚拟机或容器等沙箱环境中运行服务器,并正确配置用户权限,限制对敏感文件和网络段的访问。最重要的是:除非你清楚这样做的安全影响,否则不要把服务器绑定到非 localhost 的其他网络接口。

## 商标声明

本项目可能包含相关项目、产品或服务的商标或徽标。微软商标和徽标的授权使用须遵守
[微软商标与品牌准则](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general)。
在本项目的修改版本中使用微软商标或徽标,不得造成混淆或暗示微软的赞助。
任何第三方商标或徽标的使用须遵守相应第三方的政策。
