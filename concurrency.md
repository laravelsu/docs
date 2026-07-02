---
git: 5f2819af590361b436498883e9ef98f73fb2c5b0
---

# Параллелизм

<a name="introduction"></a>
## Введение

Иногда вам может потребоваться выполнить несколько медленных задач, не зависящих друг от друга. Во многих случаях существенного повышения производительности можно добиться, выполняя задачи одновременно. Фасад Laravel `Concurrency` предоставляет простой и удобный API для одновременного выполнения замыканий.

<a name="how-it-works"></a>
#### Как это работает

Laravel обеспечивает параллелизм путем сериализации заданных замыканий и отправки их скрытой команде Artisan CLI, которая десериализует замыкания и вызывает их в собственном PHP-процессе. После вызова замыкания полученное значение сериализуется обратно в родительский процесс.

Фасад `Concurrency` поддерживает три драйвера: `process` (по умолчанию), `fork` и `sync`.

Драйвер `fork` обеспечивает улучшенную производительность по сравнению с драйвером по умолчанию `process`, но его можно использовать только в контексте CLI PHP, поскольку PHP не поддерживает разветвление во время веб-запросов. Перед использованием драйвера `fork` вам необходимо установить пакет `spatie/fork`:

```shell
composer require spatie/fork
```

Драйвер `sync` в первую очередь полезен во время тестирования, когда вы хотите отключить весь параллелизм и просто последовательно выполнить заданные замыкания внутри родительского процесса.

<a name="running-concurrent-tasks"></a>
## Запуск параллельных задач

Для запуска параллельных задач вы можете вызвать метод `run` фасада `Concurrency`. Метод `run` принимает массив замыканий, которые должны выполняться одновременно в дочерних процессах PHP:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
]);
```

Чтобы использовать конкретный драйвер, вы можете использовать метод `driver`:

```php
$results = Concurrency::driver('fork')->run(...);
```

Или, чтобы изменить драйвер параллелизма по умолчанию, вам следует опубликовать файл конфигурации `concurrency` с помощью Artisan-команды `config:publish` и обновить параметр `default` в файле:

```shell
php artisan config:publish concurrency
```

<a name="named-results"></a>
### Именованные результаты

Если вы хотите обращаться к результатам параллельных задач по имени, а не по позиции, вы можете передать ассоциативный массив замыканий. Каждый результат будет возвращен с тем же ключом, что и соответствующее замыкание:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

$results = Concurrency::run([
    'users' => fn () => DB::table('users')->count(),
    'orders' => fn () => DB::table('orders')->count(),
]);

$userCount = $results['users'];
$orderCount = $results['orders'];
```

<a name="task-timeouts"></a>
### Тайм-ауты задач

При использовании драйвера `process` (по умолчанию) вы можете указать максимальное количество секунд, в течение которых параллельной задаче разрешено выполняться до принудительного завершения, передав тайм-аут методу `run`:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
], timeout: 30);
```

Вы также можете передать экземпляр `CarbonInterval`, если предпочитаете более выразительное определение тайм-аута:

```php
use Illuminate\Support\Facades\Concurrency;

use function Illuminate\Support\seconds;

Concurrency::run([...], timeout: seconds(30));
```

<a name="deferring-concurrent-tasks"></a>
## Отсрочка параллельных задач

Если вы хотите одновременно выполнить массив замыканий, но вас не интересуют результаты, возвращаемые этими замыканиями, вам следует рассмотреть возможность использования метода `defer`. Когда вызывается метод `defer`, данные замыкания не выполняются немедленно. Вместо этого Laravel будет выполнять замыкания одновременно после отправки пользователю HTTP-ответа:

```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```
