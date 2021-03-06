# TamTam Bot SDK <sup><span title="Alpha version. Use in production at your own risk">𝛂</span></sup>

[![build status](https://github.com/tamtam-chat/tamtam-bot-sdk/workflows/Build/badge.svg?branch=master)](https://github.com/tamtam-chat/tamtam-bot-sdk/actions?query=workflow%3A%22Build%22)

TamTam Bot SDK is a simple library built on top of [tamtam-bot-api](https://github.com/tamtam-chat/tamtam-bot-api) that
helps you to develop bots for [TamTam](https://tamtam.chat) quickly.

## Usage

Add the following dependency to your project:

Maven:
```xml
<dependency>
    <groupId>chat.tamtam</groupId>
    <artifactId>tamtam-bot-sdk</artifactId>
    <version>0.0.1</version>
</dependency>
```

Gradle:
```
implementation group: 'chat.tamtam', name: 'tamtam-bot-sdk', version: '0.0.1'
```

Then you should choose the way how your bot will receive notifications: long-polling or webhook.

### Long-polling

Long-polling is the easiest way to receive updates for your bot because it does not require running web server.

To start your bot just extend [`LongPollingBot`](src/main/java/chat/tamtam/bot/longpolling/LongPollingBot.java) class and add methods annotated by [`@UpdateHandler`](src/main/java/chat/tamtam/bot/annotations/UpdateHandler.java) annotation.

These methods should have **only** one parameter of type [`Update`](https://github.com/tamtam-chat/tamtam-bot-api/blob/master/src/main/java/chat/tamtam/botapi/model/Update.java). Every method will handle update of such type.

For example, simple bot that just replies on incoming message:

```java
public class ReplyBot extends LongPollingBot {
    public ReplyBot(String accessToken) {
        super(accessToken);
    }

    @UpdateHandler
    public Object onMessageCreated(MessageCreatedUpdate update) {
        Message message = update.getMessage();
        return NewMessageBodyBuilder.ofText("Reply on: " + message.getBody()).build(); // return null if you do not want to reply synchronously
    }
}
```

All other updates will be ignored. If you want to handle every update just override `onUpdate` method of [`TamTamBotBase`](src/main/java/chat/tamtam/bot/TamTamBotBase.java).

Alternatively, you can directly create instance of `LongPollingBot` and pass handlers to constructor:
```java
// handler can be any object that has methods annotated with `@UpdateHandler`
LongPollingBot bot = new LongPollingBot("%ACCESS_TOKEN%", handler1, handler2);
```

As soon as you created instance of bot you must `start` it:

```java
ReplyBot bot = new ReplyBot("%ACCESS_TOKEN%");
bot.start();
```
This method starts separated *non-daemon* thread that polls Bot API in cycle.
Call `stop` as soon as you ready to shutdown it:

```java
bot.stop();
```

Check out [EchoBot](examples/longpolling-echobot/src/main/java/chat/tamtam/echobot/Main.java) for more complete example.

### Webhooks

Webhook subscribed bot requires running HTTP server. By default we use [Jetty](https://www.eclipse.org/jetty/)
but you can use any server you want.

All webhook bots should be put in container that manages server and handle all incoming HTTP-requests to bots:

```java
WebhookEchoBot bot1 = new WebhookEchoBot("%ACCESS_TOKEN%");
WebhookBot bot2 = new WebhookBot("%ANOTHER_ACCESS_TOKEN%");

JettyWebhookBotContainer botContainer = new JettyWebhookBotContainer("mysupercoolbot.com", 8080);
botContainer.register(bot1);
botContainer.register(bot2);

// Register JVM shutdown hook to stop our server
Runtime.getRuntime().addShutdownHook(new Thread(botContainer::stop));

// `start` will run underlying Jetty server and register webhook subscription for each bot
botContainer.start();
```

Check out [jetty-webhook-echobot](examples/jetty-webhook-echobot/src/main/java/chat/tamtam/echobot/WebhookEchoBot.java)
for more complete example or [tomcat-webhook-echobot](examples/tomcat-webhook-echobot/src/main/java/chat/tamtam/echobot/WebhookEchoBot.java)
as an example of alternative container implementation.

## Requirements

Minimum required version of Java is 8.

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

This project is licensed under the [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0).
