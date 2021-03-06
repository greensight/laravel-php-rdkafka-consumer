# Laravel PHP Rdkafka Consumer

`Deprecated, use https://github.com/ensi-platform/laravel-php-rdkafka-consumer instead`

Opiniated High Level consumer for [greensight/laravel-phprdkafka](https://github.com/greensight/laravel-php-rdkafka)

## Installation

Firstly, you have to install and configure [greensight/laravel-phprdkafka](https://github.com/greensight/laravel-php-rdkafka)

Then,
```bash
composer require greensight/laravel-phprdkafka-consumer
```

Publish the config file with:
```bash
php artisan vendor:publish --provider="Greensight\LaravelPhpRdKafkaConsumer\LaravelPhpRdKafkaConsumerServiceProvider" --tag="kafka-consumer-config"
```

Now go to `config/kafka-consumer.php` and add processors there.

## Usage

The package provides `php artisan kafka:consume {topic} {consumer=default}` command that executes the first processor that matches given topic and consumer name. Consumer name is taken from greensight/laravel-phprdkafka config file.

Processors in config have the following configuration options:

```php
[
   /*
   | Optional, defaults to `null`.
   | Here you may specify which topic should be handled by this processor.
   | Processor handles all topics by default.
   */
   'topic' => 'stage.crm.fact.registrations.1',

   /*
   | Optional, defaults to `null`.
   | Here you may specify which greensight/laravel-phprdkafka consumer should be handled by this processor.
   | Processor handles all consumers by default.
   */
   'consumer' => 'default',

   /*
   | Optional, defaults to `action`.
   | Here you may specify processor's type. Defaults to `action`
   | Supported types:
   |  - `action` - a simple class with execute method;
   |  - `job` - Laravel Queue Job. It will be dispatched using `dispatch` or `dispatchSync` method;
   */
   'type' => 'action',

   /*
   | Required.
   | Fully qualified class name of a processor class.
   */
   'class' => \App\Domain\Communication\Actions\SendConfirmationEmailAction::class,
   
   /*
   | Optional, defaults to `false`.
   | Proxy messages to Laravel's queue.
   | Supported values:
   |  - `false` - do not stream message. Execute processor in syncronous mode;
   |  - `true` - stream message to Laravel's default queue;
   |  - `<your-favorite-queue-name-as-string>` - stream message to this queue;
   */
   'queue' => false,

   /*
   | Optional, defaults to 5000.
   | Kafka consume timeout in milliseconds .
   */
   'consume_timeout' => 5000,
]

```
### Synchronous processors

Most of the time all tou need is a synchronous processor.
A simple example of such processor:

```php
class SendConfirmationEmailAction
{
   public function execute(Message $message): void
   {
      // var_dump($message->payload);
   }
}
```

### Queueable processors

If you want to stream message to Laravel's own queue you can use [spatie/laravel-queueable-action](https://github.com/spatie/laravel-queueable-action)  

If for some reason you don't want to rely on that package you can swich to [Laravel Jobs](https://laravel.com/docs/master/queues#class-structure)  

In both cases you also need to specify `'queue' => true` or `'queue' => 'my-favorite-queue'` in the package's config for a given processor.  

Processor using Laravel Job example:

```php

use RdKafka\Message;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;

class ConsumeMessageJob implements ShouldQueue
{
   use Dispatchable, InteractsWithQueue, Queueable;

   public function __construct(protected Message $message)
   {
   }

   public function handle(): void
   {
      // var_dump($this->message->payload);
   }
}

```

## Testing

```bash
composer test
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
