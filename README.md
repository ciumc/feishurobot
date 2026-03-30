# Feishu Custom Bot SDK

[![Go Version](https://img.shields.io/badge/Go-1.16+-00ADD8?style=flat&logo=go)](https://golang.org/)
[![Test Coverage](https://img.shields.io/badge/Coverage-90.2%25-green?style=flat)](https://golang.org/)
[![License](https://img.shields.io/badge/License-MIT-blue?style=flat)](LICENSE)
[![GoDoc](https://img.shields.io/badge/GoDoc-Reference-00ADD8?style=flat)](https://pkg.go.dev/github.com/ciumc/feishurobot)

A Go SDK toolkit for Feishu (Lark) custom bot webhook API, supporting message sending with optional security verification (signatures).

飞书自定义机器人 Webhook API 的 Go SDK 工具包，支持发送各类消息并提供可选的安全签名验证。

## Features / 功能特性

- ✅ Send text messages with @mentions / 发送文本消息，支持 @提及
- ✅ Send rich text (post) messages with flexible element builders / 发送富文本消息，提供灵活的元素构建器
- ✅ Send image messages / 发送图片消息
- ✅ Send share chat (group card) messages / 发送分享群名片消息
- ✅ Send interactive card messages with builder pattern / 发送交互式卡片消息，支持构建器模式
- ✅ Multi-language support for post messages / 富文本消息支持多语言
- ✅ Optional signature verification (HmacSHA256 + Base64) / 可选签名验证
- ✅ Context support for request cancellation / 支持上下文取消请求
- ✅ 90%+ test coverage / 90%+ 测试覆盖率

## Installation / 安装

```bash
go get github.com/ciumc/feishurobot
```

## Quick Start / 快速开始

### Basic Usage / 基本用法

```go
package main

import (
    "context"
    "log"

    "github.com/ciumc/feishurobot"
)

func main() {
    // Create client with webhook URL / 使用 webhook URL 创建客户端
    client := feishubot.NewClient(
        "https://open.feishu.cn/open-apis/bot/v2/hook/your_webhook_id",
        "", // No secret / 不使用签名
    )

    // Send text message / 发送文本消息
    message := feishubot.NewTextMessage("Hello, World!")
    resp, err := client.Send(context.Background(), message)
    if err != nil {
        log.Fatalf("Failed to send message: %v", err)
    }

    log.Printf("Message sent! Code: %d, Msg: %s", resp.Code, resp.Msg)
}
```

### With Signature Verification / 带签名验证

```go
// Create client with webhook URL and secret / 使用 webhook URL 和密钥创建客户端
client := feishubot.NewClient(
    "https://open.feishu.cn/open-apis/bot/v2/hook/your_webhook_id",
    "your_secret_here", // Enable signature verification / 启用签名验证
)

// Messages will automatically include timestamp and signature / 消息会自动包含时间戳和签名
message := feishubot.NewTextMessage("Secure message!")
resp, err := client.Send(context.Background(), message)
```

## Message Types / 消息类型

### Text Message / 文本消息

```go
// Simple text / 简单文本
message := feishubot.NewTextMessage("Hello, World!")

// @ single user (replace with actual user ID) / @ 单个用户
message := feishubot.NewTextMessage(`<at user_id="ou_xxx">Tom</at> notification for you.`)

// @ all users / @ 所有人
message := feishubot.NewTextMessage(`<at user_id="all">所有人</at> announcement.`)
```

### Rich Text (Post) Message / 富文本消息

#### Simple Post / 简单富文本

```go
content := feishubot.NewPostContent(
    "Project Update",
    feishubot.NewParagraph(
        feishubot.NewTextElement("Project has been updated successfully!"),
    ),
)

message := feishubot.NewPostMessage(feishubot.LanguageZhCN, content)
```

#### Post with Multiple Elements / 多元素富文本

```go
content := feishubot.NewPostContent(
    "Daily Report",
    feishubot.NewParagraph(
        feishubot.NewTextElement("Project update: "),
        feishubot.NewLinkElement("View Details", "https://example.com/project/123"),
        feishubot.NewTextElement(" - Assigned to: "),
        feishubot.NewAtElement("ou_xxx", "Tom"),
    ),
    feishubot.NewParagraph(
        feishubot.NewTextElement("Please review the changes by end of day."),
    ),
)

message := feishubot.NewPostMessage(feishubot.LanguageZhCN, content)
```

#### Available Element Types / 可用元素类型

| Function / 函数 | Description / 描述 |
|----------------|-------------------|
| `NewTextElement(text)` | Plain text / 纯文本 |
| `NewLinkElement(text, href)` | Hyperlink / 超链接 |
| `NewAtElement(userID, userName)` | @ mention / @提及 |
| `NewImageElement(imageKey)` | Inline image / 内联图片 |
| `NewEmoticonElement(emojiKey)` | Emoji / 表情 |

#### Multi-Language Post / 多语言富文本

```go
content := feishubot.NewPostContent(
    "Title",
    feishubot.NewParagraph(feishubot.NewTextElement("Content")),
)

message := feishubot.NewPostMessageMultiLanguage(
    feishubot.NewPostLanguageContent(feishubot.LanguageZhCN, content),
    feishubot.NewPostLanguageContent(feishubot.LanguageEnUS, content),
)
```

### Image Message / 图片消息

```go
message := feishubot.NewImageMessage("img_ecffc3b9-8f14-400f-a014-05eca1a4310g")
```

> The imageKey must be obtained from Feishu image upload API.
> imageKey 需要通过飞书图片上传 API 获取。

### Share Chat Message / 分享群名片消息

```go
message := feishubot.NewShareChatMessage("oc_f5b1a7eb27ae2****339ff")
```

> The bot can only share the group it belongs to.
> 机器人只能分享其所在的群。

### Interactive Card Message / 交互式卡片消息

#### Simple Card / 简单卡片

```go
card := feishubot.NewCard("2.0").
    SetHeader(&feishubot.CardHeader{
        Title:    feishubot.NewCardTitle("Welcome"),
        Template: "blue",
    }).
    SetBody(&feishubot.CardBody{
        Elements: []feishubot.CardElement{
            feishubot.NewMarkdownElement("Hello! This is an interactive card message."),
        },
    })

message := feishubot.NewInteractiveMessage(card)
```

#### Card with Markdown and Buttons / 带 Markdown 和按钮的卡片

```go
card := feishubot.NewCard("2.0").
    SetConfig(map[string]interface{}{
        "wide_screen_mode": true,
    }).
    SetHeader(&feishubot.CardHeader{
        Title:    feishubot.NewCardTitle("Task Alert"),
        Template: "red",
    }).
    SetBody(&feishubot.CardBody{
        Direction: "vertical",
        Padding:   "12px 12px 12px 12px",
        Elements: []feishubot.CardElement{
            feishubot.NewMarkdownElement("**High Priority Task**\n\nPlease complete by EOD."),
            feishubot.NewButtonElement("View Details", "primary", "https://example.com/task/123"),
            feishubot.NewButtonElement("Dismiss", "default", "https://example.com/dismiss"),
        },
    })

message := feishubot.NewInteractiveMessage(card)
```

#### Card from Map / 从 Map 创建卡片

```go
cardMap := map[string]interface{}{
    "schema": "2.0",
    "body": map[string]interface{}{
        "elements": []map[string]interface{}{
            {"tag": "markdown", "content": "Your card content here..."},
        },
    },
}

message := feishubot.NewInteractiveMessageFromMap(cardMap)
```

> Use this with [Feishu Card Builder Tool](https://open.feishu.cn/document/uAjLw4CM/ukzMukzMukzM/feishu-cards/feishu-card-cardkit/feishu-cardkit-overview).
> 可配合飞书卡片搭建工具使用。

## API Reference / API 参考

### Client Methods / 客户端方法

| Method / 方法 | Description / 描述 |
|--------------|-------------------|
| `NewClient(webhookURL, secret)` | Create new client / 创建新客户端 |
| `client.Send(ctx, message)` | Send message / 发送消息 |
| `client.SetHTTPClient(httpClient)` | Set custom HTTP client / 设置自定义 HTTP 客户端 |

### Message Constructors / 消息构造器

| Function / 函数 | Description / 描述 |
|----------------|-------------------|
| `NewTextMessage(text)` | Text message / 文本消息 |
| `NewPostMessage(lang, content)` | Post message / 富文本消息 |
| `NewPostMessageMultiLanguage(...)` | Multi-language post / 多语言富文本 |
| `NewImageMessage(imageKey)` | Image message / 图片消息 |
| `NewShareChatMessage(shareChatID)` | Share chat message / 分享群名片消息 |
| `NewInteractiveMessage(card)` | Interactive card / 交互式卡片 |
| `NewInteractiveMessageFromMap(map)` | Card from map / 从 Map 创建卡片 |

### Language Constants / 语言常量

```go
LanguageZhCN  // Simplified Chinese / 简体中文
LanguageEnUS  // English / 英语
LanguageJa    // Japanese / 日语
```

### Response Structure / 响应结构

```go
type Response struct {
    Code    int         `json:"code"`     // 0 = success / 0 表示成功
    Msg     string      `json:"msg"`      // Response message / 响应消息
    Data    interface{} `json:"data"`     // Response data / 响应数据
}
```

## Rate Limits / 速率限制

| Limit / 限制 | Value / 值 |
|-------------|-----------|
| Requests per minute / 每分钟请求 | 100 |
| Requests per second / 每秒请求 | 5 |
| Request body size / 请求体大小 | ≤ 20 KB |

> Avoid sending messages at peak times (10:00, 17:30, etc.) to prevent rate limiting errors.
> 避免在高峰时段（如 10:00、17:30）发送消息，以免触发速率限制错误。

## Error Codes / 错误码

| Code | Message | Description / 描述 |
|------|---------|-------------------|
| 0 | success | Success / 成功 |
| 9499 | Bad Request | Invalid request format / 请求格式无效 |
| 19021 | sign match fail... | Signature verification failed / 签名验证失败 |
| 19022 | Ip Not Allowed | IP not in whitelist / IP 不在白名单 |
| 19024 | Key Words Not Found | Keywords not found / 关键词未找到 |

## Project Structure / 项目结构

```
feishurobot/
├── bot.go           # Client and send logic / 客户端和发送逻辑
├── bot_test.go      # Client tests / 客户端测试
├── message.go       # Message types and builders / 消息类型和构建器
├── message_test.go  # Message tests / 消息测试
├── sign.go          # Signature generation / 签名生成
├── sign_test.go     # Signature tests / 签名测试
├── cmd/example/     # Example usage / 示例代码
├── go.mod           # Go module file / Go 模块文件
└── README.md        # This file / 本文档
```

## Development / 开发

### Running Tests / 运行测试

```bash
go test ./... -v           # Run all tests / 运行所有测试
go test ./... -cover       # Run with coverage / 运行并显示覆盖率
go test ./... -race        # Run with race detection / 运行竞态检测
```

### Running Example / 运行示例

```bash
export FEISHU_WEBHOOK_URL="your_webhook_url"
export FEISHU_SECRET="your_secret"  # Optional / 可选
go run cmd/example/main.go
```

## License / 许可证

MIT License

## Contributing / 贡献

Contributions are welcome! Please feel free to submit a Pull Request.
欢迎贡献代码！请随时提交 Pull Request。

## References / 参考资料

- [Feishu Custom Bot Documentation / 飞书自定义机器人文档](https://open.feishu.cn/document/uAjLw4CM/ukTMukTMukTM/bot-v3/add-custom-bot)
- [Feishu Card Builder / 飞书卡片搭建工具](https://open.feishu.cn/document/uAjLw4CM/ukzMukzMukzM/feishu-cards/feishu-card-cardkit/feishu-cardkit-overview)