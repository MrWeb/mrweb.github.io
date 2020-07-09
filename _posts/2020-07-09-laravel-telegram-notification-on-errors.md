---
layout: post
title:  "Laravel Telegram Notification on Error"
date:   2020-07-09 08:24:48 +0100
categories: php laravel
---

I'm working on an important project and I had the need to keep track of errors in real time.

The best option I found is to create a Telegram Bot, that every time a production error happens it sends me the first part of the error stack along with the user who fired it, his whatsapp number (so that I can click and start chat directly in case I need to) and the exact url it happened at.

![image](/assets/tg-bot-error.png)

To do this I installed the package [laravel-notification-channels](https://github.com/laravel-notification-channels/telegram), created a TelegramBot, set the API in the `.env` (as explained in the package installation) and my `chat_id` inside `app\User`

```php
public function routeNotificationForTelegram()
{
    return 'my-numeric-chat-id';
}
```

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