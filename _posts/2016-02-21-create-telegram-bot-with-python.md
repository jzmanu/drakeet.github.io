---
title: 使用 Python 创建 Telegram 机器人
---

要说聊天体验，telegram 比微信好很多，微信消息一多就卡，而且没法直接引用消息进行回复导致经常找不到上下文。telegram 则始终异常流畅，记录阅读位置，消息多的时候也不会觉得跟不上聊天节奏或卡 ... 当然了，这一段都不是今天要写的这篇文章的重点，这篇文章主要还是要介绍一下如何开发 telegram 机器人，telegram 在去年中旬的时候开放了机器人的 API，可以设置 hook，使得所有消息都能被转发到你的服务上，然后作出自动化回复。

机器人可以被邀请入群做很多辅助工作，比如输入 `/google xxx` 就可以得到谷歌查询的结果等等，甚至还有人开发了 `/fff` 命令，用来烧死异性恋😂什么的，只要发送一条 `/fff install someone` 的消息到机器人所在的群聊当中即可。总之，利用 bot API 几乎无所不能，也十分有意思。

俺从几天前开始玩耍 telegram bot API，开发了一个属于自己的机器人，<a href="http://telegram.me/XiaoaiBot" target="_blank">@XiaoaiBot</a>，可以回声、计算两个日期的间隔时间、以及帮用户在群里找出最近一个 @ 消息，应该说还是非常好玩的。

<h4>@BotFather</h4>
一开始得在 telegram 中添加一个“机器人之父”的账号，便是 <a href="http://telegram.me/BotFather" target="_blank">@BotFather</a> 这个账号，然后给它发送 `/newbot` 命令，逐步建立起一个机器人，包括头像、介绍，以及它可以支持的命令，另外，最重要的就是能够得到这个机器人的 token，通过这个 token 可以调用官方机器人 API 收发消息。
<h4>token 的使用</h4>
telegram bot API 的官方文档是：<a href="https://core.telegram.org/bots/api" target="_blank">https://core.telegram.org/bots/api</a>，在这里可以看到，所有的接口都是基于 <a href="https://api.telegram.org/bot+你的token" target="_blank">https://api.telegram.org/bot+你的token</a> 这个 base url，比如你的 token 是 `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`，那么你调用任何 API 都得基于 https://api.telegram.org/bot123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11/，后面再加上具体 API 名。

<h4>设置消息 hook</h4>
telegram 支持两种获得消息的方式，一种是开发中利用 `getUpdates` 接口主动去查询是否有新消息，另外一种就是推荐的 `setWebhook`，设置一个 `https` 的服务地址，完事之后，所有机器人收到的消息都会同时发往这个地址，并携带上 json 数据，使用 python 的时候，可以通过 `flask` 这个框架的 `request.get_json(force=True)` 来获取到 json 数据。

设置 hook 的示例 API url 如下：

https://api.telegram.org/bot你的token/setWebhook?url=https://xxx.com
<h4>使用 python-telegram-bot 库进行开发</h4>
GitHub 上有一个别人已经封装好了的 `python-telegram-bot` 库，可以帮助开发者更近轻松快速地使用官方 API：<a href="https://github.com/python-telegram-bot/python-telegram-bot" target="_blank">https://github.com/python-telegram-bot/python-telegram-bot</a>

将它装载下来后，你可以通过以下代码进行初始化：

```python
import telegram
bot = telegram.Bot(token='你的token')
```

然后就可以通过这个 bot 对象发送各种消息了，更多使用内容可以参看其开源代码的 readme，或者官方文档：<a href="http://python-telegram-bot.readthedocs.org/en/latest/py-modindex.html" target="_blank">http://python-telegram-bot.readthedocs.org/en/latest/py-modindex.html</a>

<h4>我的 @XiaoaiBot 项目</h4>
有了各种准备内容后，我们还需要 `flask` 框架来提供 `http` 服务响应，`flask` 我就不多说了，应该学过 python 的都会懂得使用这个框架。主要说一下我开发小爱 bot 的一些思路和代码。

我写了一个 `launcher` 和 `handle_message` 方法用来分发命令到具体到执行代码：

```python
@app.route('/<token>', methods=['POST'])
def launcher(token):
    if request.method == "POST":
        update = telegram.Update.de_json(request.get_json(force=True))
        handle_message(update.message)
    return 'ok'

def handle_message(message):
    text = message.text
    if '/echo' in text:
        echo(message)
    elif '/milestone' in text:
        milestone(message)
    elif '/help' in text:
        help(message)
    elif '/getmylastat' in text:
        get_my_last_at(message)
    elif '/pic' in text:
        pic(message)
    elif '/delpic' in text:
        delpic(message)

    if not '/' in text and '@' in text:
        save_at_message(message)
```

这个方法应该一目了然，不多说，主要作为分发命令消息到入口。

另外我还写了一个用来分离命令和附着文本的方法：

```python
def parse_cmd_text(text):
    # Telegram understands UTF-8, so encode text for unicode compatibility
    text = text.encode('utf-8')
    cmd = None
    if '/' in text:
        try:
            index = text.index(' ')
        except ValueError as e:
            return (text, None)
        cmd = text[:index]
        text = text[index + 1:]
    if not cmd == None and '@' in cmd:
        cmd = cmd.replace(bot_name, '')
    return (cmd, text)
```

其中需要注意到是，有可能用户发送到命令是：`/xxx@XiaoaiBot 123` 这样，就得把@XiaoaiBot 这个无用的内容去掉，我使用 `replace` 方法将机器人的名字替换为空字符，这样就相当于删掉这个内容了。另外，也可以使用 python 的 re 正则表达式来匹配得到同样的结果，不多说。

整个开发过程还是非常愉快和轻松的，我使用的是 leancloud 美国节点进行部署，其中需要注意的是，其美国节点不支持命令行部署，只能 git 部署，于是我不得不开源我的 token，开源代码是：

<a href="https://github.com/drakeet/DrakeetLoveBot" target="_blank">https://github.com/drakeet/DrakeetLoveBot</a>

希望大家如果 fork 我的代码使用，不要使用我的 token，因为会造成我的服务有可能出现混乱，我也会伤心以后不敢再开源了之类的，而且，申请一个 token 是非常容易的，完全没必要使用我的，祝玩得开心，也欢迎来玩耍俺的机器人 <a href="http://telegram.me/XiaoaiBot" target="_blank">@XiaoaiBot</a>，提供的功能如下：

- `/echo` - Repeat the same message back
- `/milestone` - Get drakeet's milestone
- `/getmylastat` - Get my last AT message
- `/pic`
- `/delpic`
- `/songci` - 获取一篇宋词
