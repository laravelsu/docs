---
git: 7069be16037d772d289b504f5bc878fb384e9238
---

# Laravel MCP

- [Введение](#introduction)
- [Установка](#installation)
    - [Публикация маршрутов](#publishing-routes)
- [Создание серверов](#creating-servers)
    - [Регистрация сервера](#server-registration)
    - [Web-серверы](#web-servers)
    - [Локальные серверы](#local-servers)
- [Tools](#tools)
    - [Создание tools](#creating-tools)
    - [Схемы входных данных tool](#tool-input-schemas)
    - [Схемы выходных данных tool](#tool-output-schemas)
    - [Валидация аргументов tool](#validating-tool-arguments)
    - [Dependency injection для tool](#tool-dependency-injection)
    - [Аннотации tool](#tool-annotations)
    - [Условная регистрация tool](#conditional-tool-registration)
    - [Ответы tool](#tool-responses)
- [Prompts](#prompts)
    - [Создание prompts](#creating-prompts)
    - [Аргументы prompt](#prompt-arguments)
    - [Валидация аргументов prompt](#validating-prompt-arguments)
    - [Dependency injection для prompt](#prompt-dependency-injection)
    - [Условная регистрация prompt](#conditional-prompt-registration)
    - [Ответы prompt](#prompt-responses)
- [Resources](#resources)
    - [Создание resources](#creating-resources)
    - [Resource templates](#resource-templates)
    - [URI и MIME type resource](#resource-uri-and-mime-type)
    - [Resource request](#resource-request)
    - [Dependency injection для resource](#resource-dependency-injection)
    - [Аннотации resource](#resource-annotations)
    - [Условная регистрация resource](#conditional-resource-registration)
    - [Ответы resource](#resource-responses)
- [Apps](#apps)
    - [Создание app resources](#creating-app-resources)
    - [Рендеринг apps из tools](#rendering-apps-from-tools)
    - [Видимость tools для app](#app-tool-visibility)
    - [Конфигурация app](#app-configuration)
    - [Создание apps с Boost](#building-apps-with-boost)
- [Metadata](#metadata)
- [Icons](#icons)
- [Аутентификация](#authentication)
    - [OAuth 2.1](#oauth)
    - [Sanctum](#sanctum)
- [Авторизация](#authorization)
- [MCP-клиент](#client)
    - [Подключение к серверам](#client-connecting)
    - [Именованные clients](#named-clients)
    - [Аутентификация client](#client-authentication)
    - [Tools](#client-tools)
    - [Prompts](#client-prompts)
    - [Resources](#client-resources)
- [Тестирование серверов](#testing-servers)
    - [MCP Inspector](#mcp-inspector)
    - [Unit-тесты](#unit-tests)

<a name="introduction"></a>
## Введение

[Laravel MCP](https://github.com/laravel/mcp) предоставляет простой и элегантный способ для AI-клиентов взаимодействовать с вашим Laravel-приложением через [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro). Пакет предлагает выразительный fluent-интерфейс для определения серверов, tools, resources и prompts, которые включают AI-взаимодействия с приложением.

<a name="installation"></a>
## Установка

Установите Laravel MCP через Composer:

```shell
composer require laravel/mcp
```

<a name="publishing-routes"></a>
### Публикация маршрутов

После установки выполните Artisan-команду `vendor:publish`, чтобы опубликовать файл `routes/ai.php`, в котором будут определяться MCP-серверы:

```shell
php artisan vendor:publish --tag=ai-routes
```

Команда создаст файл `routes/ai.php` в каталоге маршрутов приложения.

<a name="creating-servers"></a>
## Создание серверов

MCP-сервер создается Artisan-командой `make:mcp-server`. Сервер является центральной точкой коммуникации, которая открывает AI-клиентам MCP-возможности: tools, resources и prompts.

```shell
php artisan make:mcp-server WeatherServer
```

Команда создаст класс сервера в `app/Mcp/Servers`. Сгенерированный сервер расширяет базовый класс `Laravel\Mcp\Server` и содержит свойства для регистрации tools, resources и prompts:

```php
<?php

namespace App\Mcp\Servers;

use Laravel\Mcp\Server\Attributes\Instructions;
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Version;
use Laravel\Mcp\Server;

#[Name('Weather Server')]
#[Version('1.0.0')]
#[Instructions('This server provides weather information and forecasts.')]
class WeatherServer extends Server
{
    protected array $tools = [
        // GetCurrentWeatherTool::class,
    ];

    protected array $resources = [
        // WeatherGuidelinesResource::class,
    ];

    protected array $prompts = [
        // DescribeWeatherPrompt::class,
    ];
}
```

<a name="server-registration"></a>
### Регистрация сервера

После создания сервер нужно зарегистрировать в `routes/ai.php`. Laravel MCP предоставляет два способа регистрации: `web` для HTTP-доступных серверов и `local` для command-line серверов.

<a name="web-servers"></a>
### Web-серверы

Web-серверы доступны через HTTP POST-запросы и подходят для удаленных AI-клиентов или web-интеграций:

```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

Mcp::web('/mcp/weather', WeatherServer::class);
```

Как и обычные маршруты, web-серверы можно защищать middleware:

```php
Mcp::web('/mcp/weather', WeatherServer::class)
    ->middleware(['throttle:mcp']);
```

<a name="local-servers"></a>
### Локальные серверы

Локальные серверы запускаются как Artisan-команды и удобны для локальных AI-assistant интеграций, например [Laravel Boost](/docs/{{version}}/installation#installing-laravel-boost):

```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

Mcp::local('weather', WeatherServer::class);
```

Обычно не нужно вручную запускать Artisan-команду `mcp:start`. Настройте MCP-клиент (AI-агент), чтобы он запускал сервер, или используйте [MCP Inspector](#mcp-inspector).

<a name="tools"></a>
## Tools

Tools позволяют серверу открывать функциональность, которую AI-клиенты могут вызывать. Через tools языковые модели могут выполнять действия, запускать код или взаимодействовать с внешними системами:

```php
<?php

namespace App\Mcp\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Tool;

#[Description('Fetches the current weather forecast for a specified location.')]
class CurrentWeatherTool extends Tool
{
    public function handle(Request $request): Response
    {
        $location = $request->get('location');

        return Response::text('The weather is...');
    }

    public function schema(JsonSchema $schema): array
    {
        return [
            'location' => $schema->string()
                ->description('The location to get the weather for.')
                ->required(),
        ];
    }
}
```

<a name="creating-tools"></a>
### Создание tools

Создайте tool командой `make:mcp-tool`:

```shell
php artisan make:mcp-tool CurrentWeatherTool
```

Затем зарегистрируйте его в свойстве `$tools` сервера:

```php
use App\Mcp\Tools\CurrentWeatherTool;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    protected array $tools = [
        CurrentWeatherTool::class,
    ];
}
```

<a name="tool-name-title-description"></a>
#### Имя, заголовок и описание tool

По умолчанию имя и title tool выводятся из имени класса. Например, `CurrentWeatherTool` получит имя `current-weather` и title `Current Weather Tool`. Эти значения можно настроить атрибутами `Name` и `Title`:

```php
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Title;

#[Name('get-optimistic-weather')]
#[Title('Get Optimistic Weather Forecast')]
class CurrentWeatherTool extends Tool
{
    // ...
}
```

Описание tool не генерируется автоматически. Всегда добавляйте осмысленное описание через `Description`:

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Fetches the current weather forecast for a specified location.')]
class CurrentWeatherTool extends Tool
{
    //
}
```

> [!NOTE]
> Description является критичной частью metadata tool, так как помогает AI-модели понять, когда и как использовать tool.

<a name="tool-input-schemas"></a>
### Схемы входных данных tool

Tools могут определять input schemas, описывающие аргументы, принимаемые от AI-клиентов. Используйте builder `Illuminate\Contracts\JsonSchema\JsonSchema`:

```php
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    public function schema(JsonSchema $schema): array
    {
        return [
            'location' => $schema->string()
                ->description('The location to get the weather for.')
                ->required(),

            'units' => $schema->string()
                ->enum(['celsius', 'fahrenheit'])
                ->description('The temperature units to use.')
                ->default('celsius'),
        ];
    }
}
```

<a name="tool-output-schemas"></a>
### Схемы выходных данных tool

Tools могут определять [output schemas](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema), чтобы описать структуру своих ответов. Это улучшает интеграцию с AI-клиентами, которым нужны parseable results:

```php
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    public function outputSchema(JsonSchema $schema): array
    {
        return [
            'temperature' => $schema->number()->description('Temperature in Celsius')->required(),
            'conditions' => $schema->string()->description('Weather conditions')->required(),
            'humidity' => $schema->integer()->description('Humidity percentage')->required(),
        ];
    }
}
```

<a name="validating-tool-arguments"></a>
### Валидация аргументов tool

JSON Schema задает базовую структуру аргументов, но для более сложных правил можно использовать [валидацию Laravel](/docs/{{version}}/validation) внутри метода `handle`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    public function handle(Request $request): Response
    {
        $validated = $request->validate([
            'location' => 'required|string|max:100',
            'units' => 'in:celsius,fahrenheit',
        ]);

        // Fetch weather data using the validated arguments...
    }
}
```

При ошибке валидации AI-клиент будет действовать на основе ваших сообщений об ошибках, поэтому они должны быть ясными и actionable:

```php
$validated = $request->validate([
    'location' => ['required','string','max:100'],
    'units' => 'in:celsius,fahrenheit',
],[
    'location.required' => 'You must specify a location to get the weather for. For example, "New York City" or "Tokyo".',
    'units.in' => 'You must specify either "celsius" or "fahrenheit" for the units.',
]);
```

<a name="tool-dependency-injection"></a>
#### Dependency injection для tool

Все tools разрешаются через [service container](/docs/{{version}}/container) Laravel, поэтому зависимости можно type-hint в конструкторе:

```php
use App\Repositories\WeatherRepository;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    public function __construct(
        protected WeatherRepository $weather,
    ) {}
}
```

Зависимости можно также type-hint в методе `handle()`:

```php
use App\Repositories\WeatherRepository;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    public function handle(Request $request, WeatherRepository $weather): Response
    {
        $forecast = $weather->getForecastFor($request->get('location'));

        // ...
    }
}
```

<a name="tool-annotations"></a>
### Аннотации tool

Tools можно дополнить [annotations](https://modelcontextprotocol.io/specification/2025-06-18/schema#toolannotations), которые дают AI-клиентам дополнительную metadata о поведении tool:

```php
use Laravel\Mcp\Server\Tools\Annotations\IsIdempotent;
use Laravel\Mcp\Server\Tools\Annotations\IsReadOnly;
use Laravel\Mcp\Server\Tool;

#[IsIdempotent]
#[IsReadOnly]
class CurrentWeatherTool extends Tool
{
    //
}
```

Доступные аннотации:

<div class="overflow-auto">

| Аннотация          | Тип     | Описание                                                                                  |
| ------------------ | ------- | ----------------------------------------------------------------------------------------- |
| `#[IsReadOnly]`    | boolean | Tool не изменяет окружение.                                                               |
| `#[IsDestructive]` | boolean | Tool может выполнять разрушительные изменения.                                            |
| `#[IsIdempotent]`  | boolean | Повторные вызовы с теми же аргументами не имеют дополнительного эффекта.                  |
| `#[IsOpenWorld]`   | boolean | Tool может взаимодействовать с внешними сущностями.                                       |

</div>

Значения можно задавать явно:

```php
#[IsReadOnly(true)]
#[IsDestructive(false)]
#[IsOpenWorld(false)]
#[IsIdempotent(true)]
class CurrentWeatherTool extends Tool
{
    //
}
```

<a name="conditional-tool-registration"></a>
### Условная регистрация tool

Tool можно регистрировать условно во время выполнения, реализовав метод `shouldRegister`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    public function shouldRegister(Request $request): bool
    {
        return $request?->user()?->subscribed() ?? false;
    }
}
```

Если `shouldRegister` возвращает `false`, tool не появится в списке доступных и не сможет быть вызван AI-клиентами.

<a name="tool-responses"></a>
### Ответы tool

Tools должны возвращать `Laravel\Mcp\Response`. Для простого текста используйте `text`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

public function handle(Request $request): Response
{
    return Response::text('Weather Summary: Sunny, 72°F');
}
```

Для ошибки используйте `error`:

```php
return Response::error('Unable to fetch weather data. Please try again.');
```

Для image/audio content используйте `image` и `audio`:

```php
return Response::image(file_get_contents(storage_path('weather/radar.png')), 'image/png');

return Response::audio(file_get_contents(storage_path('weather/alert.mp3')), 'audio/mp3');
```

Image и audio можно загружать напрямую из Laravel filesystem disk методом `fromStorage`; MIME type будет определен автоматически:

```php
return Response::fromStorage('weather/radar.png');
```

При необходимости можно указать конкретный disk или переопределить MIME type:

```php
return Response::fromStorage('weather/radar.png', disk: 's3');

return Response::fromStorage('weather/radar.png', mimeType: 'image/webp');
```

<a name="multiple-content-responses"></a>
#### Ответы с несколькими content-блоками

Tool может вернуть несколько частей content, вернув массив `Response`:

```php
public function handle(Request $request): array
{
    return [
        Response::text('Weather Summary: Sunny, 72°F'),
        Response::text("**Detailed Forecast**\n- Morning: 65°F\n- Afternoon: 78°F\n- Evening: 70°F")
    ];
}
```

<a name="structured-responses"></a>
#### Структурированные ответы

Tools могут возвращать [structured content](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#structured-content) методом `structured`:

```php
return Response::structured([
    'temperature' => 22.5,
    'conditions' => 'Partly cloudy',
    'humidity' => 65,
]);
```

Если нужно добавить пользовательский текст вместе со structured content, используйте `withStructuredContent`:

```php
return Response::make(
    Response::text('Weather is 22.5°C and sunny')
)->withStructuredContent([
    'temperature' => 22.5,
    'conditions' => 'Sunny',
]);
```

<a name="streaming-responses"></a>
#### Потоковые ответы

Для долгих операций или real-time data streaming tool может вернуть [generator](https://www.php.net/manual/ru/language.generators.overview.php) из `handle`, отправляя промежуточные обновления клиенту:

```php
use Generator;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    public function handle(Request $request): Generator
    {
        $locations = $request->array('locations');

        foreach ($locations as $index => $location) {
            yield Response::notification('processing/progress', [
                'current' => $index + 1,
                'total' => count($locations),
                'location' => $location,
            ]);

            yield Response::text($this->forecastFor($location));
        }
    }
}
```

Для web-серверов streaming responses автоматически открывают SSE (Server-Sent Events) stream.

<a name="prompts"></a>
## Prompts

[Prompts](https://modelcontextprotocol.io/specification/2025-06-18/server/prompts) позволяют серверу делиться reusable prompt templates, которые AI-клиенты используют при взаимодействии с language models. Это стандартизированный способ структурировать типовые запросы.

<a name="creating-prompts"></a>
### Создание prompts

Создайте prompt командой:

```shell
php artisan make:mcp-prompt DescribeWeatherPrompt
```

Затем зарегистрируйте его в `$prompts` сервера:

```php
use App\Mcp\Prompts\DescribeWeatherPrompt;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    protected array $prompts = [
        DescribeWeatherPrompt::class,
    ];
}
```

<a name="prompt-name-title-and-description"></a>
#### Имя, заголовок и описание prompt

Имя и title prompt по умолчанию выводятся из класса. Например, `DescribeWeatherPrompt` получит имя `describe-weather` и title `Describe Weather Prompt`. Настроить эти значения можно атрибутами `Name` и `Title`:

```php
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Title;

#[Name('weather-assistant')]
#[Title('Weather Assistant Prompt')]
class DescribeWeatherPrompt extends Prompt
{
    // ...
}
```

Descriptions для prompts не генерируются автоматически. Всегда задавайте осмысленное описание через атрибут `Description`:

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Generates a natural-language explanation of the weather for a given location.')]
class DescribeWeatherPrompt extends Prompt
{
    //
}
```

> [!NOTE]
> Description - критически важная часть metadata prompt, так как помогает AI-моделям понять, когда и как лучше использовать этот prompt.

<a name="prompt-arguments"></a>
### Аргументы prompt

Prompts могут определять аргументы для настройки шаблона:

```php
use Laravel\Mcp\Server\Prompt;
use Laravel\Mcp\Server\Prompts\Argument;

class DescribeWeatherPrompt extends Prompt
{
    public function arguments(): array
    {
        return [
            new Argument(
                name: 'tone',
                description: 'The tone to use in the weather description (e.g., formal, casual, humorous).',
                required: true,
            ),
        ];
    }
}
```

<a name="validating-prompt-arguments"></a>
### Валидация аргументов prompt

Аргументы prompt автоматически валидируются на основе определения, но сложные правила можно добавить через Laravel validation в `handle`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    public function handle(Request $request): Response
    {
        $validated = $request->validate([
            'tone' => 'required|string|max:50',
        ]);

        $tone = $validated['tone'];
    }
}
```

Сообщения об ошибках должны быть понятными:

```php
$validated = $request->validate([
    'tone' => ['required','string','max:50'],
],[
    'tone.*' => 'You must specify a tone for the weather description. Examples include "formal", "casual", or "humorous".',
]);
```

<a name="prompt-dependency-injection"></a>
### Dependency injection для prompt

Prompts разрешаются через service container, поэтому зависимости можно type-hint в конструкторе или в методе `handle`:

```php
use App\Repositories\WeatherRepository;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    public function __construct(
        protected WeatherRepository $weather,
    ) {}
}
```

```php
use App\Repositories\WeatherRepository;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    public function handle(Request $request, WeatherRepository $weather): Response
    {
        $isAvailable = $weather->isServiceAvailable();

        // ...
    }
}
```

<a name="conditional-prompt-registration"></a>
### Условная регистрация prompt

Prompts можно регистрировать условно методом `shouldRegister`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Server\Prompt;

class CurrentWeatherPrompt extends Prompt
{
    public function shouldRegister(Request $request): bool
    {
        return $request?->user()?->subscribed() ?? false;
    }
}
```

Если метод возвращает `false`, prompt не появится в списке доступных.

<a name="prompt-responses"></a>
### Ответы prompt

Prompt может вернуть один `Laravel\Mcp\Response` или iterable responses:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    public function handle(Request $request): array
    {
        $tone = $request->string('tone');

        $systemMessage = "You are a helpful weather assistant. Please provide a weather description in a {$tone} tone.";

        $userMessage = "What is the current weather like in New York City?";

        return [
            Response::text($systemMessage)->asAssistant(),
            Response::text($userMessage),
        ];
    }
}
```

Метод `asAssistant()` указывает, что сообщение ответа должно трактоваться как сообщение AI assistant; обычные messages считаются user input.

<a name="resources"></a>
## Resources

[Resources](https://modelcontextprotocol.io/specification/2025-06-18/server/resources) позволяют серверу открывать данные и content, которые AI-клиенты могут читать и использовать как контекст. Это способ делиться статической или динамической информацией: документацией, конфигурацией или любыми данными, помогающими формировать ответы.

<a name="creating-resources"></a>
## Создание resources

Создайте resource командой:

```shell
php artisan make:mcp-resource WeatherGuidelinesResource
```

Затем зарегистрируйте его в `$resources` сервера:

```php
use App\Mcp\Resources\WeatherGuidelinesResource;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    protected array $resources = [
        WeatherGuidelinesResource::class,
    ];
}
```

<a name="resource-name-title-and-description"></a>
#### Имя, заголовок и описание resource

Имя и title resource по умолчанию выводятся из класса. Настройте их атрибутами `Name` и `Title`:

```php
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Title;

#[Name('weather-api-docs')]
#[Title('Weather API Documentation')]
class WeatherGuidelinesResource extends Resource
{
    // ...
}
```

Descriptions для resources не генерируются автоматически. Всегда задавайте осмысленное описание через атрибут `Description`:

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Comprehensive guidelines for using the Weather API.')]
class WeatherGuidelinesResource extends Resource
{
    //
}
```

> [!NOTE]
> Description - критически важная часть metadata resource, так как помогает AI-моделям понять, когда и как лучше использовать этот resource.

<a name="resource-templates"></a>
### Resource templates

[Resource templates](https://modelcontextprotocol.io/specification/2025-06-18/server/resources#resource-templates) позволяют открывать динамические resources, URI которых соответствуют шаблонам с переменными. Вместо статического URI для каждого resource можно создать один resource, обрабатывающий несколько URI по template pattern.

<a name="creating-resource-templates"></a>
#### Создание resource templates

Реализуйте `HasUriTemplate` и верните `UriTemplate`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Attributes\MimeType;
use Laravel\Mcp\Server\Contracts\HasUriTemplate;
use Laravel\Mcp\Server\Resource;
use Laravel\Mcp\Support\UriTemplate;

#[Description('Access user files by ID')]
#[MimeType('text/plain')]
class UserFileResource extends Resource implements HasUriTemplate
{
    public function uriTemplate(): UriTemplate
    {
        return new UriTemplate('file://users/{userId}/files/{fileId}');
    }

    public function handle(Request $request): Response
    {
        $userId = $request->get('userId');
        $fileId = $request->get('fileId');

        return Response::text($content);
    }
}
```

Когда resource реализует `HasUriTemplate`, он регистрируется как template. AI-клиенты могут запрашивать URI, соответствующие шаблону, а переменные автоматически извлекаются в request.

<a name="uri-template-syntax"></a>
#### Синтаксис URI template

URI templates используют placeholders в фигурных скобках:

```php
new UriTemplate('file://users/{userId}');
new UriTemplate('file://users/{userId}/files/{fileId}');
new UriTemplate('https://api.example.com/{version}/{resource}/{id}');
```

<a name="accessing-template-variables"></a>
#### Доступ к переменным template

Когда URI соответствует template, переменные объединяются с request и доступны через `get`:

```php
class UserProfileResource extends Resource implements HasUriTemplate
{
    public function uriTemplate(): UriTemplate
    {
        return new UriTemplate('file://users/{userId}/profile');
    }

    public function handle(Request $request): Response
    {
        $userId = $request->get('userId');
        $uri = $request->uri();

        return Response::text("Profile for user {$userId}");
    }
}
```

<a name="resource-uri-and-mime-type"></a>
### URI и MIME type resource

Каждый resource идентифицируется уникальным URI и связанным MIME type. По умолчанию URI строится из имени resource, поэтому `WeatherGuidelinesResource` получит URI `weather://resources/weather-guidelines`, а default MIME type - `text/plain`.

Значения можно настроить атрибутами `Uri` и `MimeType`:

```php
use Laravel\Mcp\Server\Attributes\MimeType;
use Laravel\Mcp\Server\Attributes\Uri;
use Laravel\Mcp\Server\Resource;

#[Uri('weather://resources/guidelines')]
#[MimeType('application/pdf')]
class WeatherGuidelinesResource extends Resource
{
}
```

<a name="resource-request"></a>
### Resource request

В отличие от tools и prompts, resources не могут определять input schemas или arguments. Однако вы по-прежнему можете работать с request object в методе `handle` вашего resource:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    public function handle(Request $request): Response
    {
        // ...
    }
}
```

<a name="resource-dependency-injection"></a>
### Dependency injection для resource

Resources разрешаются через service container, поэтому зависимости можно type-hint в конструкторе или в `handle`:

```php
use App\Repositories\WeatherRepository;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    public function __construct(
        protected WeatherRepository $weather,
    ) {}
}
```

```php
use App\Repositories\WeatherRepository;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    public function handle(WeatherRepository $weather): Response
    {
        $guidelines = $weather->guidelines();

        return Response::text($guidelines);
    }
}
```

<a name="resource-annotations"></a>
### Аннотации resource

Resources можно дополнить [annotations](https://modelcontextprotocol.io/specification/2025-06-18/schema#resourceannotations), которые передают AI-клиентам дополнительную metadata:

```php
use Laravel\Mcp\Enums\Role;
use Laravel\Mcp\Server\Annotations\Audience;
use Laravel\Mcp\Server\Annotations\LastModified;
use Laravel\Mcp\Server\Annotations\Priority;
use Laravel\Mcp\Server\Resource;

#[Audience(Role::User)]
#[LastModified('2025-01-12T15:00:58Z')]
#[Priority(0.9)]
class UserDashboardResource extends Resource
{
    //
}
```

<div class="overflow-auto">

| Аннотация        | Тип           | Описание                                                                  |
| ---------------- | ------------- | ------------------------------------------------------------------------- |
| `#[Audience]`    | Role or array | Целевая аудитория: `Role::User`, `Role::Assistant` или оба варианта.      |
| `#[Priority]`    | float         | Числовая оценка важности resource от `0.0` до `1.0`.                      |
| `#[LastModified]`| string        | ISO 8601 timestamp последнего обновления resource.                        |

</div>

<a name="conditional-resource-registration"></a>
### Условная регистрация resource

Resources можно регистрировать условно методом `shouldRegister`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    public function shouldRegister(Request $request): bool
    {
        return $request?->user()?->subscribed() ?? false;
    }
}
```

Если метод возвращает `false`, resource не появится в списке доступных.

<a name="resource-responses"></a>
### Ответы resource

Resources должны возвращать `Laravel\Mcp\Response`. Для простого текста используйте `text`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

public function handle(Request $request): Response
{
    return Response::text($weatherData);
}
```

<a name="resource-link-responses"></a>
#### Resource link responses

Чтобы вернуть ссылку на resource, используйте метод `resourceLink`, передав URI и имя. В отличие от embedded resource, resource link возвращает указатель URI, который AI-клиент получает самостоятельно:

```php
return Response::resourceLink(
    uri: 'file:///data/report.json',
    name: 'monthly-report',
    mimeType: 'application/json',
);
```

Также можно передать зарегистрированный класс или экземпляр resource; в этом случае URI, имя, title, description и MIME type будут автоматически взяты из resource:

```php
return Response::resourceLink(new WeatherForecastResource);
```

<a name="resource-blob-responses"></a>
#### Blob responses

Для blob content используйте метод `blob`:

```php
return Response::blob(file_get_contents(storage_path('weather/radar.png')));
```

MIME type blob определяется MIME type, настроенным у resource:

```php
use Laravel\Mcp\Server\Attributes\MimeType;
use Laravel\Mcp\Server\Resource;

#[MimeType('image/png')]
class WeatherGuidelinesResource extends Resource
{
    //
}
```

<a name="resource-error-responses"></a>
#### Error responses

Чтобы сообщить об ошибке при получении resource, используйте `error()`:

```php
return Response::error('Unable to fetch weather data for the specified location.');
```

<a name="apps"></a>
## Apps

Laravel MCP поддерживает [MCP Apps](https://modelcontextprotocol.io/extensions/apps/overview) - расширение Model Context Protocol, которое позволяет tools отображать интерактивные HTML-приложения внутри sandboxed iframes в поддерживаемых hosts. Это позволяет создавать dashboards, формы, визуализации и другие насыщенные интерфейсы, выходящие за рамки простых текстовых ответов.

MCP app состоит из двух совместно работающих частей:

<div class="content-list" markdown="1">

- **App resource**, который возвращает самодостаточный HTML вашего приложения.
- **Tool**, связанный с app resource через атрибут `#[RendersApp]`. Когда tool вызывается, host получает и отображает связанный resource.

</div>

<a name="creating-app-resources"></a>
### Создание app resources

Создать app resource можно с помощью Artisan-команды `make:mcp-app-resource`:

```shell
php artisan make:mcp-app-resource WeatherDashboardApp
```

Эта команда создает два файла: PHP-класс в `app/Mcp/Resources` и Blade-view в `resources/views/mcp`. Имя view автоматически выводится из имени класса. Например, `WeatherDashboardApp` соответствует `mcp.weather-dashboard-app`:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\AppMeta;
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\AppResource;

#[Description('An interactive weather dashboard.')]
#[AppMeta]
class WeatherDashboardApp extends AppResource
{
    public function handle(Request $request): Response
    {
        return Response::view('mcp.weather-dashboard-app', [
            'title' => $this->title(),
        ]);
    }
}
```

`AppResource` расширяет базовый класс `Resource` и автоматически настраивает схему URI `ui://` и MIME type `text/html;profile=mcp-app`, необходимые спецификацией MCP Apps. Как и любой другой resource, его нужно зарегистрировать в массиве `$resources` вашего сервера.

Сгенерированный Blade-view использует компонент `<x-mcp::app>`, который рендерит полный HTML-документ со встроенным client-side MCP SDK, готовым к использованию:

```blade
<x-mcp::app :title="$title">
    <x-slot:head>
        <script type="module">
        createMcpApp(async (app) => {
            document.getElementById('run-btn').addEventListener('click', async () => {
                const result = await app.callServerTool('get-weather-data', {});
                document.getElementById('output').textContent = result.content[0]?.text ?? '';
            });
        });
        </script>
    </x-slot:head>

    <div id="app">
        <button id="run-btn">Refresh</button>
        <p id="output"></p>
    </div>
</x-mcp::app>
```

Глобальная функция `createMcpApp` предоставляется встроенным SDK и отвечает за подключение iframe к серверу, применение host-темы и доступ к helper-методам вроде `callServerTool`, `sendMessage`, `openLink` и event callbacks. Полное client-side API смотрите в [спецификации MCP Apps](https://modelcontextprotocol.io/extensions/apps/overview).

<a name="rendering-apps-from-tools"></a>
### Рендеринг apps из tools

Чтобы отобразить app resource, свяжите с ним tool с помощью атрибута `#[RendersApp]`. Когда tool вызывается, Laravel MCP добавляет URI resource в metadata tool, чтобы host мог отрендерить app в sandboxed iframe:

```php
<?php

namespace App\Mcp\Tools;

use App\Mcp\Resources\WeatherDashboardApp;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\RendersApp;
use Laravel\Mcp\Server\Tool;

#[RendersApp(resource: WeatherDashboardApp::class)]
class ShowWeatherDashboard extends Tool
{
    public function handle(Request $request): Response
    {
        return Response::text('Weather dashboard loaded.');
    }
}
```

Laravel MCP автоматически объявляет capability `io.modelcontextprotocol/ui`, когда зарегистрирован любой `AppResource`, поэтому дополнительная конфигурация сервера не требуется.

<a name="app-tool-visibility"></a>
### Видимость tools для app

Каждый tool с атрибутом `#[RendersApp]` может ограничить, кто имеет право его вызывать, через аргумент `visibility`. Это полезно для приватных, доступных только app tools, которые UI вызывает для загрузки или обновления данных, не показывая эти tools модели:

```php
use Laravel\Mcp\Server\Attributes\RendersApp;
use Laravel\Mcp\Server\Ui\Enums\Visibility;

#[RendersApp(resource: WeatherDashboardApp::class, visibility: [Visibility::App])]
class GetWeatherData extends Tool
{
    // ...
}
```

Enum `Visibility` содержит два значения: `Model` и `App`; по умолчанию используются оба. Используйте `[Visibility::App]` для backend actions, которые UI вызывает напрямую, или `[Visibility::Model]`, чтобы сделать tool недоступным для UI.

<a name="app-configuration"></a>
### Конфигурация app

Атрибут `#[AppMeta]` на app resource настраивает Content Security Policy iframe, browser permissions и library scripts, которые должны быть включены в `<head>` view:

```php
use Laravel\Mcp\Server\Attributes\AppMeta;
use Laravel\Mcp\Server\Ui\Enums\Library;
use Laravel\Mcp\Server\Ui\Enums\Permission;

#[AppMeta(
    connectDomains: ['https://api.weather.com'],
    permissions: [Permission::Geolocation],
    libraries: [Library::Tailwind, Library::Alpine],
)]
class WeatherDashboardApp extends AppResource
{
    // ...
}
```

Enum `Library` включает заранее настроенные CDN scripts для популярных front-end libraries, таких как `Library::Tailwind` и `Library::Alpine`, а их CDN origins автоматически добавляются в CSP. Enum `Permission` покрывает browser permissions вроде `Camera`, `Microphone`, `Geolocation` и `ClipboardWrite`.

Для вычисляемой или динамической конфигурации переопределите метод `appMeta` в resource, используя fluent builders `AppMeta`, `Csp` и `Permissions` из namespace `Laravel\Mcp\Server\Ui`.

<a name="building-apps-with-boost"></a>
### Создание apps с Boost

Laravel MCP включает специальный reference для навыка [Boost](/docs/{{version}}/boost), посвященный созданию MCP Apps. Если у вас установлен [Laravel Boost](/docs/{{version}}/boost), ваш AI coding agent может вызвать навык `mcp-development` и попросить его сгенерировать app resource, Blade-view и связанный tool.

Полный protocol reference, включая полное client-side API и детали schema, смотрите в официальной [документации MCP Apps](https://modelcontextprotocol.io/extensions/apps/overview).

<a name="metadata"></a>
## Metadata

Laravel MCP поддерживает поле `_meta`, определенное в [спецификации MCP](https://modelcontextprotocol.io/specification/2025-06-18/basic#meta). Metadata можно применять к tools, resources, prompts и их responses.

Metadata для отдельного response content добавляется методом `withMeta`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

public function handle(Request $request): Response
{
    return Response::text('The weather is sunny.')
        ->withMeta(['source' => 'weather-api', 'cached' => true]);
}
```

Для result-level metadata, применяемой ко всему response envelope, оберните responses через `Response::make`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\ResponseFactory;

public function handle(Request $request): ResponseFactory
{
    return Response::make(
        Response::text('The weather is sunny.')
    )->withMeta(['request_id' => '12345']);
}
```

Чтобы добавить metadata к самому tool, resource или prompt, определите свойство `$meta`:

```php
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Tool;

#[Description('Fetches the current weather forecast.')]
class CurrentWeatherTool extends Tool
{
    protected ?array $meta = [
        'version' => '2.0',
        'author' => 'Weather Team',
    ];
}
```

<a name="icons"></a>
## Icons

MCP-клиенты могут отображать icons для вашего сервера и его primitives. Icons можно объявлять на server, tool, resource или prompt с помощью атрибута `Icon`:

```php
use Laravel\Mcp\Enums\IconTheme;
use Laravel\Mcp\Server\Attributes\Icon;

#[Icon('mcp/server.png', mimeType: 'image/png', sizes: ['48x48'])]
#[Icon('mcp/server-dark.svg', theme: IconTheme::Dark)]
class WeatherServer extends Server
{
    // ...
}
```

Атрибут `Icon` можно повторять, поэтому вы можете объявить несколько icons для разных размеров или вариантов светлой и темной темы.

Также icons можно определить программно, переопределив метод `icons`; это удобно, когда icon зависит от условий во время выполнения:

```php
use Laravel\Mcp\Schema\Icon;

class CurrentWeatherTool extends Tool
{
    /**
     * Get the tool's icons.
     *
     * @return array<int, Icon>
     */
    public function icons(): array
    {
        return [
            Icon::from('mcp/tool.png', mimeType: 'image/png'),
        ];
    }
}
```

Icons, заданные через атрибут и метод `icons`, автоматически объединяются. Пути icons разрешаются следующим образом:

<div class="content-list" markdown="1">

- Пути со схемой URI, например `https:` или `data:`, используются как есть.
- Относительные пути преобразуются в URL с помощью Laravel helper `asset`.

</div>

<a name="authentication"></a>
## Аутентификация

Как и маршруты, web MCP-серверы можно аутентифицировать middleware. Добавление аутентификации потребует от пользователя пройти проверку перед использованием любых возможностей сервера.

Есть два основных способа защитить MCP-сервер: простая token-based аутентификация через [Laravel Sanctum](/docs/{{version}}/sanctum) или любой token из HTTP-заголовка `Authorization`; либо OAuth через [Laravel Passport](/docs/{{version}}/passport).

<a name="oauth"></a>
### OAuth 2.1

Самый надежный способ защитить web MCP-серверы - OAuth через [Laravel Passport](/docs/{{version}}/passport).

При OAuth-аутентификации вызовите `Mcp::oauthRoutes` в `routes/ai.php`, чтобы зарегистрировать OAuth2 discovery и client registration routes. Затем примените middleware Passport `auth:api` к маршруту `Mcp::web`:

```php
use App\Mcp\Servers\WeatherExample;
use Laravel\Mcp\Facades\Mcp;

Mcp::oauthRoutes();

Mcp::web('/mcp/weather', WeatherExample::class)
    ->middleware('auth:api');
```

#### Новая установка Passport

Если приложение еще не использует Laravel Passport, следуйте [руководству по установке и deployment Passport](/docs/{{version}}/passport#installation). Перед продолжением у вас должны быть `OAuthenticatable` model, новый authentication guard и passport keys.

Затем опубликуйте view авторизации Passport, предоставляемый Laravel MCP:

```shell
php artisan vendor:publish --tag=mcp-views
```

После этого укажите Passport использовать этот view через `Passport::authorizationView`, обычно в методе `boot` `AppServiceProvider`:

```php
use Laravel\Passport\Passport;

public function boot(): void
{
    Passport::authorizationView(function ($parameters) {
        return view('mcp.authorize', $parameters);
    });
}
```

Этот view будет показан пользователю во время аутентификации, чтобы разрешить или отклонить попытку аутентификации AI-агента.

> [!NOTE]
> В этом сценарии OAuth используется как слой трансляции к базовой authenticatable model. Многие аспекты OAuth, например scopes, здесь намеренно не используются.

#### Использование существующей установки Passport

Если приложение уже использует Laravel Passport, Laravel MCP должен работать внутри существующей установки Passport. Однако custom scopes сейчас не поддерживаются, так как OAuth в основном используется как слой трансляции к underlying authenticatable model.

Метод `Mcp::oauthRoutes` добавляет, объявляет и использует один scope `mcp:use`.

#### Passport или Sanctum

OAuth 2.1 является документированным механизмом аутентификации в спецификации Model Context Protocol и наиболее широко поддерживается MCP-клиентами. Поэтому по возможности рекомендуется использовать Passport.

Если приложение уже использует [Sanctum](/docs/{{version}}/sanctum), добавление Passport может быть избыточным. В таком случае можно использовать Sanctum без Passport до тех пор, пока не появится явная необходимость в MCP-клиенте, поддерживающем только OAuth.

<a name="sanctum"></a>
### Sanctum

Чтобы защитить MCP-сервер через [Sanctum](/docs/{{version}}/sanctum), добавьте middleware аутентификации Sanctum к серверу в `routes/ai.php`. MCP-клиенты должны передавать заголовок `Authorization: Bearer <token>`:

```php
use App\Mcp\Servers\WeatherExample;
use Laravel\Mcp\Facades\Mcp;

Mcp::web('/mcp/demo', WeatherExample::class)
    ->middleware('auth:sanctum');
```

<a name="custom-mcp-authentication"></a>
#### Пользовательская MCP-аутентификация

Если приложение выпускает собственные API tokens, назначьте любой нужный middleware маршрутам `Mcp::web`. Ваш middleware может вручную читать заголовок `Authorization` и аутентифицировать входящий MCP-запрос.

<a name="authorization"></a>
## Авторизация

Текущий аутентифицированный пользователь доступен через `$request->user()`, что позволяет выполнять [проверки авторизации](/docs/{{version}}/authorization) внутри MCP-инструментов и ресурсов:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

public function handle(Request $request): Response
{
    if (! $request->user()->can('read-weather')) {
        return Response::error('Permission denied.');
    }

    // ...
}
```

<a name="client"></a>
## MCP-клиент

Помимо создания серверов, Laravel MCP включает клиент для подключения к другим MCP-серверам - как официальным, так и сторонним. Клиент позволяет вашему приложению обнаруживать и вызывать инструменты, предоставленные MCP-сервером, что особенно полезно для предоставления вашим [AI-агентам](/docs/{{version}}/ai-sdk#mcp-tools) доступа к возможностям внешних MCP-серверов.

<a name="client-connecting"></a>
### Подключение к серверам

Подключиться к MCP-серверу, доступному по HTTP, можно методом `Client::web`, передав URL сервера:

```php
use Laravel\Mcp\Client;

$client = Client::web('https://mcp.example.com');
```

Чтобы подключиться к локальному MCP-серверу, который запускается командой, используйте метод `Client::local`, передав команду и необходимые arguments:

```php
use Laravel\Mcp\Client;

$client = Client::local('php', ['artisan', 'mcp:start']);
```

Client подключается лениво, автоматически устанавливая соединение при первом получении списка tools или вызове tool. Если нужно управлять соединением вручную, используйте методы `connect`, `connected`, `ping` и `disconnect`:

```php
$client->connect();

$client->ping();

if ($client->connected()) {
    // ...
}

$client->disconnect();
```

Timeout request можно настроить методом `withTimeout`:

```php
$client = Client::web('https://mcp.example.com')->withTimeout(30);
```

<a name="named-clients"></a>
### Именованные clients

Вместо создания client каждый раз, когда он нужен, можно зарегистрировать переиспользуемые именованные clients. Обычно это делают в методе `boot` service provider с помощью facade `Mcp`:

```php
use Laravel\Mcp\Client;
use Laravel\Mcp\Facades\Mcp;

Mcp::registerClient('github', fn () => Client::web('https://mcp.example.com'));
```

После регистрации client можно получить по имени в любом месте приложения:

```php
use Laravel\Mcp\Facades\Mcp;

$client = Mcp::client('github');
```

Именованные clients разрешаются один раз за request и автоматически отключаются в конце request lifecycle.

<a name="client-authentication"></a>
### Аутентификация client

Для подключения к web MCP-серверу, защищенному bearer token, используйте метод `withToken`. Можно передать строку token или closure, который лениво разрешит token:

```php
use Illuminate\Support\Facades\Auth;
use Laravel\Mcp\Client;

$client = Client::web('https://mcp.example.com')->withToken($token);

$client = Client::web('https://mcp.example.com')->withToken(
    fn () => Auth::user()->mcpToken(),
);
```

Для серверов, защищенных [OAuth 2.1](#oauth), настройте client с помощью метода `withOAuth`. Это client-side аналог защиты ваших собственных серверов через OAuth:

```php
use Laravel\Mcp\Client;
use Laravel\Mcp\Facades\Mcp;

Mcp::registerClient('github', fn () => Client::web('https://mcp.example.com')->withOAuth(
    clientId: config('services.github_mcp.client_id'),
    clientSecret: config('services.github_mcp.client_secret'),
));
```

> [!NOTE]
> Аргументы `clientId` и `clientSecret` можно опустить, когда MCP-сервер поддерживает [dynamic client registration](https://datatracker.ietf.org/doc/html/rfc7591); в этом случае client зарегистрирует себя автоматически.

Затем зарегистрируйте OAuth routes для именованного client в файле `routes/ai.php`, используя метод `oAuthRoutesFor`. Переданный closure получает имя client и итоговый `TokenSet` после обмена authorization code на access token:

```php
use Illuminate\Support\Facades\Auth;
use Laravel\Mcp\Client\OAuth\TokenSet;
use Laravel\Mcp\Facades\Mcp;

Mcp::oAuthRoutesFor('github', function (string $client, TokenSet $token) {
    Auth::user()->update([
        'github_mcp_token' => $token->accessToken,
    ]);

    return redirect('/dashboard');
});
```

Это зарегистрирует два именованных route: connect route (`mcp.oauth.{client}.connect`), который перенаправляет пользователя на authorization server, и callback route (`mcp.oauth.{client}.callback`), который обменивает authorization code и вызывает ваш handler. Оба route по умолчанию используют middleware group `web`; ее можно переопределить через аргумент `middleware`.

Чтобы начать authorization flow, перенаправьте пользователя на connect route:

```php
return redirect()->route('mcp.oauth.github.connect');
```

<a name="client-tools"></a>
### Tools

Получить tools, предоставленные MCP-сервером, можно методом `tools`, который возвращает collection tools с ключами по имени:

```php
use Laravel\Mcp\Facades\Mcp;

$tools = Mcp::client('github')->tools();

foreach ($tools as $tool) {
    $tool->name;
    $tool->title;
    $tool->description;
    $tool->inputSchema;
}
```

Client автоматически проходит все страницы доступных tools. Ограничить количество возвращаемых tools можно аргументом `limit`:

```php
$tools = Mcp::client('github')->tools(limit: 10);
```

Чтобы вызвать tool, используйте метод `callTool`, передав имя tool и массив arguments. Возвращаемый экземпляр `ToolResult` предоставляет доступ к response tool:

```php
use Laravel\Mcp\Facades\Mcp;

$result = Mcp::client('github')->callTool('current-weather', [
    'location' => 'New York',
]);

$result->text(); // The text content of the response...
(string) $result; // Equivalent to calling text()...
$result->isError; // Whether the tool reported an error...
$result->structuredContent;  // Structured content, if any...
```

Также tool можно вызвать напрямую из экземпляра, полученного в списке:

```php
$tools = Mcp::client('github')->tools();

$result = $tools['current-weather']->call([
    'location' => 'New York',
]);
```

Если вы создаете agents с помощью [Laravel AI SDK](/docs/{{version}}/ai-sdk), tools из MCP client можно передавать напрямую agent, позволяя модели вызывать их при ответе на prompt. Подробнее смотрите раздел [MCP Tools](/docs/{{version}}/ai-sdk#mcp-tools) документации AI SDK.

<a name="client-prompts"></a>
### Prompts

Получить prompts, предоставленные MCP-сервером, можно методом `prompts`, который возвращает collection prompts с ключами по имени:

```php
use Laravel\Mcp\Facades\Mcp;

$prompts = Mcp::client('github')->prompts();

foreach ($prompts as $prompt) {
    $prompt->name;
    $prompt->title;
    $prompt->description;
    $prompt->arguments;
}
```

Client автоматически проходит все страницы доступных prompts. Ограничить количество возвращаемых prompts можно аргументом `limit`:

```php
$prompts = Mcp::client('github')->prompts(limit: 10);
```

Чтобы получить prompt, используйте метод `getPrompt`, передав имя prompt и массив arguments. Возвращаемый экземпляр `PromptResult` предоставляет доступ к сгенерированным messages:

```php
use Laravel\Mcp\Facades\Mcp;

$result = Mcp::client('github')->getPrompt('describe-weather', [
    'location' => 'New York',
]);

$result->text(); // The text content of the messages...
(string) $result; // Equivalent to calling text()...
$result->messages; // The raw messages returned by the prompt...
$result->description; // The prompt description, if any...
```

<a name="client-resources"></a>
### Resources

Получить resources, предоставленные MCP-сервером, можно методом `resources`, который возвращает collection resources с ключами по URI:

```php
use Laravel\Mcp\Facades\Mcp;

$resources = Mcp::client('github')->resources();

foreach ($resources as $resource) {
    $resource->uri;
    $resource->name;
    $resource->title;
    $resource->description;
    $resource->mimeType;
    $resource->size;
}
```

Client автоматически проходит все страницы доступных resources. Ограничить количество возвращаемых resources можно аргументом `limit`:

```php
$resources = Mcp::client('github')->resources(limit: 10);
```

Чтобы прочитать resource, используйте метод `readResource`, передав URI resource. Возвращаемый экземпляр `ResourceReadResult` предоставляет доступ к content resource:

```php
use Laravel\Mcp\Facades\Mcp;

$result = Mcp::client('github')->readResource('weather://guidelines');

$result->content(); // The content of the resource, decoding base64 blobs as needed...
(string) $result; // Equivalent to calling content()...
$result->mimeType(); // The MIME type of the resource, if any...
$result->contents; // The raw contents returned by the resource...
```

<a name="testing-servers"></a>
## Тестирование серверов

MCP-серверы можно тестировать через встроенный MCP Inspector или unit-тесты.

<a name="mcp-inspector"></a>
### MCP Inspector

[MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector) - интерактивный инструмент для тестирования и отладки MCP-серверов. Он позволяет подключиться к серверу, проверить аутентификацию и попробовать tools, resources и prompts.

Inspector можно запустить для любого зарегистрированного сервера:

```shell
# Web server...
php artisan mcp:inspector mcp/weather

# Local server named "weather"...
php artisan mcp:inspector weather
```

Команда запускает MCP Inspector и выводит client settings, которые можно скопировать в MCP-клиент. Если web-сервер защищен middleware аутентификации, укажите необходимые headers, например bearer token `Authorization`.

<a name="unit-tests"></a>
### Unit-тесты

Можно писать unit-тесты для MCP-серверов, tools, resources и prompts. Создайте test case и вызовите нужный primitive на сервере, где он зарегистрирован. Например, tool на `WeatherServer`:

```php tab=Pest
test('tool', function () {
    $response = WeatherServer::tool(CurrentWeatherTool::class, [
        'location' => 'New York City',
        'units' => 'fahrenheit',
    ]);

    $response
        ->assertOk()
        ->assertSee('The current weather in New York City is 72°F and sunny.');
});
```

```php tab=PHPUnit
/**
 * Test a tool.
 */
public function test_tool(): void
{
    $response = WeatherServer::tool(CurrentWeatherTool::class, [
        'location' => 'New York City',
        'units' => 'fahrenheit',
    ]);

    $response
        ->assertOk()
        ->assertSee('The current weather in New York City is 72°F and sunny.');
}
```

Prompts и resources тестируются аналогично:

```php
$response = WeatherServer::prompt(...);
$response = WeatherServer::resource(...);
```

Чтобы действовать от имени аутентифицированного пользователя, используйте `actingAs` перед вызовом primitive:

```php
$response = WeatherServer::actingAs($user)->tool(...);
```

После получения response можно использовать различные assertion methods для проверки содержимого и статуса response.

Метод `assertOk` проверяет, что response успешен и не содержит ошибок:

```php
$response->assertOk();
```

Метод `assertSee` проверяет, что response содержит конкретный текст:

```php
$response->assertSee('The current weather in New York City is 72°F and sunny.');
```

Метод `assertHasErrors` проверяет, что response содержит ошибку:

```php
$response->assertHasErrors();

$response->assertHasErrors([
    'Something went wrong.',
]);
```

Метод `assertHasNoErrors` проверяет, что response не содержит ошибок:

```php
$response->assertHasNoErrors();
```

Metadata проверяется методами `assertName()`, `assertTitle()` и `assertDescription()`:

```php
$response->assertName('current-weather');
$response->assertTitle('Current Weather Tool');
$response->assertDescription('Fetches the current weather forecast for a specified location.');
```

Notifications проверяются через `assertSentNotification` и `assertNotificationCount`:

```php
$response->assertSentNotification('processing/progress', [
    'step' => 1,
    'total' => 5,
]);

$response->assertSentNotification('processing/progress', [
    'step' => 2,
    'total' => 5,
]);

$response->assertNotificationCount(5);
```

Для отладки raw response content используйте `dd` или `dump`:

```php
$response->dd();
$response->dump();
```
