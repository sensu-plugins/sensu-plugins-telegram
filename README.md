## Sensu-Plugins-telegram

[![Build Status](https://travis-ci.org/sensu-plugins/sensu-plugins-telegram.svg?branch=master)](https://travis-ci.org/sensu-plugins/sensu-plugins-telegram)
[![Gem Version](https://badge.fury.io/rb/sensu-plugins-telegram.svg)](http://badge.fury.io/rb/sensu-plugins-telegram)

## Functionality

This plugin provides a handler to send notifications to a Telegram chat.

## Files
 * bin/handler-telegram.rb

## Usage

After installation, you have to set up a `pipe` type handler, like so:

```json
{
  "handlers": {
    "telegram": {
      "type": "pipe",
      "command": "handler-telegram.rb",
      "filter": "occurrences"
    }
  }
}
```

> Note: You need the `filter` key in there if you use the `occurrences` and/or the `refresh`
attributes in your [check definitions](https://docs.sensu.io/sensu-core/1.6/reference/checks/).

This gem also expects a JSON configuration file with the following contents:

```json
{
  "telegram": {
    "bot_token": "YOUR_BOT_TOKEN",
    "chat_id": -123123,
    "error_file_location": "/tmp/telegram_handler_error"
  }
}
```

### Parameters:
- `bot_token`: your bot's token, as provided by
   [@BotFather](https://telegram.me/botfather).
- `chat_id`: the chat to which the error message is to be sent.
  The bot must be a member of this channel or group.
  You can get the `chat_id` by adding the bot to the corresponding group,
  sending a message like `/fakecmd @your-bot-username`, and then accessing
  `https://api.telegram.org/bot<TOKEN>/getUpdates`.
- `error_file_location` (optional): in case there is a failure sending the
  message to Telegram (ie. connectivity issues), the exception mesage will
  be written to a file in this location. You can then monitor this
  location to detect any errors with the Telegram handler.
- `message_template` (optional): An ERB template to use to format messages
  instead of the default. Supports the following variables:
  - `action_name`
  - `action_icon`
  - `client_name`
  - `check_name`
  - `status`
  - `status_icon`
  - `output`
- `message_template_file` (optional): A file to read an ERB template from to
  format messages. Supports the same variables as `message_template`.

### Check configuration

You can then set up your checks to use the handler like this:

```
{
  "checks": {
    "sensu-website": {
      "command": "check-http.rb -u https://sensuapp.org",
      "subscribers": ["production"],
      "interval": 60,
      "handler": "telegram"
    }
  }
}
```

For more information about configuring checks, see the
[Sensu documentation](https://docs.sensu.io/sensu-core/1.6/reference/checks).

### Advanced configuration

By default, the handler assumes that the config parameters are specified in the
`telegram` top-level key of the JSON, as shown above. You also have the option
to make the handler fetch the config from a different key. To do this, pass the
`-j` option to the handler with the name of the desired key You can define
multiple handlers, and each handler can send notifications to a different chat
and from a different bot. You could, for example, have critical and non-critical
Telegram groups, and send the notifications to one or the other depending on the
check. For example:

```json
{
  "handlers": {
    "critical_telegram": {
      "type": "pipe",
      "command": "handler-telegram.rb -j critical_telegram_options"
    },
    "non_critical_telegram": {
      "type": "pipe",
      "command": "handler-telegram.rb -j non_critical_telegram_options"
    }
  }
}
```

This example will fetch the options from a JSON like this:

```json
{
  "telegram": {
    "bot_token": "YOUR_BOT_TOKEN"
  },
  "critical_telegram_options": {
    "chat_id": -123123
  },
  "non_critical_telegram_options": {
    "chat_id": -456456
  }
}
```

As you can see, you can specify the default config in the `telegram` key, and
the rest of the config in their own custom keys.

You can also directly add the configuration parameters to the event data using a
mutator. For example:

```ruby
#!/usr/bin/env ruby
require 'rubygems'
require 'json'
event = JSON.parse(STDIN.read, :symbolize_names => true)
event.merge!(chat_id: -456456)
puts JSON.dump(event)
```

### Configuration precedence

The handler will load the config as follows (from least to most priority):

* Default `telegram` key
* Custom config keys
* Event data

## Installation

[Installation and Setup](http://sensu-plugins.io/docs/installation_instructions.html)

## Notes
