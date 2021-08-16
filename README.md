# Bindings bằng ngôn ngữ Go cho Bot API của KingTalk


## Install

Đầu tiên, đảm bảo library đã được cài đặt và cập nhận phiên bản mới nhất:

`go get -u github.com/nvhai245/kingtalk-bot-api`.


## Example

Ví dụ dưới đây là một bot đơn giản thực hiện lấy về các updates mới nhất rồi đăng chúng vào trong nhóm chat hiện tại:

```go
package main

import (
	"log"

	
)

func main() {
	bot, err := ktbotapi.NewBotAPI("Token_Của_Bot")
	if err != nil {
		log.Panic(err)
	}

	bot.Debug = true

	log.Printf("Authorized on account %s", bot.Self.UserName)

	u := ktbotapi.NewUpdate(0)
	u.Timeout = 60

	updates, err := bot.GetUpdatesChan(u)

	for update := range updates {
		if update.Message == nil { // ignore any non-Message Updates
			continue
		}

		log.Printf("[%s] %s", update.Message.From.UserName, update.Message.Text)

		msg := ktbotapi.NewMessage(update.Message.Chat.ID, update.Message.Text)
		msg.ReplyToMessageID = update.Message.MessageID

		bot.Send(msg)
	}
}
```

Xem nhiều hơn các ví dụ tại [wiki](https://github.com/nvhai245/kingtalk-bot-api/wiki)
với nhiều thông tin chi tiết hơn vè cách sử dụng api, commands và các reply markup.

Dưới đây là một ví dụ cho bot muốn sử dụng webhook của Google App Engine.

```go
package main

import (
	"log"
	"net/http"

	
)

func main() {
	bot, err := ktbotapi.NewBotAPI("Token_Của_Bot")
	if err != nil {
		log.Fatal(err)
	}

	bot.Debug = true

	log.Printf("Authorized on account %s", bot.Self.UserName)

	_, err = bot.SetWebhook(ktbotapi.NewWebhookWithCert("https://www.google.com:8443/"+bot.Token, "cert.pem"))
	if err != nil {
		log.Fatal(err)
	}
	info, err := bot.GetWebhookInfo()
	if err != nil {
		log.Fatal(err)
	}
	if info.LastErrorDate != 0 {
		log.Printf("KingTalk callback failed: %s", info.LastErrorMessage)
	}
	updates := bot.ListenForWebhook("/" + bot.Token)
	go http.ListenAndServeTLS("0.0.0.0:8443", "cert.pem", "key.pem", nil)

	for update := range updates {
		log.Printf("%+v\n", update)
	}
}
```
