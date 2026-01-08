# Gemini-Business-2API

**Gemini-Business-2API** 是一个将 Google Gemini Business (`business.gemini.google`) 网页版逆向转换为 OpenAI 兼容 API 的网关服务。

它允许你使用支持 OpenAI API 的客户端（如 NextChat, LangChain, OpenAI Python SDK 等）直接调用 Gemini Business 的模型能力，支持流式输出和多模态（文本+图片）输入。

## ✨ 特性

*   **OpenAI 接口兼容**: 提供 `/v1/chat/completions` 和 `/v1/models` 接口，无缝对接现有生态。
*   **多模态支持**: 支持发送文本和图片（Base64 编码或 Data URI）。
*   **流式响应**: 支持 Server-Sent Events (SSE) 流式输出，打字机效果流畅。
*   **自动会话管理**: 自动创建和维护 Gemini 会话，支持多轮对话上下文。
*   **配置提取助手**: 提供配套的 UserScript 脚本，一键从浏览器提取所需的 Cookie 和 ID。
*   **模型支持**: 支持 `gemini-2.5-flash`, `gemini-2.5-pro`, `gemini-3-pro-preview` 等模型。

## 🛠️ 准备工作

在使用本项目之前，你需要：

1.  一个拥有 **Gemini Business** 权限的 Google 账号。
2.  安装 Tampermonkey 或 Violentmonkey 浏览器扩展（用于运行配置提取脚本）。

## 🚀 快速开始

### 第一步：获取配置信息

为了让服务能够代表你访问 Gemini Business，你需要提取认证信息。我们提供了一个便捷的 UserScript 来完成此操作。

1.  安装 [Gemini Business 2API Helper.user.js](./Gemini%20Business%202API%20Helper.user.js) 脚本到你的浏览器插件中。
2.  登录 [Gemini Business](https://business.gemini.google/)。
3.  进入任意对话页面，点击页面右下角的浮动按钮（带有 2API 标识）。
4.  在弹出的面板中点击“复制配置”，你将获得类似如下的内容：

```env
SECURE_C_SES=...
CSESIDX=...
CONFIG_ID=...
HOST_C_OSES=...
PROXY=
```

### 第二步：部署服务

你可以选择使用 Docker 或直接运行 Python 源码。

#### 方式 A: Docker Compose (推荐)

1.  克隆本项目或下载文件。
2.  在项目根目录下创建一个 `.env` 文件，并将第一步中复制的配置粘贴进去。
3.  运行 Docker Compose：

```bash
docker-compose up -d
```

服务将在 `http://localhost:3003` 启动。

#### 方式 B: 本地 Python 运行

1.  确保安装了 Python 3.11+。
2.  安装依赖：

```bash
pip install -r requirements.txt
```

3.  设置环境变量（Linux/Mac 示例，Windows 请用 `set` 或直接修改代码）：

```bash
export SECURE_C_SES="你的SECURE_C_SES"
export CSESIDX="你的CSESIDX"
export CONFIG_ID="你的CONFIG_ID"
export HOST_C_OSES="你的HOST_C_OSES"
# 可选
export PROXY="http://127.0.0.1:7890"
```

4.  运行服务：

```bash
python main.py
```

服务将在 `http://localhost:8000` 启动（注意 Docker 映射到了 3003，本地直接运行默认是 8000）。

## 🔌 API 使用

### Base URL

*   Docker 部署: `http://localhost:3003/v1`
*   本地运行: `http://localhost:8000/v1`

### 支持的模型

*   `gemini-auto` (默认)
*   `gemini-2.5-flash`
*   `gemini-2.5-pro`
*   `gemini-3-pro-preview`

### 示例代码 (Python)

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:3003/v1",
    api_key="dummy-key" # 或者是任意非空字符串
)

# 文本对话
response = client.chat.completions.create(
    model="gemini-2.5-pro",
    messages=[
        {"role": "user", "content": "你好，请介绍一下你自己。"}
    ],
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

### 示例请求 (cURL)

```bash
curl http://localhost:3003/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy-key" \
  -d 
{
    "model": "gemini-2.5-flash",
    "messages": [
      {
        "role": "user",
        "content": "Hello!"
      }
    ],
    "stream": true
  }
```

## ⚙️ 环境变量说明

| 变量名 | 必填 | 说明 |
| :--- | :---: | :--- |
| `SECURE_C_SES` | ✅ | 核心认证 Cookie (`__Secure-C_SES`) |
| `CSESIDX` | ✅ | 会话索引 ID，通常在 URL 参数中 |
| `CONFIG_ID` | ✅ | 配置 ID，通常在 URL 路径中 |
| `HOST_C_OSES` | ❌ | 辅助认证 Cookie (`__Host-C_OSES`)，部分环境可能需要 |
| `PROXY` | ❌ | HTTP 代理地址 (如 `http://127.0.0.1:7890`) |
| `TIMEOUT_SECONDS` | ❌ | 请求超时时间，默认 600 秒 |

## ⚠️ 免责声明

*   本项目仅供学习和研究目的使用。
*   本项目与 Google 或 Gemini 没有任何官方关联。
*   使用本项目产生的任何后果（如账号被封禁）由使用者自行承担。
*   请勿将本项目用于非法用途。
