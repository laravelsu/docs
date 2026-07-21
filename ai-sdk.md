---
git: 7e0ec215e19fda46743ef7eaca7f83c2916db281
---

# Laravel AI SDK

- [Введение](#introduction)
- [Установка](#installation)
    - [Конфигурирование](#configuration)
    - [Пользовательские базовые URL](#custom-base-urls)
    - [OpenAI-совместимые провайдеры](#openai-compatible-providers)
    - [Поддержка провайдеров](#provider-support)
- [Агенты](#agents)
    - [Запросы к агентам](#prompting)
    - [Контекст разговора](#conversation-context)
    - [Структурированный вывод](#structured-output)
    - [Вложения](#attachments)
    - [Потоковая передача](#streaming)
    - [Broadcasting](#broadcasting)
    - [Очереди](#queueing)
    - [Инструменты](#tools)
    - [Инструменты файлового хранилища](#file-storage-tools)
    - [MCP Tools](#mcp-tools)
    - [Инструменты провайдеров](#provider-tools)
    - [Sub-agents](#sub-agents)
    - [Middleware](#middleware)
    - [Анонимные агенты](#anonymous-agents)
    - [Конфигурация агента](#agent-configuration)
    - [Опции провайдера](#provider-options)
- [Изображения](#images)
- [Аудио (TTS)](#audio)
- [Транскрипции (STT)](#transcription)
- [Embeddings](#embeddings)
    - [Запросы по embeddings](#querying-embeddings)
    - [Кеширование embeddings](#caching-embeddings)
- [Реранжирование](#reranking)
- [Файлы](#files)
- [Vector stores](#vector-stores)
    - [Добавление файлов в хранилища](#adding-files-to-stores)
- [Failover](#failover)
- [Тестирование](#testing)
    - [Агенты](#testing-agents)
    - [Изображения](#testing-images)
    - [Аудио](#testing-audio)
    - [Транскрипции](#testing-transcriptions)
    - [Embeddings](#testing-embeddings)
    - [Реранжирование](#testing-reranking)
    - [Файлы](#testing-files)
    - [Vector stores](#testing-vector-stores)
- [События](#events)

<a name="introduction"></a>
## Введение

[Laravel AI SDK](https://github.com/laravel/ai) предоставляет единый выразительный API для работы с AI-провайдерами, такими как OpenAI, Anthropic, Gemini и другими. С AI SDK можно создавать интеллектуальных агентов с инструментами и структурированным выводом, генерировать изображения, синтезировать и транскрибировать аудио, создавать vector embeddings и многое другое через последовательный Laravel-friendly интерфейс.

<a name="installation"></a>
## Установка

Установите Laravel AI SDK через Composer:

```shell
composer require laravel/ai
```

Затем опубликуйте конфигурацию и миграции AI SDK с помощью Artisan-команды `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
```

После этого выполните миграции приложения. Они создадут таблицы `agent_conversations` и `agent_conversation_messages`, которые AI SDK использует для хранения разговоров:

```shell
php artisan migrate
```

<a name="configuration"></a>
### Конфигурирование

Учетные данные AI-провайдеров можно определить в файле `config/ai.php` или через переменные окружения в `.env`:

```ini
ANTHROPIC_API_KEY=
AZURE_OPENAI_API_KEY=
COHERE_API_KEY=
DEEPSEEK_API_KEY=
ELEVENLABS_API_KEY=
GEMINI_API_KEY=
GROQ_API_KEY=
MISTRAL_API_KEY=
OLLAMA_API_KEY=
OPENAI_API_KEY=
OPENAI_COMPATIBLE_API_KEY=
OPENAI_COMPATIBLE_URL=
OPENROUTER_API_KEY=
JINA_API_KEY=
VOYAGEAI_API_KEY=
XAI_API_KEY=
```

Модели по умолчанию для текста, изображений, аудио, транскрипций и embeddings также настраиваются в `config/ai.php`.

<a name="custom-base-urls"></a>
### Пользовательские базовые URL

По умолчанию Laravel AI SDK подключается напрямую к публичному API-эндпоинту каждого провайдера. Иногда запросы нужно направлять через другой эндпоинт, например через прокси-сервис для централизованного управления API-ключами, ограничения частоты запросов или корпоративный шлюз.

Пользовательские базовые URL настраиваются через параметр `url` в конфигурации провайдера:

```php
'providers' => [
    'openai' => [
        'driver' => 'openai',
        'key' => env('OPENAI_API_KEY'),
        'url' => env('OPENAI_URL'),
    ],

    'anthropic' => [
        'driver' => 'anthropic',
        'key' => env('ANTHROPIC_API_KEY'),
        'url' => env('ANTHROPIC_BASE_URL'),
    ],
],
```

Это полезно при маршрутизации запросов через прокси-сервисы вроде LiteLLM или Azure OpenAI Gateway, а также при использовании альтернативных эндпоинтов. Пользовательские URL поддерживаются для OpenAI, Anthropic, Gemini, Groq, Cohere, DeepSeek, xAI и OpenRouter.

<a name="openai-compatible-providers"></a>
### OpenAI-совместимые провайдеры

Если вы используете OpenAI-совместимый API, например LM Studio, vLLM, Together, Fireworks или локальный шлюз, вы можете настроить провайдер `openai-compatible`. Опция `url` обязательна, а опция `key` необязательна и при наличии будет отправляться как bearer token:

```php
'providers' => [
    'local' => [
        'driver' => 'openai-compatible',
        'url' => env('LOCAL_AI_URL'),
        'key' => env('LOCAL_AI_API_KEY'),
    ],
],
```

После настройки именованный провайдер можно использовать как любой другой:

```php
agent()->prompt('What is Laravel?', provider: 'local', model: 'local-model');
```

Вы также можете настроить модель по умолчанию для текста, чтобы не передавать модель явно:

```php
'local' => [
    'driver' => 'openai-compatible',
    'url' => env('LOCAL_AI_URL'),
    'key' => env('LOCAL_AI_API_KEY'),
    'models' => [
        'text' => [
            'default' => env('LOCAL_AI_MODEL'),
        ],
    ],
],
```

OpenAI-совместимые провайдеры поддерживают генерацию текста, потоковую передачу, tools, structured output и вложения изображений. Если вашему эндпоинту требуются дополнительные поля тела запроса, передайте их с помощью [опций провайдера](#provider-options).

<a name="provider-support"></a>
### Поддержка провайдеров

AI SDK поддерживает разных провайдеров для разных возможностей:

<div class="overflow-auto">

| Возможность | Провайдеры |
|---|---|
| Text | OpenAI, OpenAI Compatible, Anthropic, Gemini, Azure, Bedrock, Groq, xAI, DeepSeek, Mistral, Ollama, OpenRouter |
| Images | OpenAI, Gemini, xAI, Azure, Bedrock, OpenRouter |
| TTS | OpenAI, ElevenLabs, Gemini |
| STT | OpenAI, ElevenLabs, Mistral, Gemini |
| Embeddings | OpenAI, Gemini, Azure, Bedrock, Cohere, Mistral, Jina, VoyageAI, Ollama, OpenRouter |
| Reranking | Cohere, Jina, VoyageAI |
| Files | OpenAI, Anthropic, Gemini, Azure |

</div>

Enum `Laravel\Ai\Enums\Lab` можно использовать для ссылки на провайдеров в коде вместо строк:

```php
use Laravel\Ai\Enums\Lab;

Lab::Anthropic;
Lab::OpenAI;
Lab::OpenAiCompatible;
Lab::Gemini;
// ...
```

<a name="agents"></a>
## Агенты

Агенты - основной строительный блок для взаимодействия с AI-провайдерами в Laravel AI SDK. Каждый агент является отдельным PHP-классом, инкапсулирующим инструкции, контекст разговора, инструменты и схему вывода, необходимые для работы с большой языковой моделью. Думайте об агенте как о специализированном помощнике: sales coach, анализатор документов, support bot, которого вы настраиваете один раз и затем вызываете по мере необходимости.

Агента можно создать Artisan-командой `make:agent`:

```shell
php artisan make:agent SalesCoach

php artisan make:agent SalesCoach --structured
```

В сгенерированном классе можно определить system prompt / instructions, контекст сообщений, доступные tools и схему вывода:

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\RetrievePreviousTranscripts;
use App\Models\History;
use App\Models\User;
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Messages\Message;
use Laravel\Ai\Promptable;
use Stringable;

class SalesCoach implements Agent, Conversational, HasTools, HasStructuredOutput
{
    use Promptable;

    public function __construct(public User $user) {}

    public function instructions(): Stringable|string
    {
        return 'You are a sales coach, analyzing transcripts and providing feedback and an overall sales strength score.';
    }

    public function messages(): iterable
    {
        return History::where('user_id', $this->user->id)
            ->latest()
            ->limit(50)
            ->get()
            ->reverse()
            ->map(function ($message) {
                return new Message($message->role, $message->content);
            })->all();
    }

    public function tools(): iterable
    {
        return [
            new RetrievePreviousTranscripts,
        ];
    }

    public function schema(JsonSchema $schema): array
    {
        return [
            'feedback' => $schema->string()->required(),
            'score' => $schema->integer()->min(1)->max(10)->required(),
        ];
    }
}
```

<a name="prompting"></a>
### Запросы к агентам

Чтобы обратиться к агенту, создайте экземпляр через `make` или обычный конструктор, затем вызовите `prompt`:

```php
$response = (new SalesCoach)
    ->prompt('Analyze this sales transcript...');

return (string) $response;
```

Метод `make` разрешает агента из контейнера, поэтому работает автоматический dependency injection. В конструктор агента также можно передавать аргументы:

```php
$agent = SalesCoach::make(user: $user);
```

Передав дополнительные аргументы в `prompt`, можно переопределить `provider`, `model` или HTTP timeout:

```php
$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: Lab::Anthropic,
    model: 'claude-haiku-4-5-20251001',
    timeout: 120,
);
```

<a name="conversation-context"></a>
### Контекст разговора

Если агент реализует интерфейс `Conversational`, метод `messages` может возвращать предыдущий контекст разговора:

```php
use App\Models\History;
use Laravel\Ai\Messages\Message;

public function messages(): iterable
{
    return History::where('user_id', $this->user->id)
        ->latest()
        ->limit(50)
        ->get()
        ->reverse()
        ->map(function ($message) {
            return new Message($message->role, $message->content);
        })->all();
}
```

<a name="remembering-conversations"></a>
#### Запоминание разговоров

> **Note:** Перед использованием трейта `RemembersConversations` опубликуйте и выполните миграции AI SDK через `vendor:publish`, чтобы создать таблицы для хранения разговоров.

Если вы хотите, чтобы Laravel автоматически сохранял и извлекал историю разговоров агента, используйте трейт `RemembersConversations`. Он позволяет сохранять сообщения в базе данных без ручной реализации `Conversational`:

```php
use Laravel\Ai\Concerns\RemembersConversations;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, Conversational
{
    use Promptable, RemembersConversations;

    public function instructions(): string
    {
        return 'You are a sales coach...';
    }
}
```

При использовании trait `RemembersConversations` не определяйте метод `messages` вручную в классе агента. Если метод `messages` присутствует, он получит приоритет над реализацией trait, и history разговора не будет загружена из базы данных.

Чтобы начать новый разговор для пользователя, вызовите `forUser` перед `prompt`:

```php
$response = (new SalesCoach)->forUser($user)->prompt('Hello!');

$conversationId = $response->conversationId;
```

ID разговора возвращается в response и может быть сохранен для дальнейшего использования. Если вы хотите получать все conversations пользователя через Eloquent, добавьте trait `HasConversations` к вашей user model:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Ai\Concerns\HasConversations;

class User extends Authenticatable
{
    use HasConversations;
}
```

После добавления trait к модели conversations пользователя можно получать и запрашивать через relationship `conversations`:

```php
$conversations = $user->conversations()
    ->latest('updated_at')
    ->paginate(20);
```

Продолжить существующий разговор можно методом `continue`:

```php
$response = (new SalesCoach)
    ->continue($conversationId, as: $user)
    ->prompt('Tell me more about that.');
```

При использовании `RemembersConversations` предыдущие сообщения автоматически загружаются и включаются в контекст при prompt. Новые сообщения пользователя и ассистента сохраняются после каждого взаимодействия.

<a name="structured-output"></a>
### Структурированный вывод

Если агент должен возвращать структурированный вывод, реализуйте интерфейс `HasStructuredOutput` и определите метод `schema`:

```php
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasStructuredOutput
{
    use Promptable;

    public function schema(JsonSchema $schema): array
    {
        return [
            'score' => $schema->integer()->required(),
        ];
    }
}
```

Ответ `StructuredAgentResponse` можно читать как массив:

```php
$response = (new SalesCoach)->prompt('Analyze this sales transcript...');

return $response['score'];
```

<a name="structured-output-nested-objects"></a>
#### Nested objects

Чтобы определить nested structured output, используйте метод `object` с closure:

```php
<?php

namespace App\Ai\Agents;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasStructuredOutput
{
    use Promptable;

    // ...

    /**
     * Get the agent's structured output schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'score' => $schema->integer()->required(),
            'metadata' => $schema->object(fn ($schema) => [
                'confidence' => $schema->string()->enum(['low', 'medium', 'high'])->required(),
                'language' => $schema->string()->required(),
            ])->required(),
        ];
    }
}
```

<a name="structured-output-arrays-of-objects"></a>
#### Arrays of objects

Если agent должен вернуть список structured items, объедините методы `array` и `object`:

```php
public function schema(JsonSchema $schema): array
{
    return [
        'feedback' => $schema->array()
            ->items(
                $schema->object(fn ($schema) => [
                    'comment' => $schema->string()->required(),
                    'score' => $schema->integer()->required(),
                ])
            )
            ->required(),
    ];
}
```

Если значение может соответствовать одной из нескольких schemas, используйте метод `anyOf`:

```php
public function schema(JsonSchema $schema): array
{
    return [
        'content' => $schema->anyOf([
            $schema->object(fn ($schema) => [
                'type' => $schema->string()->enum(['article'])->required(),
                'title' => $schema->string()->required(),
            ]),
            $schema->object(fn ($schema) => [
                'type' => $schema->string()->enum(['image'])->required(),
                'url' => $schema->string()->required(),
            ]),
        ])->required(),
    ];
}
```

<a name="attachments"></a>
### Вложения

При prompt можно передать вложения, чтобы модель могла анализировать изображения и документы:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Analyze the attached sales transcript...',
    attachments: [
        Files\Document::fromStorage('transcript.pdf'),
        Files\Document::fromPath('/home/laravel/transcript.md'),
        $request->file('transcript'),
    ]
);
```

Класс `Laravel\Ai\Files\Image` используется для прикрепления изображений:

```php
use App\Ai\Agents\ImageAnalyzer;
use Laravel\Ai\Files;

$response = (new ImageAnalyzer)->prompt(
    'What is in this image?',
    attachments: [
        Files\Image::fromStorage('photo.jpg'),
        Files\Image::fromPath('/home/laravel/photo.jpg'),
        $request->file('photo'),
    ]
);
```

<a name="streaming"></a>
### Потоковая передача

Ответ агента можно передавать потоком через метод `stream`. Возвращаемый `StreamableAgentResponse` можно вернуть из маршрута, чтобы автоматически отправить клиенту streaming response (SSE):

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)->stream('Analyze this sales transcript...');
});
```

Метод `then` позволяет выполнить замыкание после завершения потока:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Responses\StreamedAgentResponse;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Analyze this sales transcript...')
        ->then(function (StreamedAgentResponse $response) {
            // $response->text, $response->events, $response->usage...
        });
});
```

Также можно вручную итерироваться по streamed events:

```php
$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    // ...
}
```

<a name="streaming-using-the-vercel-ai-sdk-protocol"></a>
#### Потоковая передача через протокол Vercel AI SDK

Чтобы передавать события через [stream protocol Vercel AI SDK](https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol), вызовите `usingVercelDataProtocol`:

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Analyze this sales transcript...')
        ->usingVercelDataProtocol();
});
```

<a name="broadcasting"></a>
### Broadcasting

Streamed events можно транслировать несколькими способами. Например, вызвать `broadcast` или `broadcastNow` на streamed event:

```php
use App\Ai\Agents\SalesCoach;
use Illuminate\Broadcasting\Channel;

$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    $event->broadcast(new Channel('channel-name'));
}
```

Или вызвать `broadcastOnQueue`, чтобы поставить операцию агента в очередь и транслировать streamed events по мере готовности:

```php
(new SalesCoach)->broadcastOnQueue(
    'Analyze this sales transcript...',
    new Channel('channel-name'),
);
```

<a name="skipping-oversized-events"></a>
#### Пропуск слишком больших событий

Некоторые broadcasting-платформы ограничивают WebSocket-сообщения примерно 10KB. События stream с большим объемом данных, например крупные результаты tools, могут превысить этот лимит и привести к ошибке broadcasting. Вы можете исключить определенные типы событий из broadcasting с помощью атрибута `WithoutBroadcasting`:

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Attributes\WithoutBroadcasting;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Promptable;
use Laravel\Ai\Streaming\Events\ToolCall;
use Laravel\Ai\Streaming\Events\ToolResult;

#[WithoutBroadcasting(ToolCall::class, ToolResult::class)]
class SearchAgent implements Agent, HasTools
{
    use Promptable;

    // ...
}
```

Исключенные события никогда не транслируются, но по-прежнему записываются в таблицу `agent_conversation_messages`, поэтому ваш фронтенд сможет загрузить полные данные tool после завершения stream. Это работает как для queued broadcasting (`broadcastOnQueue`), так и для синхронного broadcasting (`broadcast` / `broadcastNow`).

<a name="queueing"></a>
### Очереди

Метод `queue` позволяет отправить prompt агенту и обработать ответ в фоне, сохраняя приложение быстрым и отзывчивым. Методы `then` и `catch` регистрируют callbacks, которые вызываются при готовом ответе или исключении:

```php
use Illuminate\Http\Request;
use Laravel\Ai\Responses\AgentResponse;
use Throwable;

Route::post('/coach', function (Request $request) {
    (new SalesCoach)
        ->queue($request->input('transcript'))
        ->then(function (AgentResponse $response) {
            // ...
        })
        ->catch(function (Throwable $e) {
            // ...
        });

    return back();
});
```

<a name="tools"></a>
### Инструменты

Tools дают агентам дополнительные возможности, которыми они могут пользоваться при ответе. Tool создается Artisan-командой `make:tool`:

```shell
php artisan make:tool RandomNumberGenerator
```

Сгенерированный tool будет размещен в `app/Ai/Tools`. Каждый tool содержит метод `handle`, вызываемый агентом при необходимости:

```php
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Tool;
use Laravel\Ai\Tools\Request;
use Stringable;

class RandomNumberGenerator implements Tool
{
    public function description(): Stringable|string
    {
        return 'This tool may be used to generate cryptographically secure random numbers.';
    }

    public function handle(Request $request): Stringable|string
    {
        return (string) random_int($request['min'], $request['max']);
    }

    public function schema(JsonSchema $schema): array
    {
        return [
            'min' => $schema->integer()->min(0)->required(),
            'max' => $schema->integer()->required(),
        ];
    }
}
```

Tool возвращается из метода `tools` любого агента:

```php
use App\Ai\Tools\RandomNumberGenerator;

public function tools(): iterable
{
    return [
        new RandomNumberGenerator,
    ];
}
```

<a name="similarity-search"></a>
#### Similarity Search

Tool `SimilaritySearch` позволяет агентам искать документы, похожие на запрос, используя vector embeddings из базы данных. Это удобно для retrieval-augmented generation (RAG), когда агенту нужен доступ к данным приложения.

Самый простой способ создать similarity search tool - метод `usingModel` с Eloquent-моделью, имеющей vector embeddings:

```php
use App\Models\Document;
use Laravel\Ai\Tools\SimilaritySearch;

public function tools(): iterable
{
    return [
        SimilaritySearch::usingModel(Document::class, 'embedding'),
    ];
}
```

Первый аргумент - класс Eloquent-модели, второй - столбец с vector embeddings. Можно указать минимальный порог сходства, лимит и замыкание для настройки запроса:

```php
SimilaritySearch::usingModel(
    model: Document::class,
    column: 'embedding',
    minSimilarity: 0.7,
    limit: 10,
    query: fn ($query) => $query->where('published', true),
),
```

Для полного контроля создайте tool с пользовательским замыканием:

```php
use App\Models\Document;
use Laravel\Ai\Tools\SimilaritySearch;

public function tools(): iterable
{
    return [
        new SimilaritySearch(using: function (string $query) {
            return Document::query()
                ->where('user_id', $this->user->id)
                ->whereVectorSimilarTo('embedding', $query)
                ->limit(10)
                ->get();
        }),
    ];
}
```

Описание tool можно настроить методом `withDescription`:

```php
SimilaritySearch::usingModel(Document::class, 'embedding')
    ->withDescription('Search the knowledge base for relevant articles.'),
```

<a name="file-storage-tools"></a>
### Инструменты файлового хранилища

Фабрика tools `FileStorage` позволяет предоставить agents доступ к [диску файловой системы](/docs/{{version}}/filesystem) Laravel. Метод `all` возвращает tools, которые позволяют agent просматривать список файлов, читать, инспектировать, генерировать URL, записывать, удалять и копировать файлы на указанном диске:

```php
use Laravel\Ai\Tools\FileStorage;

public function tools(): iterable
{
    return FileStorage::all('local');
}
```

Если agent должен иметь возможность только инспектировать файлы, используйте метод `readOnly`:

```php
return FileStorage::readOnly('local');
```

Эти методы возвращают `Illuminate\Support\Collection`, что позволяет дополнительно фильтровать tools, предоставляемые agent:

```php
use Laravel\Ai\Tools\Filesystem\DeleteFile;

return FileStorage::all('s3')
    ->reject(fn ($tool) => $tool instanceof DeleteFile);
```

<a name="mcp-tools"></a>
### MCP Tools

Если приложение использует [Laravel MCP](/docs/{{version}}/mcp), вы можете предоставить agents tools, опубликованные серверами [Model Context Protocol](https://modelcontextprotocol.io). С помощью [Laravel MCP client](/docs/{{version}}/mcp#client) можно подключиться к remote или local MCP server и передать его tools напрямую agent.

> [!NOTE]
> MCP tools требуют, чтобы в приложении был установлен package [Laravel MCP](/docs/{{version}}/mcp).

Поскольку метод `tools` MCP client возвращает collection, разверните ее в массив `tools` вашего agent с помощью оператора `...`:

```php
use App\Ai\Tools\RandomNumberGenerator;
use Laravel\Mcp\Client;

/**
 * Get the tools available to the agent.
 *
 * @return Tool[]
 */
public function tools(): iterable
{
    return [
        ...Client::web('https://mcp.example.com')
            ->withToken($token)
            ->tools(),

        new RandomNumberGenerator,
    ];
}
```

AI SDK автоматически оборачивает каждый MCP tool, чтобы agent мог вызывать его как любой другой tool. Также можно использовать [именованный MCP client](/docs/{{version}}/mcp#named-clients):

```php
use Laravel\Mcp\Facades\Mcp;

public function tools(): iterable
{
    return [
        ...Mcp::client('github')->tools(),
    ];
}
```

Или подключиться к [локальному MCP server](/docs/{{version}}/mcp#client-connecting):

```php
use Laravel\Mcp\Client;

public function tools(): iterable
{
    return [
        ...Client::local('php', ['artisan', 'mcp:start'])->tools(),
    ];
}
```

Подробнее о создании и аутентификации MCP-клиентов, включая bearer-токены и OAuth, смотрите в [документации MCP-клиента](/docs/{{version}}/mcp#client).

<a name="provider-tools"></a>
### Инструменты провайдеров

Инструменты провайдеров - специальные инструменты, нативно реализованные AI-провайдерами. Они дают возможности вроде веб-поиска, получения содержимого URL и поиска по файлам. В отличие от обычных инструментов, инструменты провайдеров выполняются самим провайдером, а не вашим приложением, и возвращаются из метода `tools` агента.

<a name="web-search"></a>
#### Web Search

Provider tool `WebSearch` позволяет агентам искать в интернете актуальную информацию. Это полезно для вопросов о текущих событиях, свежих данных или темах, которые могли измениться после даты обучения модели.

**Поддерживаемые провайдеры:** Anthropic, OpenAI, Gemini, OpenRouter

```php
use Laravel\Ai\Providers\Tools\WebSearch;

public function tools(): iterable
{
    return [
        new WebSearch,
    ];
}
```

Поиск можно ограничить количеством запросов или доменами:

```php
(new WebSearch)->max(5)->allow(['laravel.com', 'php.net']),
```

Для уточнения результатов по местоположению используйте `location`:

```php
(new WebSearch)->location(
    city: 'New York',
    region: 'NY',
    country: 'US'
);
```

<a name="web-fetch"></a>
#### Web Fetch

Provider tool `WebFetch` позволяет агентам получать и читать содержимое веб-страниц. Это полезно, когда агент должен анализировать конкретные URL или получить подробную информацию с известных страниц.

**Поддерживаемые провайдеры:** Anthropic, Gemini

```php
use Laravel\Ai\Providers\Tools\WebFetch;

public function tools(): iterable
{
    return [
        new WebFetch,
    ];
}
```

Количество fetch-запросов и домены можно ограничить:

```php
(new WebFetch)->max(3)->allow(['docs.laravel.com']),
```

<a name="file-search"></a>
#### File Search

Provider tool `FileSearch` позволяет агентам искать по [файлам](#files), сохраненным в [vector stores](#vector-stores). Это включает RAG-сценарии, где агент ищет релевантную информацию в загруженных документах.

**Поддерживаемые провайдеры:** OpenAI, Gemini

```php
use Laravel\Ai\Providers\Tools\FileSearch;

public function tools(): iterable
{
    return [
        new FileSearch(stores: ['store_id']),
    ];
}
```

Можно указать несколько vector store IDs:

```php
new FileSearch(stores: ['store_1', 'store_2']);
```

Если у файлов есть [metadata](#adding-files-to-stores), результаты можно фильтровать через аргумент `where`:

```php
new FileSearch(stores: ['store_id'], where: [
    'author' => 'Taylor Otwell',
    'year' => 2026,
]);
```

Для сложных фильтров передайте замыкание с `FileSearchQuery`:

```php
use Laravel\Ai\Providers\Tools\FileSearchQuery;

new FileSearch(stores: ['store_id'], where: fn (FileSearchQuery $query) =>
    $query->where('author', 'Taylor Otwell')
        ->whereNot('status', 'draft')
        ->whereIn('category', ['news', 'updates'])
);
```

<a name="sub-agents"></a>
### Sub-agents

Agents также можно возвращать из метода `tools` другого agent. Когда agent возвращается как tool, parent agent может делегировать sub-agent конкретную задачу и использовать response sub-agent при ответе на исходный prompt. Это удобно, когда general-purpose agent должен иметь доступ к specialized agents со своими instructions, tools, model configuration или provider preferences.

Например, customer support agent может делегировать вопросы о refund eligibility отдельному refunds agent:

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Promptable;

class CustomerSupportAgent implements Agent, HasTools
{
    use Promptable;

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): string
    {
        return 'You help customers with account, order, and billing questions. Delegate refund policy questions to the refunds specialist.';
    }

    /**
     * Get the tools available to the agent.
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new RefundsAgent,
        ];
    }
}
```

Чтобы настроить, как sub-agent будет представлен parent agent, реализуйте interface `CanActAsTool` на sub-agent и определите name и description, видимые как tool:

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\LookupOrder;
use Laravel\Ai\Attributes\Provider;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\CanActAsTool;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

#[Provider(Lab::Anthropic)]
class RefundsAgent implements Agent, CanActAsTool, HasTools
{
    use Promptable;

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): string
    {
        return 'You are a refunds specialist. Use order details and the refund policy to give concise eligibility guidance.';
    }

    /**
     * Get the agent's tool name.
     */
    public function name(): string
    {
        return 'refunds_specialist';
    }

    /**
     * Get the agent's tool description.
     */
    public function description(): string
    {
        return 'Determine whether an order is eligible for a refund and explain the next step.';
    }

    /**
     * Get the tools available to the agent.
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new LookupOrder,
        ];
    }
}
```

Если sub-agent не реализует `CanActAsTool`, Laravel использует class basename agent как tool name и generic description, который просит parent agent передать clear, self-contained task description. Каждый вызов sub-agent выполняется изолированно и не получает conversation history parent agent.

<a name="middleware"></a>
### Middleware

Агенты поддерживают middleware, позволяя перехватывать и изменять prompts до отправки провайдеру. Middleware создается командой `make:agent-middleware`:

```shell
php artisan make:agent-middleware LogPrompts
```

Чтобы добавить middleware к агенту, реализуйте `HasMiddleware` и верните список middleware:

```php
use App\Ai\Middleware\LogPrompts;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasMiddleware;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasMiddleware
{
    use Promptable;

    public function middleware(): array
    {
        return [
            new LogPrompts,
        ];
    }
}
```

Middleware-класс определяет метод `handle`, получающий `AgentPrompt` и `Closure` для передачи запроса дальше:

```php
use Closure;
use Laravel\Ai\Prompts\AgentPrompt;

class LogPrompts
{
    public function handle(AgentPrompt $prompt, Closure $next)
    {
        Log::info('Prompting agent', ['prompt' => $prompt->prompt]);

        return $next($prompt);
    }
}
```

Метод `then` на response позволяет выполнить код после завершения обработки, как для синхронных, так и для streaming responses:

```php
public function handle(AgentPrompt $prompt, Closure $next)
{
    return $next($prompt)->then(function (AgentResponse $response) {
        Log::info('Agent responded', ['text' => $response->text]);
    });
}
```

<a name="anonymous-agents"></a>
### Анонимные агенты

Иногда нужно быстро обратиться к модели без отдельного класса агента. Для этого используйте функцию `agent`:

```php
use function Laravel\Ai\{agent};

$response = agent(
    instructions: 'You are an expert at software development.',
    messages: [],
    tools: [],
)->prompt('Tell me about Laravel')
```

Анонимные агенты также могут возвращать структурированный вывод:

```php
use Illuminate\Contracts\JsonSchema\JsonSchema;

use function Laravel\Ai\{agent};

$response = agent(
    schema: fn (JsonSchema $schema) => [
        'number' => $schema->integer()->required(),
    ],
)->prompt('Generate a random number less than 100')
```

<a name="agent-configuration"></a>
### Конфигурация агента

Параметры генерации текста можно задать через PHP-атрибуты:

- `MaxSteps`: максимальное количество шагов агента при использовании tools.
- `MaxTokens`: максимальное количество tokens, которое может сгенерировать модель.
- `Model`: модель агента.
- `Provider`: AI-провайдер или список провайдеров для failover.
- `Temperature`: sampling temperature для генерации (от `0.0` до `1.0`).
- `Timeout`: HTTP timeout в секундах (по умолчанию 60).
- `TopP`: nucleus sampling probability для генерации (от `0.0` до `1.0`).
- `UseCheapestModel`: использовать самую дешевую text-модель провайдера.
- `UseSmartestModel`: использовать самую мощную text-модель провайдера.

```php
use Laravel\Ai\Attributes\MaxSteps;
use Laravel\Ai\Attributes\MaxTokens;
use Laravel\Ai\Attributes\Model;
use Laravel\Ai\Attributes\Provider;
use Laravel\Ai\Attributes\Temperature;
use Laravel\Ai\Attributes\Timeout;
use Laravel\Ai\Attributes\TopP;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

#[Provider(Lab::Anthropic)]
#[Model('claude-haiku-4-5-20251001')]
#[MaxSteps(10)]
#[MaxTokens(4096)]
#[Temperature(0.7)]
#[Timeout(120)]
#[TopP(0.9)]
class SalesCoach implements Agent
{
    use Promptable;
}
```

Атрибуты `UseCheapestModel` и `UseSmartestModel` автоматически выбирают наиболее экономичную или мощную модель без явного имени модели:

```php
use Laravel\Ai\Attributes\UseCheapestModel;
use Laravel\Ai\Attributes\UseSmartestModel;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Promptable;

#[UseCheapestModel]
class SimpleSummarizer implements Agent
{
    use Promptable;
}

#[UseSmartestModel]
class ComplexReasoner implements Agent
{
    use Promptable;
}
```

> [!NOTE]
> Underlying model, выбранная `UseCheapestModel` и `UseSmartestModel`, может изменяться между релизами Laravel AI SDK по мере выхода новых моделей у провайдеров. Смена модели может приводить к behavioral changes, deprecated parameters и существенной разнице в стоимости. Если вам нужны стабильная, предсказуемая model и pricing, явно укажите model с помощью атрибута `Model`.

<a name="provider-options"></a>
### Опции провайдера

Если агенту нужно передать опции конкретного провайдера, например OpenAI reasoning effort или настройки штрафов, реализуйте `HasProviderOptions` и метод `providerOptions`:

```php
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasProviderOptions;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasProviderOptions
{
    use Promptable;

    public function providerOptions(Lab|string $provider): array
    {
        return match ($provider) {
            Lab::OpenAI => [
                'reasoning' => ['effort' => 'low'],
                'frequency_penalty' => 0.5,
                'presence_penalty' => 0.3,
            ],
            Lab::Anthropic => [
                'thinking' => ['budget_tokens' => 1024],
                'cache_control' => ['type' => 'ephemeral'],
            ],
            default => [],
        };
    }
}
```

Метод получает текущего провайдера (`Lab` enum или строку), поэтому можно возвращать разные опции для каждого провайдера. Это особенно полезно с [failover](#failover), где каждый fallback-провайдер может иметь свою конфигурацию.

Пример Anthropic выше также включает [prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) через `cache_control`.

<a name="images"></a>
## Изображения

Класс `Laravel\Ai\Image` используется для генерации изображений через провайдеры `openai`, `gemini` или `xai`:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

Методы `square`, `portrait` и `landscape` управляют aspect ratio, `quality` задает желаемое качество (`high`, `medium`, `low`), а `timeout` задает HTTP timeout:

```php
$image = Image::of('A donut sitting on the kitchen counter')
    ->quality('high')
    ->landscape()
    ->timeout(120)
    ->generate();
```

Можно прикреплять reference images:

```php
use Laravel\Ai\Files;
use Laravel\Ai\Image;

$image = Image::of('Update this photo of me to be in the style of an impressionist painting.')
    ->attachments([
        Files\Image::fromStorage('photo.jpg'),
        // Files\Image::fromPath('/home/laravel/photo.jpg'),
        // Files\Image::fromUrl('https://example.com/photo.jpg'),
        // $request->file('photo'),
    ])
    ->landscape()
    ->generate();
```

Сгенерированные изображения легко сохранить на default disk из `config/filesystems.php`:

```php
$image = Image::of('A donut sitting on the kitchen counter');

$path = $image->store();
$path = $image->storeAs('image.jpg');
$path = $image->storePublicly();
$path = $image->storePubliclyAs('image.jpg');
```

Генерацию изображений можно поставить в очередь:

```php
use Laravel\Ai\Image;
use Laravel\Ai\Responses\ImageResponse;

Image::of('A donut sitting on the kitchen counter')
    ->portrait()
    ->queue()
    ->then(function (ImageResponse $image) {
        $path = $image->store();
    });
```

<a name="audio"></a>
## Аудио

Класс `Laravel\Ai\Audio` генерирует аудио из текста:

```php
use Laravel\Ai\Audio;

$audio = Audio::of('I love coding with Laravel.')->generate();

$rawContent = (string) $audio;
```

Также можно сгенерировать аудио из строки с помощью метода `toAudio`, доступного через Laravel `Stringable` class:

```php
use Illuminate\Support\Str;

$audio = Str::of('I love coding with Laravel.')->toAudio();
```

Методы `male`, `female` и `voice` задают голос:

```php
$audio = Audio::of('I love coding with Laravel.')
    ->female()
    ->generate();

$audio = Audio::of('I love coding with Laravel.')
    ->voice('voice-id-or-name')
    ->generate();
```

Метод `instructions` позволяет динамически подсказать модели, как должно звучать аудио:

```php
$audio = Audio::of('I love coding with Laravel.')
    ->female()
    ->instructions('Said like a pirate')
    ->generate();
```

Аудио можно сохранить на default disk и генерировать через очередь:

```php
$audio = Audio::of('I love coding with Laravel.')->generate();

$path = $audio->store();
$path = $audio->storeAs('audio.mp3');
$path = $audio->storePublicly();
$path = $audio->storePubliclyAs('audio.mp3');
```

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Responses\AudioResponse;

Audio::of('I love coding with Laravel.')
    ->queue()
    ->then(function (AudioResponse $audio) {
        $path = $audio->store();
    });
```

<a name="transcription"></a>
## Транскрипции

Класс `Laravel\Ai\Transcription` создает transcript для аудио:

```php
use Laravel\Ai\Transcription;

$transcript = Transcription::fromPath('/home/laravel/audio.mp3')->generate();
$transcript = Transcription::fromStorage('audio.mp3')->generate();
$transcript = Transcription::fromUpload($request->file('audio'))->generate();

return (string) $transcript;
```

Метод `diarize` добавляет diarized transcript, сегментированный по говорящим:

```php
$transcript = Transcription::fromStorage('audio.mp3')
    ->diarize()
    ->generate();
```

Транскрипцию также можно поставить в очередь:

```php
use Laravel\Ai\Transcription;
use Laravel\Ai\Responses\TranscriptionResponse;

Transcription::fromStorage('audio.mp3')
    ->queue()
    ->then(function (TranscriptionResponse $transcript) {
        // ...
    });
```

<a name="embeddings"></a>
## Embeddings

Vector embeddings для строки можно сгенерировать через метод `toEmbeddings`, доступный в `Stringable`:

```php
use Illuminate\Support\Str;

$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

Для нескольких входов используйте класс `Embeddings`:

```php
use Laravel\Ai\Embeddings;

$response = Embeddings::for([
    'Napa Valley has great wine.',
    'Laravel is a PHP framework.',
])->generate();

$response->embeddings; // [[0.123, 0.456, ...], [0.789, 0.012, ...]]
```

Можно указать размеры и провайдера:

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->dimensions(1536)
    ->generate(Lab::OpenAI, 'text-embedding-3-small');
```

<a name="querying-embeddings"></a>
### Запросы по embeddings

Обычно embeddings сохраняются в `vector`-столбце базы данных для последующих запросов. Laravel поддерживает vector-столбцы PostgreSQL через расширение `pgvector`:

```php
Schema::ensureVectorExtensionExists();

Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->vector('embedding', dimensions: 1536);
    $table->timestamps();
});
```

Для ускорения similarity search добавьте vector index. При вызове `index` Laravel создаст HNSW-индекс с cosine distance:

```php
$table->vector('embedding', dimensions: 1536)->index();
```

В Eloquent-модели приведите vector-столбец к `array`:

```php
protected function casts(): array
{
    return [
        'embedding' => 'array',
    ];
}
```

Для поиска похожих записей используйте `whereVectorSimilarTo`. Метод фильтрует результаты по минимальному cosine similarity (от `0.0` до `1.0`) и сортирует по сходству:

```php
use App\Models\Document;

$documents = Document::query()
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```

`$queryEmbedding` может быть массивом floats или обычной строкой. Если передана строка, Laravel автоматически сгенерирует embeddings:

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', 'best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

Для низкоуровневого контроля используйте `whereVectorDistanceLessThan`, `selectVectorDistance` и `orderByVectorDistance`:

```php
$documents = Document::query()
    ->select('*')
    ->selectVectorDistance('embedding', $queryEmbedding, as: 'distance')
    ->whereVectorDistanceLessThan('embedding', $queryEmbedding, maxDistance: 0.3)
    ->orderByVectorDistance('embedding', $queryEmbedding)
    ->limit(10)
    ->get();
```

Если агенту нужен similarity search как tool, смотрите раздел [Similarity Search](#similarity-search).

> [!NOTE]
> Vector-запросы сейчас поддерживаются только на PostgreSQL-соединениях с расширением `pgvector`.

<a name="caching-embeddings"></a>
### Кеширование embeddings

Генерацию embeddings можно кешировать, чтобы избежать повторных API-вызовов для одинаковых входов. Для включения кеша установите `ai.caching.embeddings.cache` в `true`:

```php
'caching' => [
    'embeddings' => [
        'cache' => true,
        'store' => env('CACHE_STORE', 'database'),
        // ...
    ],
],
```

При включенном кеше embeddings хранятся 30 дней. Ключ кеша строится из провайдера, модели, размерности и входного содержимого, поэтому одинаковые запросы получают кешированные результаты, а разные конфигурации генерируют новые embeddings.

Кеш можно включить для конкретного запроса методом `cache`, даже если глобально он отключен:

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->cache()
    ->generate();
```

Можно указать duration в секундах:

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->cache(seconds: 3600)
    ->generate();
```

Метод `toEmbeddings` также принимает аргумент `cache`:

```php
$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings(cache: true);

$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings(cache: 3600);
```

<a name="reranking"></a>
## Реранжирование

Reranking позволяет переупорядочить список документов по релевантности заданному запросу. Это полезно для улучшения результатов поиска через семантическое понимание:

```php
use Laravel\Ai\Reranking;

$response = Reranking::of([
    'Django is a Python web framework.',
    'Laravel is a PHP web application framework.',
    'React is a JavaScript library for building user interfaces.',
])->rerank('PHP frameworks');

$response->first()->document; // "Laravel is a PHP web application framework."
$response->first()->score;    // 0.95
$response->first()->index;    // 1 (original position)
```

Метод `limit` ограничивает количество возвращаемых результатов:

```php
$response = Reranking::of($documents)
    ->limit(5)
    ->rerank('search query');
```

<a name="reranking-collections"></a>
### Реранжирование коллекций

Для удобства коллекции Laravel можно rerank через macro `rerank`. Первый аргумент задает поле или поля, второй - запрос:

```php
$posts = Post::all()
    ->rerank('body', 'Laravel tutorials');

$reranked = $posts->rerank(['title', 'body'], 'Laravel tutorials');

$reranked = $posts->rerank(
    fn ($post) => $post->title.': '.$post->body,
    'Laravel tutorials'
);
```

Также можно ограничить количество результатов и указать provider:

```php
$reranked = $posts->rerank(
    by: 'content',
    query: 'Laravel tutorials',
    limit: 10,
    provider: Lab::Cohere
);
```

<a name="files"></a>
## Файлы

Класс `Laravel\Ai\Files` или отдельные file-классы позволяют сохранять файлы у AI-провайдера для последующего использования в разговорах. Это полезно для больших документов или файлов, на которые нужно ссылаться многократно без повторной загрузки:

```php
use Laravel\Ai\Files\Document;
use Laravel\Ai\Files\Image;

$response = Document::fromPath('/home/laravel/document.pdf')->put();
$response = Image::fromPath('/home/laravel/photo.jpg')->put();

$response = Document::fromStorage('document.pdf', disk: 'local')->put();
$response = Image::fromStorage('photo.jpg', disk: 'local')->put();

$response = Document::fromUrl('https://example.com/document.pdf')->put();
$response = Image::fromUrl('https://example.com/photo.jpg')->put();

return $response->id;
```

Также можно сохранять raw content или uploaded files:

```php
use Laravel\Ai\Files;
use Laravel\Ai\Files\Document;

$stored = Document::fromString('Hello, World!', 'text/plain')->put();

$stored = Document::fromUpload($request->file('document'))->put();
```

После сохранения файла можно ссылаться на него при генерации текста через agents, не загружая файл повторно:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Analyze the attached sales transcript...'
    attachments: [
        Files\Document::fromId('file-id')
    ]
);
```

Чтобы получить ранее сохраненный файл, используйте метод `get` у экземпляра файла:

```php
use Laravel\Ai\Files\Document;

$file = Document::fromId('file-id')->get();

$file->id;
$file->mimeType();
```

Чтобы удалить файл у провайдера, используйте метод `delete`:

```php
Document::fromId('file-id')->delete();
```

По умолчанию `Files` использует default AI provider из `config/ai.php`. Для большинства операций можно указать другого provider:

```php
$response = Document::fromPath(
    '/home/laravel/document.pdf'
)->put(provider: Lab::Anthropic);
```

Вы можете передать параметры загрузки для конкретного провайдера с помощью метода `withProviderOptions`. Например, можно указать `purpose` файла OpenAI:

```php
use Laravel\Ai\Files\Document;

$response = Document::fromPath('/home/laravel/knowledge.txt')
    ->withProviderOptions(['purpose' => 'assistants'])
    ->put();
```

Чтобы задать параметры отдельно для каждого провайдера, передайте замыкание, которое получает текущего провайдера:

```php
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Files\Document;

$response = Document::fromPath('/home/laravel/training.jsonl')
    ->withProviderOptions(fn (Lab|string $provider) => match ($provider) {
        Lab::OpenAI => ['purpose' => 'fine-tune'],
        default => [],
    })
    ->put();
```

<a name="using-stored-files-in-conversations"></a>
### Использование сохраненных файлов в разговорах

После сохранения файла у провайдера можно ссылаться на него в agent conversations через `fromId` у `Document` или `Image`:

```php
use App\Ai\Agents\DocumentAnalyzer;
use Laravel\Ai\Files\Document;

$stored = Document::fromPath('/path/to/report.pdf')->put();

$response = (new DocumentAnalyzer)->prompt(
    'Summarize this document.',
    attachments: [
        Document::fromId($stored->id),
    ],
);
```

Сохраненные изображения подключаются аналогично:

```php
use Laravel\Ai\Files;
use Laravel\Ai\Files\Image;

$stored = Image::fromPath('/path/to/photo.jpg')->put();

$response = (new ImageAnalyzer)->prompt(
    'What is in this image?',
    attachments: [
        Image::fromId($stored->id),
    ],
);
```

<a name="vector-stores"></a>
## Vector stores

Vector stores позволяют создавать searchable collections файлов для retrieval-augmented generation (RAG). Класс `Laravel\Ai\Stores` предоставляет методы для создания, получения и удаления vector stores:

```php
use Laravel\Ai\Stores;

$store = Stores::create('Knowledge Base');

$store = Stores::create(
    name: 'Knowledge Base',
    description: 'Documentation and reference materials.',
    expiresWhenIdleFor: days(30),
);

return $store->id;
```

Получить существующее хранилище можно методом `get`:

```php
use Laravel\Ai\Stores;

$store = Stores::get('store_id');

$store->id;
$store->name;
$store->fileCounts;
$store->ready;
```

Удалить vector store можно через `Stores::delete` или экземпляр store:

```php
use Laravel\Ai\Stores;

Stores::delete('store_id');

$store = Stores::get('store_id');

$store->delete();
```

<a name="adding-files-to-stores"></a>
### Добавление файлов в хранилища

После создания vector store добавьте в него [файлы](#files) методом `add`. Файлы автоматически индексируются для семантического поиска через [file search provider tool](#file-search):

```php
use Laravel\Ai\Files\Document;
use Laravel\Ai\Stores;

$store = Stores::get('store_id');

$document = $store->add('file_id');
$document = $store->add(Document::fromId('file_id'));

$document = $store->add(Document::fromPath('/path/to/document.pdf'));
$document = $store->add(Document::fromStorage('manual.pdf'));
$document = $store->add($request->file('document'));

$document->id;
$document->fileId;
```

> **Note:** Обычно при добавлении ранее сохраненного файла в vector store возвращаемый document ID совпадает с ID файла, но некоторые провайдеры могут вернуть новый "document ID". Поэтому рекомендуется хранить оба ID в базе данных.

К файлам можно добавить metadata, чтобы затем фильтровать результаты при использовании [file search provider tool](#file-search):

```php
$store->add(Document::fromPath('/path/to/document.pdf'), metadata: [
    'author' => 'Taylor Otwell',
    'department' => 'Engineering',
    'year' => 2026,
]);
```

Удалить файл из store можно методом `remove`:

```php
$store->remove('file_id');
```

Удаление файла из vector store не удаляет его из [file storage](#files) провайдера. Чтобы удалить файл и из store, и из file storage, используйте `deleteFile`:

```php
$store->remove('file_abc123', deleteFile: true);
```

<a name="failover"></a>
## Failover

При prompt или генерации медиа можно передать массив providers / models, чтобы автоматически переключаться на резервного провайдера / модель при сбое сервиса или rate limit основного провайдера:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Image;

$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: [Lab::OpenAI, Lab::Anthropic],
);

$image = Image::of('A donut sitting on the kitchen counter')
    ->generate(provider: [Lab::Gemini, Lab::xAI]);
```

Failover происходит только когда выброшен `FailoverableException`, например rate limit (`RateLimitedException`), перегруженный или недоступный provider (`ProviderOverloadedException`) или недостаток credits (`InsufficientCreditsException`). Обычные ошибки, например validation или bad request error, не запускают failover.

Когда вы передаете простой список providers, например `[Lab::OpenAI, Lab::Anthropic]`, каждый provider использует свою default model. Чтобы указать конкретную model для каждого provider в failover chain, передайте associative array с ключами по provider, используя `value` enum `Lab` как ключ (enum cases нельзя напрямую использовать как PHP array keys):

```php
use Laravel\Ai\Enums\Lab;

$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: [
        Lab::Gemini->value => 'gemini-3-flash-preview',
        Lab::DeepSeek->value => 'deepseek-v4-pro',
    ],
);
```

<a name="testing"></a>
## Тестирование

<a name="testing-agents"></a>
### Агенты

Чтобы подменить ответы агента в тестах, вызовите `fake` на классе агента. Можно передать список ответов или замыкание:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Prompts\AgentPrompt;

SalesCoach::fake();

SalesCoach::fake([
    'First response',
    'Second response',
]);

SalesCoach::fake(function (AgentPrompt $prompt) {
    return 'Response for: '.$prompt->prompt;
});
```

При подмене агента, который возвращает structured output, можно передавать массивы в качестве ответов. Агент вернет structured response с переданными данными:

```php
SalesCoach::fake([
    ['score' => 87],
]);
```

> **Note:** Если `Agent::fake()` вызван для агента со structured output и fake output не был передан явно, Laravel автоматически сгенерирует fake data, соответствующие схеме вывода.

После prompt можно утверждать, какие prompts были получены:

```php
use Laravel\Ai\Prompts\AgentPrompt;

SalesCoach::assertPrompted('Analyze this...');

SalesCoach::assertPrompted(function (AgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotPrompted('Missing prompt');

SalesCoach::assertNeverPrompted();
```

Для queued invocations используйте queued assertions:

```php
use Laravel\Ai\QueuedAgentPrompt;

SalesCoach::assertQueued('Analyze this...');

SalesCoach::assertQueued(function (QueuedAgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotQueued('Missing prompt');

SalesCoach::assertNeverQueued();
```

Метод `preventStrayPrompts` гарантирует, что для каждого обращения к агенту есть fake response:

```php
SalesCoach::fake()->preventStrayPrompts();
```

<a name="testing-images"></a>
### Изображения

Генерацию изображений можно подменить методом `fake` класса `Image`, после чего доступны assertions по записанным prompts:

```php
use Laravel\Ai\Image;
use Laravel\Ai\Prompts\ImagePrompt;
use Laravel\Ai\Prompts\QueuedImagePrompt;

Image::fake();

Image::fake([
    base64_encode($firstImage),
    base64_encode($secondImage),
]);

Image::fake(function (ImagePrompt $prompt) {
    return base64_encode('...');
});
```

```php
Image::assertGenerated(function (ImagePrompt $prompt) {
    return $prompt->contains('sunset') && $prompt->isLandscape();
});

Image::assertNotGenerated('Missing prompt');

Image::assertNothingGenerated();
```

Для queued image generations используйте queued assertion methods:

```php
Image::assertQueued(
    fn (QueuedImagePrompt $prompt) => $prompt->contains('sunset')
);

Image::assertNotQueued('Missing prompt');

Image::assertNothingQueued();
```

Чтобы убедиться, что для каждой генерации изображения есть соответствующий fake response, используйте `preventStrayImages`. Если изображение будет сгенерировано без заданного fake response, будет выброшено исключение:

```php
Image::fake()->preventStrayImages();
```

<a name="testing-audio"></a>
### Аудио

Аудиогенерацию можно подменить методом `fake` класса `Audio`:

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Prompts\AudioPrompt;
use Laravel\Ai\Prompts\QueuedAudioPrompt;

Audio::fake();

Audio::fake([
    base64_encode($firstAudio),
    base64_encode($secondAudio),
]);

Audio::fake(function (AudioPrompt $prompt) {
    return base64_encode('...');
});
```

```php
Audio::assertGenerated(function (AudioPrompt $prompt) {
    return $prompt->contains('Hello') && $prompt->isFemale();
});

Audio::assertNotGenerated('Missing prompt');
Audio::assertNothingGenerated();
```

Для queued audio generations используйте queued assertion methods:

```php
Audio::assertQueued(
    fn (QueuedAudioPrompt $prompt) => $prompt->contains('Hello')
);

Audio::assertNotQueued('Missing prompt');
Audio::assertNothingQueued();
```

Чтобы убедиться, что для каждой генерации аудио есть соответствующий fake response, используйте `preventStrayAudio`. Если аудио будет сгенерировано без заданного fake response, будет выброшено исключение:

```php
Audio::fake()->preventStrayAudio();
```

<a name="testing-transcriptions"></a>
### Транскрипции

Транскрипции можно подменить методом `fake` класса `Transcription`:

```php
use Laravel\Ai\Transcription;
use Laravel\Ai\Prompts\TranscriptionPrompt;
use Laravel\Ai\Prompts\QueuedTranscriptionPrompt;

Transcription::fake();

Transcription::fake([
    'First transcription text.',
    'Second transcription text.',
]);

Transcription::fake(function (TranscriptionPrompt $prompt) {
    return 'Transcribed text...';
});
```

```php
Transcription::assertGenerated(function (TranscriptionPrompt $prompt) {
    return $prompt->language === 'en' && $prompt->isDiarized();
});

Transcription::assertNotGenerated(
    fn (TranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingGenerated();
```

Для queued transcriptions используйте queued assertion methods:

```php
Transcription::assertQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->isDiarized()
);

Transcription::assertNotQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingQueued();
```

Чтобы убедиться, что для каждой транскрипции есть соответствующий fake response, используйте `preventStrayTranscriptions`. Если транскрипция будет создана без заданного fake response, будет выброшено исключение:

```php
Transcription::fake()->preventStrayTranscriptions();
```

<a name="testing-embeddings"></a>
### Embeddings

Генерацию embeddings можно подменить методом `fake` класса `Embeddings`:

```php
use Laravel\Ai\Embeddings;
use Laravel\Ai\Prompts\EmbeddingsPrompt;
use Laravel\Ai\Prompts\QueuedEmbeddingsPrompt;

Embeddings::fake();

Embeddings::fake([
    [$firstEmbeddingVector],
    [$secondEmbeddingVector],
]);

Embeddings::fake(function (EmbeddingsPrompt $prompt) {
    return array_map(
        fn () => Embeddings::fakeEmbedding($prompt->dimensions),
        $prompt->inputs
    );
});
```

```php
Embeddings::assertGenerated(function (EmbeddingsPrompt $prompt) {
    return $prompt->contains('Laravel') && $prompt->dimensions === 1536;
});

Embeddings::assertNotGenerated(
    fn (EmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingGenerated();
```

Для queued embeddings используйте queued assertion methods:

```php
Embeddings::assertQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Laravel')
);

Embeddings::assertNotQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingQueued();
```

Чтобы убедиться, что для каждой генерации embeddings есть соответствующий fake response, используйте `preventStrayEmbeddings`. Если embeddings будут сгенерированы без заданного fake response, будет выброшено исключение:

```php
Embeddings::fake()->preventStrayEmbeddings();
```

<a name="testing-reranking"></a>
### Реранжирование

Операции reranking можно подменить методом `fake` класса `Reranking`:

```php
use Laravel\Ai\Reranking;
use Laravel\Ai\Prompts\RerankingPrompt;
use Laravel\Ai\Responses\Data\RankedDocument;

Reranking::fake();

Reranking::fake([
    [
        new RankedDocument(index: 0, document: 'First', score: 0.95),
        new RankedDocument(index: 1, document: 'Second', score: 0.80),
    ],
]);
```

```php
Reranking::assertReranked(function (RerankingPrompt $prompt) {
    return $prompt->contains('Laravel') && $prompt->limit === 5;
});

Reranking::assertNotReranked(
    fn (RerankingPrompt $prompt) => $prompt->contains('Django')
);

Reranking::assertNothingReranked();
```

<a name="testing-files"></a>
### Файлы

File operations можно подменить методом `fake` класса `Files`:

```php
use Laravel\Ai\Files;

Files::fake();
```

После этого можно проверять uploads и deletions:

```php
use Laravel\Ai\Contracts\Files\StorableFile;
use Laravel\Ai\Files\Document;

Document::fromString('Hello, Laravel!', mimeType: 'text/plain')
    ->as('hello.txt')
    ->put();

Files::assertStored(fn (StorableFile $file) =>
    (string) $file === 'Hello, Laravel!' &&
        $file->mimeType() === 'text/plain';
);

Files::assertNotStored(fn (StorableFile $file) =>
    (string) $file === 'Hello, World!'
);

Files::assertNothingStored();
```

Для deletions можно передать file ID:

```php
Files::assertDeleted('file-id');
Files::assertNotDeleted('file-id');
Files::assertNothingDeleted();
```

<a name="testing-vector-stores"></a>
### Vector stores

Операции vector store можно подменить методом `fake` класса `Stores`. Это автоматически подменит и [file operations](#files):

```php
use Laravel\Ai\Stores;

Stores::fake();
```

Далее можно проверять созданные или удаленные stores:

```php
use Laravel\Ai\Stores;

$store = Stores::create('Knowledge Base');

Stores::assertCreated('Knowledge Base');

Stores::assertCreated(fn (string $name, ?string $description) =>
    $name === 'Knowledge Base'
);

Stores::assertNotCreated('Other Store');

Stores::assertNothingCreated();
```

Для удаления:

```php
Stores::assertDeleted('store_id');
Stores::assertNotDeleted('other_store_id');
Stores::assertNothingDeleted();
```

Чтобы проверить добавление или удаление файлов из store, используйте assertion methods на экземпляре `Store`:

```php
Stores::fake();

$store = Stores::get('store_id');

$store->add('added_id');
$store->remove('removed_id');

$store->assertAdded('added_id');
$store->assertRemoved('removed_id');

$store->assertNotAdded('other_file_id');
$store->assertNotRemoved('other_file_id');
```

Если файл одновременно сохраняется у провайдера и добавляется в vector store, provider ID может быть неизвестен. В этом случае передайте замыкание в `assertAdded`:

```php
use Laravel\Ai\Contracts\Files\StorableFile;
use Laravel\Ai\Files\Document;

$store->add(Document::fromString('Hello, World!', 'text/plain')->as('hello.txt'));

$store->assertAdded(fn (StorableFile $file) => $file->name() === 'hello.txt');
$store->assertAdded(fn (StorableFile $file) => $file->content() === 'Hello, World!');
```

<a name="events"></a>
## События

Laravel AI SDK отправляет разные [события](/docs/{{version}}/events), включая:

- `AddingFileToStore`
- `AgentPrompted`
- `AgentStreamed`
- `AudioGenerated`
- `CreatingStore`
- `EmbeddingsGenerated`
- `FileAddedToStore`
- `FileDeleted`
- `FileRemovedFromStore`
- `FileStored`
- `GeneratingAudio`
- `GeneratingEmbeddings`
- `GeneratingImage`
- `GeneratingTranscription`
- `ImageGenerated`
- `InvokingTool`
- `PromptingAgent`
- `RemovingFileFromStore`
- `Reranked`
- `Reranking`
- `StoreCreated`
- `StoringFile`
- `StreamingAgent`
- `ToolInvoked`
- `TranscriptionGenerated`

Вы можете слушать любые из этих событий, чтобы логировать или сохранять информацию об использовании AI SDK.
