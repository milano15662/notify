# notify

A flexible and extensible Go library for sending notifications across multiple platforms including Telegram, Slack, and more.

[![Go Reference](https://pkg.go.dev/badge/github.com/milano15662/notify.svg)](https://pkg.go.dev/github.com/milano15662/notify)
[![Go Report Card](https://goreportcard.com/badge/github.com/milano15662/notify)](https://goreportcard.com/report/github.com/milano15662/notify)

## Features

- 🚀 **Multi-Platform Support**: Send notifications to Telegram, Slack, and easily extend to other platforms
- 🎯 **Simple Interface**: Clean and intuitive API for sending notifications
- 📦 **Manager Pattern**: Centralized management of multiple notification providers
- ⚡ **Async Broadcasting**: Send notifications to multiple platforms concurrently
- 🔧 **Extensible**: Implement your own custom notification providers
- 🎨 **Rich Messages**: Support for titles, attachments, fields, and formatting
- ✅ **Type Safe**: Full type safety with Go interfaces
- 🧪 **Well Tested**: Comprehensive test coverage

## Installation

```bash
go get github.com/milano15662/notify
```

## Quick Start

### Telegram

```go
package main

import (
    "context"
    "log"
    
    "github.com/milano15662/notify"
)

func main() {
    ctx := context.Background()
    
    // Create a Telegram notifier
    telegram, err := notify.NewTelegramNotifier(notify.TelegramConfig{
        BotToken: "YOUR_BOT_TOKEN",
        ChatID:   "YOUR_CHAT_ID",
    })
    if err != nil {
        log.Fatal(err)
    }
    
    // Send a simple message
    err = telegram.Send(ctx, "Hello from notify! 🚀")
    if err != nil {
        log.Fatal(err)
    }
}
```

### Slack

```go
package main

import (
    "context"
    "log"
    
    "github.com/milano15662/notify"
)

func main() {
    ctx := context.Background()
    
    // Create a Slack notifier
    slack, err := notify.NewSlackNotifier(notify.SlackConfig{
        Token:          "YOUR_SLACK_TOKEN",
        DefaultChannel: "#general",
    })
    if err != nil {
        log.Fatal(err)
    }
    
    // Send a simple message
    err = slack.Send(ctx, "Hello from notify! 🚀")
    if err != nil {
        log.Fatal(err)
    }
}
```

## Usage

### Rich Messages

Send messages with titles, priorities, and attachments:

```go
msg := &notify.Message{
    Title:    "System Alert",
    Text:     "Server CPU usage is at 85%",
    Priority: notify.PriorityHigh,
    Attachments: []notify.Attachment{
        {
            Title: "Details",
            Color: "warning",
            Fields: []notify.Field{
                {Title: "Server", Value: "prod-01", Short: true},
                {Title: "Region", Value: "us-east-1", Short: true},
            },
        },
    },
}

err := notifier.SendWithOptions(ctx, msg)
```

### Manager - Multiple Providers

Use the Manager to handle multiple notification providers:

```go
// Create a manager
manager := notify.NewManager()

// Register providers
telegram, _ := notify.NewTelegramNotifier(telegramConfig)
slack, _ := notify.NewSlackNotifier(slackConfig)

manager.Register(telegram)
manager.Register(slack)

// Send to specific provider
manager.Send(ctx, "telegram", "Hello Telegram!")

// Broadcast to all providers (synchronous)
errors := manager.Broadcast(ctx, "Hello everyone!")

// Broadcast asynchronously
resultChan := manager.BroadcastAsync(ctx, "Async broadcast!")
for result := range resultChan {
    if result.Success {
        fmt.Printf("✓ %s: Success\n", result.Provider)
    } else {
        fmt.Printf("✗ %s: %v\n", result.Provider, result.Error)
    }
}
```

### Custom Notifier

Implement your own notification provider:

```go
type EmailNotifier struct {
    smtpServer string
    from       string
    to         string
}

func (e *EmailNotifier) Name() string {
    return "email"
}

func (e *EmailNotifier) Send(ctx context.Context, message string) error {
    // Implement your email sending logic
    return nil
}

func (e *EmailNotifier) SendWithOptions(ctx context.Context, msg *notify.Message) error {
    // Implement rich email sending logic
    return nil
}

// Use it
emailNotifier := &EmailNotifier{
    smtpServer: "smtp.gmail.com:587",
    from:       "bot@example.com",
    to:         "admin@example.com",
}

manager.Register(emailNotifier)
```

## Supported Platforms

### Telegram

Features:
- Simple text messages
- Markdown/HTML formatting
- Photo messages
- Silent notifications (low priority)
- Custom parse modes

Configuration:
```go
config := notify.TelegramConfig{
    BotToken:   "YOUR_BOT_TOKEN",      // Required
    ChatID:     "YOUR_CHAT_ID",        // Required
    ParseMode:  "Markdown",            // Optional: Markdown, HTML, or empty
    HTTPClient: &http.Client{},        // Optional: Custom HTTP client
}
```

To get a bot token:
1. Talk to [@BotFather](https://t.me/botfather) on Telegram
2. Create a new bot with `/newbot`
3. Copy the token

To get your chat ID:
1. Send a message to your bot
2. Visit `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
3. Look for the `chat.id` field

### Slack

Features:
- Simple text messages
- Rich messages with blocks
- Attachments with fields
- File uploads
- Custom username and icon

Configuration:
```go
config := notify.SlackConfig{
    Token:          "xoxb-your-token",  // Required (Bot or User token)
    DefaultChannel: "#general",          // Required
    Username:       "NotifyBot",         // Optional
    IconEmoji:      ":robot_face:",      // Optional
}
```

To get a Slack token:
1. Go to [Slack API](https://api.slack.com/apps)
2. Create a new app or use an existing one
3. Add the Bot Token Scope: `chat:write`
4. Install the app to your workspace
5. Copy the Bot User OAuth Token

## API Reference

### Notifier Interface

All notification providers implement this interface:

```go
type Notifier interface {
    Send(ctx context.Context, message string) error
    SendWithOptions(ctx context.Context, msg *Message) error
    Name() string
}
```

### Message Structure

```go
type Message struct {
    Text        string        // Main message content
    Title       string        // Optional title
    Priority    string        // high, normal, low
    Channel     string        // Target channel (provider-specific)
    Attachments []Attachment  // Rich message attachments
    Metadata    map[string]interface{} // Provider-specific data
}
```

### Manager Methods

```go
// Register/Unregister
Register(notifier Notifier) error
Unregister(name string)
Get(name string) (Notifier, bool)
List() []string

// Send to specific provider
Send(ctx context.Context, provider, message string) error
SendWithOptions(ctx context.Context, provider string, msg *Message) error

// Broadcast to all providers
Broadcast(ctx context.Context, message string) []error
BroadcastWithOptions(ctx context.Context, msg *Message) []error
BroadcastAsync(ctx context.Context, message string) <-chan NotificationResult
BroadcastAsyncWithOptions(ctx context.Context, msg *Message) <-chan NotificationResult
```

## Examples

Check out the [examples](./examples) directory for complete working examples:

- [simple](./examples/simple) - Basic usage of Telegram and Slack
- [manager](./examples/manager) - Using the Manager for multiple providers
- [custom](./examples/custom) - Implementing custom notification providers

To run examples:

```bash
# Set environment variables
export TELEGRAM_BOT_TOKEN="your_token"
export TELEGRAM_CHAT_ID="your_chat_id"
export SLACK_BOT_TOKEN="your_token"
export SLACK_CHANNEL="#general"

# Run example
cd examples/simple
go run main.go
```

## Error Handling

All errors are wrapped in `NotificationError` which includes:
- Provider name
- Error message
- Underlying error (if any)

```go
err := notifier.Send(ctx, "Hello")
if err != nil {
    if notifErr, ok := err.(*notify.NotificationError); ok {
        fmt.Printf("Provider: %s\n", notifErr.Provider)
        fmt.Printf("Message: %s\n", notifErr.Message)
        fmt.Printf("Error: %v\n", notifErr.Unwrap())
    }
}
```

## Best Practices

1. **Use Context**: Always pass a context with timeout for production use:
   ```go
   ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
   defer cancel()
   ```

2. **Environment Variables**: Store tokens in environment variables, never in code:
   ```go
   token := os.Getenv("TELEGRAM_BOT_TOKEN")
   ```

3. **Error Handling**: Always check errors and handle them appropriately:
   ```go
   if err := notifier.Send(ctx, msg); err != nil {
       log.Printf("Failed to send notification: %v", err)
       // Implement retry logic or fallback
   }
   ```

4. **Async for Multiple Providers**: Use async broadcasting when sending to multiple providers:
   ```go
   resultChan := manager.BroadcastAsync(ctx, message)
   ```

## Testing

Run tests:

```bash
go test -v ./...
```

Run with coverage:

```bash
go test -v -cover ./...
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

### Adding a New Provider

1. Implement the `Notifier` interface
2. Add configuration struct
3. Add tests
4. Update documentation
5. Add example

## License

MIT License - see [LICENSE](LICENSE) file for details

## Roadmap

- [ ] Email provider (SMTP)
- [ ] Discord provider
- [ ] Microsoft Teams provider
- [ ] WhatsApp Business API provider
- [ ] SMS providers (Twilio, AWS SNS)
- [ ] Push notifications (FCM, APNS)
- [ ] Webhook provider
- [ ] Rate limiting
- [ ] Retry logic with exponential backoff
- [ ] Message templates
- [ ] Metrics and monitoring

## Support

If you have any questions or need help, please:
- Open an issue on GitHub
- Check existing issues for solutions
- Review the examples directory

## Acknowledgments

- [slack-go/slack](https://github.com/slack-go/slack) - Slack API client
- Telegram Bot API

---

Made with ❤️ by [milano15662](https://github.com/milano15662)
