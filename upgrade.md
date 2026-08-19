---
git: 080853d694311a6d78ae0b33995f74abf9f355e5
---

# Руководство по обновлению

- [Обновление с 12.x до 13.0](#upgrade-13.0)
    - [Обновление с помощью AI](#upgrading-using-ai)

<a name="high-impact-changes"></a>
## Изменения с высоким уровнем влияния

<!-- <div class="content-list" markdown="1"> -->

- [Обновление зависимостей](#updating-dependencies)
- [Обновление Laravel Installer](#updating-the-laravel-installer)
- [Защита от подделки запросов](#request-forgery-protection)

<!-- </div> -->

<a name="medium-impact-changes"></a>
## Изменения со средним уровнем влияния

<!-- <div class="content-list" markdown="1"> -->

- [Конфигурация cache `serializable_classes`](#cache-serializable_classes-configuration)
- [`upsert` базы данных с MySQL или MariaDB](#database-upsert-mariadb-mysql)

<!-- </div> -->

<a name="low-impact-changes"></a>
## Изменения с низким уровнем влияния

<!-- <div class="content-list" markdown="1"> -->

- [Префиксы кеша и имена session cookies](#cache-prefixes-and-session-cookie-names)
- [Сериализация коллекций моделей восстанавливает жадно загруженные связи](#collection-model-serialization-restores-eager-loaded-relations)
- [`Container::call` и значения nullable-классов по умолчанию](#containercall-and-nullable-class-defaults)
- [Приоритет регистрации доменных маршрутов](#domain-route-registration-precedence)
- [Payload исключения события `JobAttempted`](#jobattempted-event-exception-payload)
- [Привязка callback-функции для `extend` в менеджере](#manager-extend-callback-binding)
- [Запросы MySQL `DELETE` с `JOIN`, `ORDER BY` и `LIMIT`](#mysql-delete-queries-with-join-order-by-and-limit)
- [Имена Bootstrap-представлений для пагинации](#pagination-bootstrap-view-names)
- [Генерация имени полиморфной сводной таблицы](#polymorphic-pivot-table-name-generation)
- [Переименование свойства события `QueueBusy`](#queuebusy-event-property-rename)
- [Конфигурация `serialization` сессии](#session-serialization-configuration)
- [Сброс factories `Str` между тестами](#str-factories-reset-between-tests)

<!-- </div> -->

<a name="upgrade-13.0"></a>
## Обновление с 12.x до 13.0

#### Оценочное время обновления: 10 минут

> [!NOTE]
> Мы стараемся документировать все возможные критические изменения. Поскольку часть этих изменений находится в редко используемых областях фреймворка, только некоторые из них могут затронуть ваше приложение. Чтобы сэкономить время, можно использовать [Shift](https://laravelshift.com). Shift - поддерживаемый сообществом сервис, который автоматизирует обновления Laravel.

<a name="upgrading-using-ai"></a>
### Обновление с помощью AI

Вы можете автоматизировать обновление с помощью [Laravel Boost](https://github.com/laravel/boost). Boost - официальный MCP-сервер, который предоставляет вашему AI-ассистенту направленные prompt для обновления. После установки в любое приложение Laravel 12 используйте slash-команду `/upgrade-laravel-v13` в Claude Code, Cursor, OpenCode, Gemini или VS Code, чтобы начать обновление до Laravel 13. Для этой команды требуется Laravel Boost `^2.0`.

<a name="updating-dependencies"></a>
### Обновление зависимостей

**Вероятность влияния: высокая**

Обновите следующие зависимости в вашем файле `composer.json`:

<!-- <div class="content-list" markdown="1"> -->

- `laravel/framework` до `^13.0`
- `laravel/boost` до `^2.0`
- `laravel/tinker` до `^3.0`
- `phpunit/phpunit` до `^12.0`
- `pestphp/pest` до `^4.0`

<!-- </div> -->

<a name="updating-the-laravel-installer"></a>
### Обновление Laravel Installer

Если вы используете Laravel installer CLI tool для создания новых Laravel-приложений, обновите installer для совместимости с Laravel 13.x.

Если Laravel installer установлен через `composer global require`, обновите installer командой `composer global update`:

```shell
composer global update laravel/installer
```

Или, если вы используете копию установщика Laravel, поставляемую с [Laravel Herd](https://herd.laravel.com), обновите Herd до последнего релиза.

<a name="cache"></a>
### Cache

<a name="cache-prefixes-and-session-cookie-names"></a>
#### Префиксы кеша и имена session cookies

**Вероятность влияния: низкая**

Префиксы кеша и ключей Redis по умолчанию в Laravel теперь используют суффиксы через дефис.

В большинстве приложений это изменение не применяется, потому что конфигурационные файлы уровня приложения уже определяют эти значения. В основном оно влияет на приложения, которые полагаются на резервную конфигурацию уровня фреймворка, когда соответствующие значения отсутствуют в конфигурации приложения.

Если ваше приложение зависит от этих сгенерированных значений по умолчанию, ключи кеша и имена session cookie могут измениться после обновления:

```php
// Laravel <= 12.x
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_cache_';
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_database_';
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_session';

// Laravel >= 13.x
Str::slug((string) env('APP_NAME', 'laravel')).'-cache-';
Str::slug((string) env('APP_NAME', 'laravel')).'-database-';
Str::slug((string) env('APP_NAME', 'laravel')).'-session';
```

Чтобы сохранить прежнее поведение, явно настройте `CACHE_PREFIX`, `REDIS_PREFIX` и `SESSION_COOKIE` в окружении.

<a name="store-and-repository-contracts-touch"></a>
#### Контракты `Store` и `Repository`: `touch`

**Вероятность влияния: очень низкая**

Контракты кеша теперь включают метод `touch` для продления TTL элементов. Если вы поддерживаете пользовательские реализации хранилища кеша, добавьте этот метод:

```php
// Illuminate\Contracts\Cache\Store
public function touch($key, $seconds);
```

<a name="cache-serializable_classes-configuration"></a>
#### Конфигурация кеша `serializable_classes`

**Вероятность влияния: средняя**

Конфигурация `cache` приложения по умолчанию теперь включает опцию `serializable_classes`, установленную в `false`. Это усиливает безопасность десериализации кеша и помогает предотвратить цепочки атак через PHP-десериализацию, если `APP_KEY` вашего приложения утечет. Если приложение намеренно хранит PHP-объекты в кеше, явно перечислите классы, которые можно десериализовать:

```php
'serializable_classes' => [
    App\Data\CachedDashboardStats::class,
    App\Support\CachedPricingSnapshot::class,
],
```

Если приложение раньше полагалось на десериализацию произвольных кешированных объектов, нужно перенести такой код на явные списки разрешенных классов или на payload кеша без объектов, например массивы.

<a name="container"></a>
### Контейнер

<a name="containercall-and-nullable-class-defaults"></a>
#### `Container::call` и значения nullable-классов по умолчанию

**Вероятность влияния: низкая**

`Container::call` теперь учитывает значения по умолчанию для nullable-параметров классов, когда привязка отсутствует, что соответствует поведению внедрения зависимостей через конструктор, представленному в Laravel 12:

```php
$container->call(function (?Carbon $date = null) {
    return $date;
});

// Laravel <= 12.x: Carbon instance
// Laravel >= 13.x: null
```

Если ваша логика внедрения зависимостей при вызове метода зависела от прежнего поведения, ее может потребоваться обновить.

<a name="contracts"></a>
### Контракты

<a name="dispatcher-contract-dispatchafterresponse"></a>
#### Контракт `Dispatcher`: `dispatchAfterResponse`

**Вероятность влияния: очень низкая**

Контракт `Illuminate\Contracts\Bus\Dispatcher` теперь включает метод `dispatchAfterResponse($command, $handler = null)`.

Если вы поддерживаете пользовательскую реализацию диспетчера, добавьте этот метод в ваш класс.

<a name="responsefactory-contract-eventstream"></a>
#### Контракт `ResponseFactory`: `eventStream`

**Вероятность влияния: очень низкая**

Контракт `Illuminate\Contracts\Routing\ResponseFactory` теперь включает сигнатуру `eventStream`.

Если вы поддерживаете пользовательскую реализацию этого контракта, добавьте этот метод.

<a name="mustverifyemail-contract-markemailasunverified"></a>
#### Контракт `MustVerifyEmail`: `markEmailAsUnverified`

**Вероятность влияния: очень низкая**

Контракт `Illuminate\Contracts\Auth\MustVerifyEmail` теперь включает `markEmailAsUnverified()`.

Если у вас есть пользовательская реализация этого контракта, добавьте этот метод для совместимости.

<a name="database"></a>
### База данных

<a name="database-upsert-mariadb-mysql"></a>
#### Database `upsert` с MySQL или MariaDB

**Вероятность влияния: средняя**

Laravel теперь проверяет, что caller передал непустое значение `uniqueBy`, и выбрасывает `InvalidArgumentException` вместо генерации некорректного SQL.

Хотя database drivers MariaDB и MySQL игнорируют значение `uniqueBy` и всегда используют primary и unique indexes таблицы для обнаружения существующих записей, validation все равно применяется. Если `uniqueBy` пустой, будет выброшен `InvalidArgumentException`.

<a name="mysql-delete-queries-with-join-order-by-and-limit"></a>
#### Запросы MySQL `DELETE` с `JOIN`, `ORDER BY` и `LIMIT`

**Вероятность влияния: низкая**

Laravel теперь компилирует полные запросы `DELETE ... JOIN`, включая `ORDER BY` и `LIMIT`, для грамматики MySQL.

В предыдущих версиях выражения `ORDER BY` / `LIMIT` могли молча игнорироваться для удалений с `JOIN`. В Laravel 13 эти выражения включаются в сгенерированный SQL. В результате движки баз данных, которые не поддерживают такой синтаксис, например стандартные варианты MySQL / MariaDB, теперь могут выбросить `QueryException` вместо выполнения неограниченного удаления.

<a name="eloquent"></a>
### Eloquent

<a name="model-booting-and-nested-instantiation"></a>
#### Загрузка моделей и вложенное создание экземпляров

**Вероятность влияния: очень низкая**

Создание нового model instance во время booting этой же model теперь запрещено и выбрасывает `LogicException`.

Это влияет на код, который создает models внутри model `boot` methods или trait `boot*` methods:

```php
protected static function boot()
{
    parent::boot();

    // Больше не разрешено во время booting...
    (new static())->getTable();
}
```

Перенесите эту логику за пределы boot cycle, чтобы избежать nested booting.

<a name="polymorphic-pivot-table-name-generation"></a>
#### Генерация имени polymorphic pivot table

**Вероятность влияния: низкая**

Когда table names выводятся для polymorphic pivot models с custom pivot model classes, Laravel теперь генерирует pluralized names.

Если приложение зависело от прежних singular inferred names для morph pivot tables и использовало custom pivot classes, явно определите table name на pivot model.

<a name="collection-model-serialization-restores-eager-loaded-relations"></a>
#### Сериализация коллекций моделей восстанавливает eager-loaded relations

**Вероятность влияния: низкая**

Когда коллекции Eloquent models сериализуются и восстанавливаются, например в queued jobs, eager-loaded relations теперь восстанавливаются для models коллекции.

Если ваш код зависел от отсутствия relations после deserialization, эту логику может потребоваться изменить.

<a name="http-client"></a>
### HTTP Client

<a name="http-client-response-throw-and-throwif-signatures"></a>
#### Signatures `Response::throw` и `throwIf` в HTTP Client

**Вероятность влияния: очень низкая**

Методы response HTTP client теперь объявляют callback parameters в method signatures:

```php
public function throw($callback = null);
public function throwIf($condition, $callback = null);
```

Если вы переопределяете эти методы в custom response classes, убедитесь, что method signatures совместимы.

<a name="notifications"></a>
### Notifications

<a name="default-password-reset-subject"></a>
#### Default subject письма сброса пароля

**Вероятность влияния: очень низкая**

Default subject письма сброса пароля Laravel изменился:

```text
// Laravel <= 12.x
Reset Password Notification

// Laravel >= 13.x
Reset your password
```

Если ваши tests, assertions или translation overrides зависят от прежней строки по умолчанию, обновите их.

<a name="queued-notifications-and-missing-models"></a>
#### Queued notifications и отсутствующие models

**Вероятность влияния: очень низкая**

Queued notifications теперь учитывают атрибут `#[DeleteWhenMissingModels]` и свойство `$deleteWhenMissingModels`, определенные на notification class.

В предыдущих версиях missing models могли все еще приводить к сбою queued notification jobs в случаях, когда вы ожидали их удаления.

<a name="queue"></a>
### Queue

<a name="jobattempted-event-exception-payload"></a>
#### Exception payload события `JobAttempted`

**Вероятность влияния: низкая**

Событие `Illuminate\Queue\Events\JobAttempted` теперь предоставляет exception object или `null` через `$exception`, заменяя прежнее boolean-свойство `$exceptionOccurred`:

```php
// Laravel <= 12.x
$event->exceptionOccurred;

// Laravel >= 13.x
$event->exception;
```

Если вы слушаете это событие, обновите код слушателя.

<a name="queuebusy-event-property-rename"></a>
#### Переименование свойства события `QueueBusy`

**Вероятность влияния: низкая**

Свойство `$connection` события `Illuminate\Queue\Events\QueueBusy` переименовано в `$connectionName` для согласованности с другими событиями очереди.

Если ваши слушатели обращаются к `$connection`, обновите их на `$connectionName`.

<a name="queue-contract-method-additions"></a>
#### Добавления методов в контракт `Queue`

**Вероятность влияния: очень низкая**

Контракт `Illuminate\Contracts\Queue\Queue` теперь включает методы проверки размера очереди, которые раньше были объявлены только в docblocks.

Если вы поддерживаете пользовательские реализации драйвера очереди для этого контракта, добавьте реализации для:

<!-- <div class="content-list" markdown="1"> -->

- `pendingSize`
- `delayedSize`
- `reservedSize`
- `creationTimeOfOldestPendingJob`

<!-- </div> -->

<a name="routing"></a>
### Маршрутизация

<a name="domain-route-registration-precedence"></a>
#### Приоритет регистрации domain routes

**Вероятность влияния: низкая**

Routes с явно указанным domain теперь имеют приоритет над non-domain routes при route matching.

Это позволяет catch-all subdomain routes вести себя согласованно, даже если non-domain routes зарегистрированы раньше. Если ваше приложение зависело от прежнего registration precedence между domain и non-domain routes, проверьте route matching behavior.

<a name="session"></a>
### Сессия

<a name="session-serialization-configuration"></a>
#### Конфигурация `serialization` сессии

**Вероятность влияния: низкая**

Чтобы снизить риск атак с использованием цепочек гаджетов десериализации PHP, стандартный каркас приложения теперь устанавливает для параметра сессии `serialization` значение `json` в файле `config/session.php`.

Если вы обновляете существующее приложение и синхронизируете его конфигурационные файлы с каркасом Laravel 13, изменение этого значения с `php` на `json` сделает недействительными все активные пользовательские сессии.

Чтобы сохранить активные сессии при обновлении, оставьте значение `php`. Однако, если приложение не хранит PHP-объекты в сессии и вы готовы потребовать от пользователей повторной аутентификации, для повышения безопасности рекомендуется изменить значение на `json`.

<a name="scheduling"></a>
### Scheduling

<a name="withscheduling-registration-timing"></a>
#### Timing регистрации `withScheduling`

**Вероятность влияния: очень низкая**

Schedules, зарегистрированные через `ApplicationBuilder::withScheduling()`, теперь откладываются до момента разрешения `Schedule`.

Если ваше приложение зависело от немедленного timing регистрации schedule во время bootstrap, эту логику может потребоваться изменить.

<a name="security"></a>
### Security

<a name="request-forgery-protection"></a>
#### Защита от подделки запросов

**Вероятность влияния: высокая**

CSRF middleware Laravel переименован с `VerifyCsrfToken` в `PreventRequestForgery` и теперь включает проверку origin запроса с помощью заголовка `Sec-Fetch-Site`.

`VerifyCsrfToken` и `ValidateCsrfToken` остаются устаревшими псевдонимами, но прямые ссылки следует обновить на `PreventRequestForgery`, особенно при исключении middleware в тестах или определениях маршрутов:

```php
use Illuminate\Foundation\Http\Middleware\PreventRequestForgery;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;

// Laravel <= 12.x
->withoutMiddleware([VerifyCsrfToken::class]);

// Laravel >= 13.x
->withoutMiddleware([PreventRequestForgery::class]);
```

API конфигурации middleware теперь также предоставляет `preventRequestForgery(...)`.

<a name="support"></a>
### Support

<a name="manager-extend-callback-binding"></a>
#### Binding callback для `extend` в manager

**Вероятность влияния: низкая**

Custom driver closures, зарегистрированные через manager methods `extend`, теперь bound к manager instance.

Если раньше вы полагались на другой bound object, например service provider instance, как `$this` внутри этих callbacks, перенесите такие значения в closure captures через `use (...)`.

<a name="str-factories-reset-between-tests"></a>
#### Сброс `Str` factories между тестами

**Вероятность влияния: низкая**

Laravel теперь сбрасывает custom `Str` factories во время test teardown.

Если ваши tests зависели от того, что custom UUID / ULID / random string factories сохраняются между test methods, задавайте их в каждом соответствующем test или setup hook.

<a name="jsfrom-uses-unescaped-unicode-by-default"></a>
#### `Js::from` по умолчанию использует unescaped Unicode

**Вероятность влияния: очень низкая**

`Illuminate\Support\Js::from` теперь по умолчанию использует `JSON_UNESCAPED_UNICODE`.

Если ваши tests или frontend output comparisons зависели от escaped Unicode sequences, например `\u00e8`, обновите expectations.

<a name="utilities"></a>
### Utilities

<a name="symfony-polyfill"></a>
#### Symfony PHP 8.5 Polyfill и конфликты global functions

**Вероятность влияния: низкая**

Laravel 13 добавляет dependency `symfony/polyfill-php85`. На версиях PHP ниже 8.5 этот polyfill определяет global functions, такие как `array_first()` и `array_last()`, если они не были определены ранее во время bootstrap.

Эти функции могут конфликтовать с legacy helper packages вроде `laravel/helpers` или custom global helpers с такими же именами. Например, исторический helper `array_first()` принимал callback, чтобы вернуть первый подходящий element, тогда как polyfilled version возвращает только первый element массива.

Чтобы избежать конфликтов и обеспечить согласованное поведение между версиями PHP, предпочитайте методы `Illuminate\Support\Arr`:

```php
use Illuminate\Support\Arr;

Arr::first($array, function ($value) {
  return /* condition */;
});
```

<a name="views"></a>
### Представления

<a name="pagination-bootstrap-view-names"></a>
#### Имена Bootstrap-представлений для пагинации

**Вероятность влияния: низкая**

Внутренние имена представлений пагинации для значений Bootstrap 3 по умолчанию теперь явные:

```nothing
// Laravel <= 12.x
pagination::default
pagination::simple-default

// Laravel >= 13.x
pagination::bootstrap-3
pagination::simple-bootstrap-3
```

Если ваше приложение напрямую ссылается на старые имена представлений пагинации, обновите эти ссылки.

<a name="miscellaneous"></a>
### Разное

Мы также рекомендуем просмотреть изменения в GitHub-репозитории [`laravel/laravel`](https://github.com/laravel/laravel). Хотя многие из них необязательны, возможно, вы захотите синхронизировать соответствующие файлы с приложением. В этом руководстве описаны не все изменения, например правки файлов конфигурации или комментариев. Полный список можно изучить с помощью [инструмента сравнения GitHub](https://github.com/laravel/laravel/compare/12.x...13.x) и выбрать нужные обновления.
