# Intents

[[toc]]

## Introduction
Intent is a command which is been executed upon activation.
Upon running you can send messages, attachments and templates.

In order to create dialog and process user replies you need to use [interactions](interactions.md).

## Creating Intents

### Generating Intent Classes

All intents are stored in `app/Intents` directory.

Create new intent by running artisan command:

```bash
php artisan fondbot:make:intent OrderPizza
```

### Class Structure

Intents consint of two methods: `activators` and `run`.

`activators` should return an array of [activators](#activators). When user sends message FondBot will go through all your intents and find matching activator in order to decide which intent to execute.

When intent is matched, `run` method will be executed with the `MessageReceived` object

```php
<?php

declare(strict_types=1);

namespace App\Intents;

use FondBot\Conversation\Activators\Exact;
use FondBot\Conversation\Intent;
use FondBot\Events\MessageReceived;

class OrderPizza extends Intent
{
    /**
     * Intent activators.
     *
     * @return \FondBot\Contracts\Conversation\Activator[]
     */
    public function activators(): array
    {
        return [
            Exact::make('order pizza'),
        ];
    }

    public function run(MessageReceived $message): void
    {
        // Send reply to user, jump to interaction or do something else...
    }
}

```

## Making Conversations

Your intent may send a reply to user and stop executing, but mostly you will need to make a dialog with user in order to retrieve some information (for example, ask pizza type and size in order to make an order).

If you want to create dialog with user and process his reply you need to make a transition to an [interaction](interactions.md) from your intent:

```php
public function run(MessageReceived $message): void
{
    \App\Interactions\AskPizzaType::jump();
}
```

## Available Activators

### Exact

Received message text must match the given value.

```php
use FondBot\Conversation\Activators\Exact;

Exact::make('weather in New York');
```

### Contains

Received message text contains one or more values using [str_contains](https://laravel.com/docs/helpers#method-str-contains) helper.
Since this activator requires an array of values you may use `Activator` class in order to create activator instance.

```php
use FondBot\Conversation\Activators\Contains;

Contains::make(['weather in', 'is it raining in'])
```

### Regex

Received message text is matched by a regular expression using [str_is](https://laravel.com/docs/helpers#method-str-is).

```php
use FondBot\Conversation\Activators\Regex;

Regex::make(['weather in *', 'is it raining in *'])
```

### In

Received message text is one of given values using `in_array` function. 
The In Array activator checks if message text matches one of the values from the array.

Since this activator requires an array of values you may use `Activator` class in order to create activator instance.

```php
use FondBot\Conversation\Activators\In;

In::make(['first', 'second'])
```

### Attachment

Received message contains attachment.

```php
use FondBot\Conversation\Activators\Attachment;

// any attachment
Attachment::make();

// only image attachment
Attachment::make()->image();
```

### Payload

Received message payload data must match the given value.

```php
use FondBot\Conversation\Activators\Payload;

Payload::make('foo');
```

## Custom Activators

### Using Activator Objects

If you wish specific activators across your application, you can make your own activator. To generate a new activator object, use `fondbot:make:activator` artisan command:

```bash
php artisan fondbot:make:activator OnlyAdmin
```

The new activator will be placed in the `app/Activators` directory. Return `true` or `false` in `matches` method depending on whether incoming message is valid for activating intent or not:

```php
<?php

declare(strict_types=1);

namespace App\Activators;

use FondBot\Contracts\Conversation\Activator;
use FondBot\Events\MessageReceived;

class OnlyAdmin implements Activator
{
    private $adminUsernames = [
        'vladimir',
    ];

    /**
     * Result of matching activator.
     *
     * @param MessageReceived $message
     *
     * @return bool
     */
    public function matches(MessageReceived $message): bool
    {
        return in_array($message->getFrom()->getUsername(), $this->adminUsernames, true);
    }
}
```

### Using Closures

If you need custom activators once in some intent, you may use a Closure based activator. The Closure receives `MessageReceived` object and should return `true` or `false` depending on whether intent should be matched by the provided message.

```php
public function activators(): array
{
    return [
        function (MessageReceived $message) {
            return $message->getText() === 'foo';
        },
    ];
}
```