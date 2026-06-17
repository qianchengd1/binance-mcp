# Binance Square MCP 使用文档

## 1. 服务说明

本 MCP 服务用于让支持 MCP 的 Agent 调用 Binance Square 发帖功能。

服务提供一个工具：

```text
publish_binance_square
```

支持：

- 发布文字帖子
- 发布一张 PNG、JPEG、WebP 或 GIF 图片
- 自动获取当前账号的 `squareUid` 并生成签名
- 关联现货或合约币种
- 返回 Binance API 的结构化结果

远程版本不会保存用户传入的 Cookie。Cookie 只在当前请求中用于调用 Binance。

## 2. MCP 服务地址

正式服务地址：

```text
https://qianxin.xyz/mcp
```

健康检查地址：

```text
https://mcp.qianxin.xyz/health
```

健康检查正常时应返回：

```json
{
  "status": "ok",
  "service": "binance-square-mcp"
}
```

浏览器直接打开 `/mcp` 不一定显示普通网页，因为它是 MCP 协议接口。

## 3. Agent 配置

在支持远程 MCP 的 Agent 中添加：

```json
{
  "mcpServers": {
    "binance-square": {
      "url": "https://mcp.qianxin.xyz/mcp"
    }
  }
}
```

保存配置后重启 Agent。

连接成功后，Agent 的工具列表中应出现：

```text
publish_binance_square
```

不同 Agent 的 MCP 配置界面和配置文件位置可能不同，但服务 URL 相同。

## 4. 工具参数

### 必填参数

| 参数 | 类型 | 说明 |
|---|---|---|
| `cookie` | string | 用户自己的完整 Binance 浏览器 Cookie |
| `content` | string | 要发布的最终帖子正文 |

### 可选参数

| 参数 | 类型 | 说明 |
|---|---|---|
| `image_base64` | string | 图片的 Base64 内容或 `data:image/...;base64,...` Data URL |
| `coins` | string | 关联币种，例如 `BTC:spot,ETH:future` |
| `user_agent` | string | 获取 Cookie 时使用的浏览器 User-Agent |
| `signature_key` | string | 当前账号的 `squareUid`，不填则自动获取 |

### 币种格式

关联现货：

```text
BTC:spot
```

关联合约：

```text
ETH:future
```

关联多个币种：

```text
BTC:spot,ETH:future,SOL:spot
```

## 5. 调用示例

### 纯文字发帖

工具参数：

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "这是一个测试帖子"
}
```

可以直接对 Agent 说：

```text
调用 publish_binance_square，使用我提供的 Cookie，
发布“这是一个测试帖子”。
```

### 关联 BTC

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "今天继续关注 BTC 的行情变化。",
  "coins": "BTC:spot"
}
```

### 关联多个币种

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "今天关注 BTC 和 ETH 的市场走势。",
  "coins": "BTC:spot,ETH:future"
}
```

### 携带图片发帖

远程 MCP 不读取 Agent 电脑上的文件路径。Agent 需要先读取图片文件，
将图片转换成 Base64，再传给 `image_base64`：

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "分享一张行情图片。",
  "image_base64": "iVBORw0KGgoAAAANSUhEUgAA..."
}
```

也支持 Data URL：

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "分享一张行情图片。",
  "image_base64": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}
```

图片要求：

- 支持 PNG、JPEG、WebP、GIF
- 解码后的图片最大 8MB
- 每次调用支持一张图片
- 上传完成或失败后，服务会立即删除临时文件

## 6. 返回结果

发布成功示例：

```json
{
  "success": true,
  "status_code": 200,
  "response": {
    "success": true,
    "data": {}
  },
  "error": null
}
```

发布失败示例：

```json
{
  "success": false,
  "status_code": 200,
  "response": {
    "code": "10002",
    "message": "Valid error",
    "success": false
  },
  "error": "Valid error"
}
```

HTTP 状态码为 `200` 不一定表示发帖成功，还必须检查返回结果中的 `success`。

## 7. Cookie 获取

1. 使用浏览器登录 Binance。
2. 打开 Binance Square 页面。
3. 按 `F12` 打开开发者工具。
4. 进入 `Network`。
5. 刷新页面或执行一次相关操作。
6. 点击 Binance 请求，在请求头中找到 `Cookie`。
7. 复制完整 Cookie 字符串。

Cookie 中通常应包含：

```text
bnc-uuid
BNC_FV_KEY
BNC_FV_KEY_T
BNC_FV_KEY_EXPIRE
cr00
logined=y
```

Cookie 过期后需要重新获取。

## 8. 常见错误

### Agent 看不到工具

检查：

- MCP URL 是否为 `https://mcp.qianxin.xyz/mcp`
- Agent 是否支持远程 Streamable HTTP MCP
- 修改配置后是否重启 Agent
- `https://mcp.qianxin.xyz/health` 是否正常

### 无法连接服务

检查：

- 域名 `mcp.qianxin.xyz` 是否解析到服务器公网 IP
- Caddy 是否正在运行
- Python MCP 服务是否正在监听 `127.0.0.1:8000`
- 服务器安全组是否开放 TCP `80` 和 `443`
- Windows 防火墙是否允许 Caddy

### 返回 `10002 Valid error`

常见原因：

- Cookie 已失效
- Cookie 与 User-Agent 不匹配
- 无法获取当前账号 `squareUid`
- Binance 网页接口发生变化

建议重新登录 Binance，并重新复制完整 Cookie。

### 返回无法获取 nonce

通常表示：

- Cookie 已过期
- Binance 登录状态失效
- 服务器无法访问 Binance
- Binance 风控拦截了当前服务器 IP
