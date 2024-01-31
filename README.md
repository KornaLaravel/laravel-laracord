# Laracord

Laracord is a micro-framework that provides a powerful starting point for your next Discord bot.

Build functional, elegant bots harnessing the full power of [Laravel](https://laravel.com/) alongside [DiscordPHP](https://github.com/discord-php/DiscordPHP).

![Screenshot](https://i.imgur.com/yW2vpPR.png)

## Requirements

Make sure all dependencies have been installed before moving on:

- [PHP](https://secure.php.net/manual/en/install.php) >= 8.1
- [Composer](https://getcomposer.org/download/)

## Getting Started

Start by creating a Laracord project:

```sh
composer create-project laracord/laracord:dev-main
cd laracord
```

In order to run the Bot, you will have to create a Application using the [Discord Developer Portal](https://discord.com/developers/applications) and obtain a **bot token**.

By default, the Bot will also require you enable the necessary [Intents](https://discord.com/developers/docs/topics/gateway#gateway-intents). These can be found and enabled in the developer portal under `Settings > Bot > Privileged Gateway Intents`.

Once you obtain a token, you can configure it inside of `.env` or `config/discord.php`.

## Usage

Out of the box, Laracord includes a simple `!ping` command to test your bot with a simple response.

### Creating Commands

Creating your own command is simple and can be generated using the `laracord` binary:

```sh
php laracord make:command Example
```

A generated command in it's simplest form will look something like:

```php
<?php

namespace App\Commands;

use Laracord\Commands\Command;

class Example extends Command
{
    /**
     * The command name.
     *
     * @var string
     */
    protected $name = 'example';

    /**
     * The command description.
     *
     * @var string
     */
    protected $description = 'The command description.';

    /**
     * Handle the command.
     *
     * @param  \Discord\Parts\Channel\Message  $message
     * @param  array  $args
     * @return void
     */
    public function handle($message, $args)
    {
        return $this
            ->message()
            ->title('Example')
            ->content('Hello world!')
            ->send($message);
    }
}
```

### Creating Slash Commands

Laracord has full support for slash commands out of the box. When a command is created, updated, or removed – the Discord application command repository is automatically updated.

Similar to creating a normal command, creating a slash command can be done using the `laracord` binary:

```sh
php laracord make:slash-command Example
```

A generated slash command in it's simplest form will look something like:

```php
<?php

namespace App\SlashCommands;

use Laracord\Commands\SlashCommand;

class Example extends SlashCommand
{
    /**
     * The slash command name.
     *
     * @var string
     */
    protected $name = 'example';

    /**
     * The slash command description.
     *
     * @var string
     */
    protected $description = 'The example slash command.';

    /**
     * Handle the slash command.
     *
     * @param  \Discord\Parts\Interactions\Interaction  $interaction
     * @return void
     */
    public function handle($interaction)
    {
        $interaction->respondWithMessage(
            $this
              ->message()
              ->title('Example')
              ->content('Hello world!')
              ->build()
        );
    }
}
```

To only register a slash command to a specific guild/server, you may set the `$guild` property:

```php
/**
 * The guild the command belongs to.
 *
 * @var string
 */
protected $guild = 'your-guild-id';
```

Adding options to your slash command can be done by passing an array of options to the `$options` property **or** overriding the `options()` method. Options can be specified as a raw array or by passing an array of [DiscordPHP Option Parts](https://github.com/discord-php/DiscordPHP/wiki/Option_commands).

```php
use Discord\Parts\Interactions\Command\Option;

/**
 * The slash command options.
 *
 * @var array
 */
protected $options = [
    [
        'name' => 'message',
        'description' => 'Send a message through the bot.',
        'type' => Option::STRING,
        'required' => true,
    ],
];
```

```php
use Discord\Parts\Interactions\Command\Option;

/**
 * Set the command options.
 *
 * @return array
 */
public function options()
{
    $option = new Option($this->discord());

    return [
        $option
          ->setName('message')
          ->setDescription('Send a message through the bot')
          ->setType(Option::STRING)
          ->setRequired(true)
          ->toArray(),
    ];
}
```

### Creating Events

Laracord makes creating handlers for specific Discord events extremely straight forward. Creating an event handler can be done using the `laracord` binary:

```sh
php laracord make:event Example
```

When creating an event, you will be prompted to choose a listener from a searchable select box.

Once you select a listener, an Event class will automatically be generated including the necessary namespaces/typing specific to the handler.

Here is an example of a simple `MESSAGE_CREATE` event handler that logs a message to console everytime a message is created:

```php
<?php

namespace App\Events;

use Discord\Discord;
use Discord\Parts\Channel\Message;
use Discord\WebSockets\Event as Events;
use Laracord\Events\Event;

class Example extends Event
{
    /**
     * The event handler.
     *
     * @var string
     */
    protected $handler = Events::MESSAGE_CREATE;

    /**
     * Handle the event.
     */
    public function handle(Message $message, Discord $discord)
    {
        $this->console()->log('The Message Create event has fired!');
    }
}
```

### Creating Services

Services are asynchronous tasks that run parallel to the Discord bot on an interval. This is useful for scenarios such as fetching data from an API every so often and sending the results to Discord through the bot.

Services created in your application are automatically booted alongside your bot with the Discord instance being easily accessible through `discord()`.

To create a service, run the `make:service` command:

```sh
php laracord make:service Example
```

The default generated service will create a task that logs `Hello World` to console every 5 seconds.

```php
<?php

namespace App\Services;

use Laracord\Services\Service;

class Example extends Service
{
    /**
     * The service interval.
     */
    protected int $interval = 5;

    /**
     * Handle the service.
     */
    public function handle(): void
    {
        $channel = $this->discord()->getChannel('your-channel-id');

        $this
            ->message()
            ->content('Hello world.')
            ->send($channel);

        $this->console()->log('Hello world.');
    }
}
```

Sending a message to a Discord channel through a service can be done similarily to commands. The exception is you must get an instance of the channel/user to send the message to. This can be done through accessing the Discord instance:

```php
$channel = $this->discord()->getChannel('your-channel-id');

$this
    ->message()
    ->content('Hello world.')
    ->send($channel);

$user = $this->discord()->users->get('id', 'a-user-id');

$this
    ->message()
    ->content('Hello world.')
    ->sendTo($user);
```

### Booting Laracord

To boot your bot, simply run `laracord` with no arguments passed:

```sh
php laracord
```

### Adding a Bot Admin

For convenience, Laracord comes with a console command to set a specified user as admin using their Discord ID:

```sh
php laracord bot:admin <id>
```

### Additional Commands

To view all built-in console commands, you can run the `list` command:

```sh
php laracord list
```

### Building for Production

To build your bot for production, simply run the `app:build` command:

```sh
php laracord app:build
```

Documentation for `app:build` can be found in the [Laravel Zero docs](https://laravel-zero.com/docs/build-a-standalone-application).

## Documentation

Laracord currently lacks it's own documentation, but you can find the documentation of the packages/frameworks it utilizes below to learn more in the mean time:

- [DiscordPHP](https://discord-php.github.io/DiscordPHP/)
- [Laravel Zero](https://laravel-zero.com/docs)
- [Laravel](https://laravel.com/docs)

You may also be interested in the underlying [Laracord Framework](https://github.com/laracord/framework) source code to get a better understanding of existing functionality until proper documentation is written.

## Bug Reports

If you discover a bug in Laracord, please [open an issue](https://github.com/laracord/laracord/issues).

## Contributing

Contributing whether it be through PRs, reporting an issue, or suggesting an idea is encouraged and appreciated.

## License

Laracord is provided under the [MIT License](LICENSE.md).
