---
layout: post
title:  "Telegram Notification on Laravel Error"
date:   2020-07-09 08:24:48 +0200
categories: laravel
---

I'm working on an important project and I have the need to keep track of errors in real time.

I thought to myself **How can I track Laravel error in realtime?** and the best option I came up with is to create a Telegram Bot, that every time a production error happens it sends me the first part of the error stack along with the user who fired it, his whatsapp number (so that I can click on the person's name and start a chat directly) and the exact url it happened at.

And this is the result:

![image](/images/tg-bot-error.png)
<br><br><br>
To do this I installed the package [laravel-notification-channels](https://github.com/laravel-notification-channels/telegram), created a TelegramBot, set the API in the `.env` (as explained in the package installation) and my `chat_id` inside `app\User`

```php
public function routeNotificationForTelegram()
{
    return 'my-numeric-chat-id';
}
```

> As a note: Telegram chat_id is the same for every Bot and chat you create, is actually your very Telegram ID which will stay the same unless you create another account.

<br><br>
Then I've extended the `report()` method inside `app/Exceptions/Handler.php` of my Project to look like this:

```php
public function report(Throwable $exception)
{
    parent::report($exception);
    if (app()->isProduction() && auth()->id()) {
        $cut_exception = explode("\n", $exception)[0];

        $app         = config('app.name');
        $user        = auth()->id();
        $name        = auth()->user()->full_name;
        $phone       = ltrim(str_replace(' ', '', auth()->user()->phone), '0');
        $current_url = url()->current();

        $mrweb = User::find(1)
            ->notify(new TelegramNotification("*{$app}*\nUser #{$user} [{$name}](https://wa.me/{$phone})\n\n{$current_url}\n\n`{$cut_exception}`"));
    }
}
```