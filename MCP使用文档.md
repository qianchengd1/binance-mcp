# Binance Square MCP 使用文档

本文档面向使用者，说明如何通过 MCP 工具发布 Binance Square 帖子。

## 1. 服务地址

MCP 地址：

```text
https://qianxin.xyz/mcp
```

健康检查地址：

```text
https://qianxin.xyz/health
```

浏览器直接打开 `/mcp` 不一定会显示网页，因为它是 MCP 协议接口。

## 2. Agent 配置

在支持远程 MCP 的 Agent 中添加：

```json
{
  "mcpServers": {
    "binance-square": {
      "url": "https://qianxin.xyz/mcp"
    }
  }
}
```

保存后重启 Agent。连接成功后，工具列表中应出现：

```text
publish_binance_square
```

## 3. 工具能力

`publish_binance_square` 支持：

- 发布文字帖子
- 携带一张图片
- 关联币种链接
- 自动获取账号 `squareUid` 并生成发布签名
- 返回 Binance API 的结构化结果

服务不会保存用户传入的 Cookie。Cookie 只用于本次发布请求。

## 4. 参数说明

必填参数：

| 参数 | 类型 | 说明 |
|---|---|---|
| `cookie` | string | 完整的 Binance 浏览器 Cookie |
| `content` | string | 最终要发布的帖子正文 |

可选参数：

| 参数 | 类型 | 说明 |
|---|---|---|
| `image_base64` | string | 图片 Base64，或 `data:image/...;base64,...` Data URL |
| `coins` | string | 关联币种，例如 `AGLD`、`BTC:spot`、`ETH:future` |
| `user_agent` | string | 获取 Cookie 时使用的浏览器 User-Agent |
| `signature_key` | string | 当前账号的 `squareUid`；通常留空，让服务自动获取 |
| `proxy` | string | 请求级代理覆盖；通常不要传，服务默认使用 `http://127.0.0.1:7892` |

## 5. 调用示例

纯文字：

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "这是一条测试帖子"
}
```

带图片：

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "这是一条带图测试帖子",
  "image_base64": "iVBORw0KGgoAAAANSUhEUgAA..."
}
```

带图片 Data URL：

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "这是一条带图测试帖子",
  "image_base64": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}
```

关联币种：

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "今天关注 $AGLD 的市场变化。",
  "coins": "AGLD"
}
```

关联多个币种：

```json
{
  "cookie": "完整的 Binance Cookie",
  "content": "今天关注 BTC 和 ETH 的市场变化。",
  "coins": "BTC:spot,ETH:future"
}
```

## 6. 图片要求

- 支持 PNG、JPEG、WebP、GIF
- 每次调用最多一张图片
- 图片解码后最大 8 MB
- 远程 MCP 不读取本地文件路径，必须传 `image_base64`

## 7. Cookie 获取建议

1. 使用浏览器登录 Binance。
2. 打开 Binance Square 页面。
3. 打开开发者工具，进入 `Network`。
4. 刷新页面或执行一次相关操作。
5. 点击 Binance 请求，在请求头中复制完整 `Cookie`。

Cookie 通常应包含：

```text
bnc-uuid
BNC_FV_KEY
BNC_FV_KEY_T
BNC_FV_KEY_EXPIRE
cr00
r20t
p20t
d1og
r2o1
f30l
logined=y
aws-waf-token
```

Cookie 过期后需要重新登录并复制新的 Cookie。

## 8. 返回结果

成功示例：

```json
{
  "success": true,
  "status_code": 200,
  "response": {
    "code": "000000",
    "data": {
      "id": 337615371354050,
      "shareLink": "https://app.binance.com/uni-qr/cpos/..."
    },
    "success": true
  },
  "error": null
}
```

失败示例：

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

注意：HTTP `200` 不代表发布成功，必须看返回里的 `success` 是否为 `true`。

## 9. 常见问题

### 看不到工具

检查：

- MCP URL 是否是 `https://qianxin.xyz/mcp`
- Agent 是否支持远程 Streamable HTTP MCP
- 修改配置后是否重启 Agent
- `https://qianxin.xyz/health` 是否正常

### 无法获取 squareUid

常见原因：

- Cookie 已过期
- Cookie 不完整
- Binance 返回 `Please log in first`
- 代理未开启或代理节点访问 Binance 不稳定
- Cookie 与 User-Agent 不匹配

处理方式：

- 重新登录 Binance 后复制完整 Cookie
- 确认服务端代理 `127.0.0.1:7892` 已开启
- 查看服务端 `signature_key.response` 日志
- 必要时传入浏览器 User-Agent

### 返回 `10002 Valid error`

常见原因：

- Cookie 登录态无效
- 自动获取的 `squareUid` 或签名流程失败
- Binance 风控或接口校验失败

处理方式：

- 刷新 Cookie
- 不要手动乱传 `signature_key`
- 查看服务端日志中的 Binance 响应

### 图片发布失败

检查：

- `image_base64` 是否有效
- 图片解码后是否小于 8 MB
- 图片格式是否为 PNG/JPEG/WebP/GIF

## 10. 安全提醒

- Cookie 等同于登录凭证，不要发给不可信服务。
- 不要把 Cookie 写入公开代码、截图、日志或文档。
- 发布是真实外部操作，重复调用会产生重复帖子。
- 建议每次发布前确认最终正文、图片和币种。
