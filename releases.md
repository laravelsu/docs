---
git: 8067015bff27252bd132f3ea313153912cd04a6c
---

# Примечания к релизу

- [Схема версионирования](#versioning-scheme)
- [Политика поддержки](#support-policy)
- [Laravel 13](#laravel-13)

<a name="versioning-scheme"></a>
## Схема версионирования

Laravel и другие его официальные пакеты следуют [семантическому версионированию](https://semver.org/lang/ru/). Мажорные релизы фреймворка выпускаются каждый год (примерно в первом квартале), тогда как минорные и patch-релизы могут выпускаться каждую неделю. Минорные и patch-релизы **никогда** не должны содержать критические изменения.

Ссылаясь на фреймворк Laravel или его компоненты из вашего приложения или пакета, всегда используйте ограничение версии вроде `^13.0`, поскольку мажорные релизы Laravel действительно включают критические изменения. Однако мы стремимся к тому, чтобы обновление до нового мажорного релиза занимало один день или меньше.

<a name="named-arguments"></a>
#### Именованные аргументы

[Именованные аргументы](https://www.php.net/manual/ru/functions.arguments.php#functions.named-arguments) не входят в рекомендации Laravel по обратной совместимости. При необходимости мы можем переименовать аргументы функций, чтобы улучшить кодовую базу Laravel. Поэтому использовать именованные аргументы при вызове методов Laravel следует осторожно и с пониманием того, что имена параметров могут измениться в будущем.

<a name="support-policy"></a>
## Политика поддержки

Для всех релизов Laravel bug fixes предоставляются в течение 18 месяцев, а security fixes - в течение 2 лет. Для всех дополнительных libraries только последний major release получает bug fixes. Кроме того, ознакомьтесь с версиями баз данных, которые [поддерживает Laravel](/docs/{{version}}/database#introduction).

<div id="support-policy">

| Версия | PHP (*)   | Дата релиза          | Bug fixes до        | Security fixes до     |
| ------ | --------- | -------------------- | ------------------- | --------------------- |
| 10     | 8.1 - 8.3 | 14 февраля 2023      | 6 августа 2024      | 4 февраля 2025        |
| 11     | 8.2 - 8.4 | 12 марта 2024        | 3 сентября 2025     | 12 марта 2026         |
| 12     | 8.2 - 8.5 | 24 февраля 2025      | 13 августа 2026     | 24 февраля 2027       |
| 13     | 8.3 - 8.5 | 17 марта 2026        | Q3 2027             | 17 марта 2028         |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>Окончание поддержки</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>Только security fixes</div>
    </div>
</div>

(*) Поддерживаемые версии PHP

<a name="laravel-13"></a>
## Laravel 13

Laravel 13 продолжает ежегодный цикл релизов Laravel с фокусом на AI-native-процессы, более строгие значения по умолчанию и более выразительные API для разработчиков. Этот релиз включает официальные AI-примитивы, JSON:API-ресурсы, возможности семантического и векторного поиска, а также постепенные улучшения очередей, кеша и безопасности.

<a name="minimal-breaking-changes"></a>
### Минимальные критические изменения

В течение этого цикла релиза значительная часть нашей работы была направлена на минимизацию критических изменений. Вместо этого мы сосредоточились на постоянных улучшениях удобства разработки в течение года, которые не ломают существующие приложения.

Поэтому Laravel 13 является относительно небольшим обновлением с точки зрения объема работ, при этом добавляя существенные новые возможности. В связи с этим большинство Laravel-приложений могут обновиться до Laravel 13 без значительных изменений кода приложения.

<a name="php-8"></a>
### PHP 8.3

Laravel 13.x требует PHP версии не ниже 8.3.

<a name="ai-sdk"></a>
### Laravel AI SDK

Laravel 13 представляет официальный [Laravel AI SDK](https://laravel.com/ai), предоставляющий единый API для генерации текста, агентов с вызовом инструментов, embeddings, аудио, изображений и интеграций с векторными хранилищами.

С AI SDK можно создавать AI-возможности, не зависящие от конкретного провайдера, сохраняя единый опыт разработки в стиле Laravel.

Например, базового агента можно вызвать одним prompt:

```php
use App\Ai\Agents\SalesCoach;

$response = SalesCoach::make()->prompt('Analyze this sales transcript...');

return (string) $response;
```

Laravel AI SDK также может генерировать изображения, аудио и embeddings.

Для визуальной генерации SDK предоставляет чистый API для создания изображений из обычных текстовых prompt:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

Для голосовых сценариев можно синтезировать естественно звучащее аудио из текста для ассистентов, озвучивания и возможностей доступности:

```php
use Laravel\Ai\Audio;

$audio = Audio::of('I love coding with Laravel.')->generate();

$rawContent = (string) $audio;
```

А для семантического поиска и retrieval-процессов можно генерировать embeddings напрямую из строк:

```php
use Illuminate\Support\Str;

$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

<a name="json-api"></a>
### JSON:API-ресурсы

Laravel теперь включает официальные [JSON:API-ресурсы](/docs/{{version}}/eloquent-resources#jsonapi-resources), упрощающие возврат ответов, соответствующих спецификации JSON:API.

JSON:API-ресурсы обрабатывают сериализацию объектов ресурсов, включение связей, разреженные наборы полей, ссылки и заголовки ответов, соответствующие JSON:API.

<a name="request-forgery-protection"></a>
### Защита от подделки запросов

Для безопасности middleware [защиты от подделки запросов](/docs/{{version}}/csrf#preventing-csrf-requests) Laravel улучшен и формализован как `PreventRequestForgery`, добавляя проверку запросов с учетом origin при сохранении совместимости с CSRF-защитой на основе токенов.

<a name="queue-routing"></a>
### Маршрутизация очередей

Laravel 13 добавляет [маршрутизацию очередей по классу](/docs/{{version}}/queues#queue-routing) через `Queue::route(...)`, позволяя централизованно определить правила маршрутизации очереди и подключения по умолчанию для конкретных заданий:

```php
Queue::route(ProcessPodcast::class, connection: 'redis', queue: 'podcasts');
```

<a name="php-attributes"></a>
### Расширенная поддержка PHP-атрибутов

Laravel 13 продолжает расширять официальную поддержку PHP-атрибутов во фреймворке, делая распространенную конфигурацию и поведенческие аспекты более декларативными и расположенными рядом с классами и методами.

Среди заметных добавлений - атрибуты контроллеров и авторизации вроде [`#[Middleware]`](/docs/{{version}}/controllers#controller-middleware) и [`#[Authorize]`](/docs/{{version}}/controllers#authorization-attributes), а также ориентированные на очереди элементы управления заданиями вроде [`#[Tries]`](/docs/{{version}}/queues#max-job-attempts-and-timeout), [`#[Backoff]`](/docs/{{version}}/queues#dealing-with-failed-jobs), [`#[Timeout]`](/docs/{{version}}/queues#max-job-attempts-and-timeout) и [`#[FailOnTimeout]`](/docs/{{version}}/queues#failing-on-timeout).

Например, middleware контроллера и проверки политик теперь можно объявлять прямо на классах и методах:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Routing\Attributes\Controllers\Authorize;
use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
class CommentController
{
    #[Middleware('subscribed')]
    #[Authorize('create', [Comment::class, 'post'])]
    public function store(Post $post)
    {
        // ...
    }
}
```

Дополнительные атрибуты также появились в Eloquent, событиях, уведомлениях, валидации, тестировании и API сериализации ресурсов, предоставляя согласованный подход на основе атрибутов во все большем числе областей фреймворка.

<a name="cache-touch"></a>
### Продление TTL кеша

Laravel теперь включает [`Cache::touch(...)`](/docs/{{version}}/cache), позволяющий продлить TTL существующего элемента кеша без получения и повторного сохранения его значения.

<a name="semantic-search"></a>
### Семантический / векторный поиск

Laravel 13 развивает семантический поиск благодаря встроенной поддержке векторных запросов, процессам работы с embeddings и связанным API, описанным в [поиске](/docs/{{version}}/search#semantic-vector-search), [запросах](/docs/{{version}}/queries#vector-similarity-clauses) и [AI SDK](/docs/{{version}}/ai-sdk#embeddings).

Эти возможности позволяют просто создавать поиск на базе AI с PostgreSQL + `pgvector`, включая поиск по сходству embeddings, сгенерированным напрямую из строк.

Например, семантический поиск по сходству можно выполнять прямо из построителя запросов:

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', 'Best wineries in Napa Valley')
    ->limit(10)
    ->get();
```
