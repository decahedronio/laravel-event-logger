# Laravel Event Logger

## Quickstart

`composer require --dev "decahedron/laravel-event-logger"`

`php artisan vendor:publish --provider="Decahedron\LaravelEventLogger\EventLoggerServiceProvider"`

We use Laravel's [package discovery](https://laravel.com/docs/5.5/packages#package-discovery) 
mechanism. This shouldn't affect most use cases.

## How-to

It's incredibly useful to have logs throughout your code. But they can also look clunky and verbose:

```php
Log::debug("Resetting user password", ['user' => $user, 'token' => $token]);
$user->password = Str::random(64);
$user->save();
Log::info("User password reset", ['user' => $user]);
```

We can choose to define these as events instead, providing our application with handy future hooks for future expansion,
_and_ logging without the verbosity:

```php
event("user.service.password.resetting", ['user' => $user, 'token' => $token]);
$user->password = Str::random(64);
$user->save();
event("user.service.password.reset", ['user' => $user]);
```

Now we just have to define how we log these events.

### Defining Events in Logs

After running the `vendor:publish` command, a new directory called `event_logs` should be in your project root, on the
same level as routes or resources. A default entry file called `event_logs.php` is where you can define your event logs.

```php
EventLog::register([
    'user.service.password.resetting' => 'Resetting user password',
    'user.service.password.reset' => 'info: User password reset',
]);
``` 

Note the `info:` prefix which we use to indicate the log level; it will be removed before sending. Only log levels 
accepted by Monolog are allowed.

#### Duplicates and Wildcards

Since we're just using laravel's event listeners, we can use wildcards:

```php
EventLog::register([
    '*.password.reset' => 'info: A password has been reset',
    'user.service.password.reset' => 'info: User password reset',
]);
```

Multiple keys matching the same event will all fire.

#### Closure Logs

If you need to customize the log event, yet you _really_ don't want to make a proper event listener:

```php
EventLog::register([
    'user.service.password.reset' => function($logger, $event, $data) {
        $logger->to('security')->notice("User password has been reset", $data + ['_event' => $event]);
    },
]);
```

### Extra Configs

See `config/event-logs.php` to change some common event logging settings.

- When an event is sent to the logs, its original event name is sent along it as `event_name_context`. This is handy
for searching and analytics.
- `channel` defines the logging channel (new in L5.6) events should be logged to.
- `log_all` determines if events not registered in `event_logs.php` will still be logged.
- `default_log_level` is the level logs are sent as, if not prefixed like `error:`.

Bear in mind that logs simply use Laravel's logging system, so for advanced features, like adding more information to
log events, can be tweaked from Laravel and Monolog directly.