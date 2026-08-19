---
git: 57ae1e7dbd4bda3bae24ce93e527f1807ae49a43
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
    - [Трансляция](#broadcasting)
    - [Очереди](#queueing)
    - [Инструменты](#tools)
    - [Инструменты файлового хранилища](#file-storage-tools)
    - [Инструменты MCP](#mcp-tools)
    - [Инструменты провайдеров](#provider-tools)
    - [Субагенты](#sub-agents)
    - [Посредники](#middleware)
    - [Анонимные агенты](#anonymous-agents)
    - [Конфигурация агента](#agent-configuration)
    - [Опции провайдера](#provider-options)
- [Подтверждение инструментов человеком](#human-tool-approval)
    - [Полный процесс подтверждения](#complete-approval-flow)
- [Изображения](#images)
- [Аудио (TTS)](#audio)
- [Транскрипции (STT)](#transcription)
- [Суммаризация текста](#text-summarization)
- [Embeddings](#embeddings)
    - [Мультимодальные embeddings](#multimodal-embeddings)
    - [Запросы по embeddings](#querying-embeddings)
    - [Кеширование embeddings](#caching-embeddings)
- [Реранжирование](#reranking)
- [Файлы](#files)
- [Векторные хранилища](#vector-stores)
    - [Добавление файлов в хранилища](#adding-files-to-stores)
- [Переключение при сбое](#failover)
- [Тестирование](#testing)
    - [Агенты](#testing-agents)
    - [Изображения](#testing-images)
    - [Аудио](#testing-audio)
    - [Транскрипции](#testing-transcriptions)
    - [Embeddings](#testing-embeddings)
    - [Реранжирование](#testing-reranking)
    - [Файлы](#testing-files)
    - [Векторные хранилища](#testing-vector-stores)
- [События](#events)

<a name="introduction"></a>
## Введение

[Laravel AI SDK](https://github.com/laravel/ai) предоставляет единый выразительный API для работы с AI-провайдерами, такими как OpenAI, Anthropic, Gemini и другими. С помощью AI SDK можно создавать интеллектуальных агентов с инструментами и структурированным выводом, генерировать изображения, синтезировать и транскрибировать аудио, создавать векторные представления и многое другое через единообразный интерфейс в стиле Laravel.

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

Это полезно при маршрутизации запросов через прокси-сервисы вроде LiteLLM или Azure OpenAI Gateway, а также при использовании альтернативных эндпоинтов.

Пользовательские базовые URL поддерживаются для OpenAI, Anthropic, Gemini, Groq, Cohere, DeepSeek, xAI и OpenRouter.

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

К каждому исходящему запросу провайдера можно добавить пользовательские HTTP-заголовки, указав массив `headers` в его конфигурации. Это удобно, когда эндпоинт помимо bearer-токена требует дополнительный заголовок для идентификации или аутентификации:

```php
'local' => [
    'driver' => 'openai-compatible',
    'url' => env('LOCAL_AI_URL'),
    'key' => env('LOCAL_AI_API_KEY'),
    'headers' => [
        'X-Tenant-Id' => env('LOCAL_AI_TENANT_ID'),
    ],
],
```

OpenAI-совместимые провайдеры поддерживают генерацию текста, потоковую передачу, инструменты, структурированный вывод, вложения изображений и векторные представления. Если вашему эндпоинту требуются дополнительные поля тела запроса, передайте их с помощью [опций провайдера](#provider-options).

<a name="openai-compatible-embeddings"></a>
#### Векторные представления в OpenAI-совместимых провайдерах

Поскольку модели произвольного эндпоинта заранее неизвестны, для использования `embeddings()` с OpenAI-совместимым провайдером необходимо настроить модель векторных представлений по умолчанию. Также можно задать фиксированную размерность. Если она не указана, запрос отправляется без параметра `dimensions` и используется собственная размерность модели.

```php
'local' => [
    'driver' => 'openai-compatible',
    'url' => env('LOCAL_AI_URL'),
    'key' => env('LOCAL_AI_API_KEY'),
    'models' => [
        'embeddings' => [
            'default' => 'text-embedding-qwen3-embedding-0.6b',
            'dimensions' => 1024, // необязательно
        ],
    ],
],
```

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
| Embeddings | OpenAI, OpenAI-Compatible, Gemini, Azure, Bedrock, Cohere, Mistral, Jina, VoyageAI, Ollama, OpenRouter |
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

Агенты - основной строительный блок для взаимодействия с AI-провайдерами в Laravel AI SDK. Каждый агент является отдельным PHP-классом, инкапсулирующим инструкции, контекст разговора, инструменты и схему вывода, необходимые для работы с большой языковой моделью. Представьте агента как специализированного помощника: наставника по продажам, анализатора документов или бота поддержки, которого вы настраиваете один раз и затем вызываете по мере необходимости.

Агента можно создать Artisan-командой `make:agent`:

```shell
php artisan make:agent SalesCoach

php artisan make:agent SalesCoach --structured
```

В сгенерированном классе можно определить системную инструкцию, контекст сообщений, доступные инструменты и схему вывода:

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
    model: 'claude-sonnet-5',
    timeout: 120,
);
```

<a name="raw-http-responses"></a>
#### Необработанные HTTP-ответы

Каждый ответ агента, генерирующего текст, предоставляет через свойство `raw` исходный HTTP-ответ от API базового провайдера. Это дает доступ к специфичным для провайдера сведениям, не входящим в универсальный ответ AI SDK: заголовкам ограничения частоты, идентификаторам запросов и другим полям исходной полезной нагрузки:

```php
$response = (new SalesCoach)->prompt('Analyze this sales transcript...');

$response->raw; // Illuminate\Http\Client\Response|null

$response->raw->header('X-RateLimit-Remaining-Requests');
$response->raw->json('id');
```

В цикле вызовов инструментов каждый шаг сохраняет исходный ответ собственного запроса:

```php
foreach ($response->steps as $step) {
    $step->raw?->header('X-RateLimit-Remaining-Requests');
}
```

> **Примечание:** Свойство `raw` равно `null` при потоковой передаче ответа, при использовании провайдера Bedrock, выполняющего API-вызовы через AWS SDK вместо HTTP-клиента, а также в поддельных ответах, если ответ не был явно передан через `withRawResponse`.

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

> **Предупреждение:** Перед использованием трейта `RemembersConversations` опубликуйте и выполните миграции AI SDK через `vendor:publish`, чтобы создать таблицы для хранения разговоров.

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

Идентификатор разговора возвращается в ответе и может быть сохранён для дальнейшего использования. Чтобы получать все разговоры пользователя через Eloquent, добавьте трейт `HasConversations` к модели пользователя:

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

При использовании `RemembersConversations` предыдущие сообщения автоматически загружаются и включаются в контекст каждого запроса. Новые сообщения пользователя и ассистента сохраняются после каждого взаимодействия.

<a name="conversation-participants"></a>
#### Участники разговоров

Хотя пользователи являются наиболее распространенными участниками разговоров, разговоры могут принадлежать любой модели Eloquent. Используйте метод `forParticipant`, чтобы начать разговор для модели другого типа:

```php
$response = (new SalesCoach)
    ->forParticipant($team)
    ->prompt('Review our latest sales results.');
```

Morph-класс и первичный ключ участника сохраняются вместе с разговором. Поэтому модели разных типов с одинаковым первичным ключом, например `User` с ID `1` и `Team` с ID `1`, имеют отдельные истории разговоров. Метод `forUser` является псевдонимом `forParticipant`.

Вы можете продолжить самый последний разговор участника с помощью метода `continueLastConversation`:

```php
$response = (new SalesCoach)
    ->continueLastConversation($team)
    ->prompt('Tell me more about that.');
```

При продолжении конкретного разговора передайте участника в метод `continue`:

```php
$response = (new SalesCoach)
    ->continue($conversationId, as: $team)
    ->prompt('Tell me more about that.');
```

Трейт `HasConversations` можно добавить к любой модели Eloquent, которая участвует в разговорах. Полученное отношение `conversations` является полиморфным отношением, ограниченным типом и первичным ключом этой модели. Вы также можете получить участника, которому принадлежит разговор, через обратное отношение:

```php
$conversations = $team->conversations;

$participant = $conversation->participant;
```

Если ваше приложение использует несколько типов моделей-участников, стоит определить [Eloquent morph map](/docs/{{version}}/eloquent-relationships#custom-polymorphic-types), чтобы сохраненные типы участников не были привязаны к именам классов моделей.

> [!WARNING]
> Метод `continue` не проверяет, что переданный участник владеет разговором. Ваше приложение должно авторизовать доступ к разговору перед его продолжением.

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

Чтобы определить вложенный структурированный вывод, используйте метод `object` с замыканием:

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

Если агент должен вернуть список структурированных элементов, объедините методы `array` и `object`:

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

При обращении к агенту можно передать вложения, чтобы модель могла анализировать изображения и документы:

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

Ответ агента можно передавать потоком через метод `stream`. Возвращаемый экземпляр `StreamableAgentResponse` можно вернуть из маршрута, чтобы автоматически отправить клиенту потоковый ответ (SSE):

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

Чтобы передавать события через [потоковый протокол Vercel AI SDK](https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol), вызовите `usingVercelDataProtocol`:

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

Некоторые платформы трансляции ограничивают размер WebSocket-сообщений примерно 10 КБ. Потоковые события с большим объёмом данных, например крупные результаты инструментов, могут превысить этот лимит и привести к ошибке трансляции. Определённые типы событий можно исключить из трансляции с помощью атрибута `WithoutBroadcasting`:

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

Исключённые события никогда не транслируются, но по-прежнему записываются в таблицу `agent_conversation_messages`, поэтому фронтенд сможет загрузить полные данные инструмента после завершения потока. Это работает как для трансляции через очередь (`broadcastOnQueue`), так и для синхронной трансляции (`broadcast` / `broadcastNow`).

<a name="queueing"></a>
### Очереди

Метод `queue` позволяет отправить запрос агенту и обработать ответ в фоновом режиме, сохраняя отзывчивость приложения. Методы `then` и `catch` регистрируют обратные вызовы, которые выполняются после получения ответа или возникновения исключения:

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

Инструменты предоставляют агентам дополнительные возможности, которыми они могут пользоваться при формировании ответа. Создать инструмент можно Artisan-командой `make:tool`:

```shell
php artisan make:tool RandomNumberGenerator
```

Сгенерированный инструмент будет размещён в каталоге `app/Ai/Tools`. Каждый инструмент содержит метод `handle`, который агент вызывает при необходимости:

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

Инструмент возвращается из метода `tools` любого агента:

```php
use App\Ai\Tools\RandomNumberGenerator;

public function tools(): iterable
{
    return [
        new RandomNumberGenerator,
    ];
}
```

<a name="repairing-tool-calls"></a>
#### Исправление вызовов инструментов

Атрибут `RepairToolCalls` позволяет агенту восстановиться, если модель вызывает неизвестный локальный инструмент. Laravel возвращает модели сведения о неудачном вызове вместе с именами доступных локальных инструментов, позволяя исправить вызов:

```php
use Laravel\Ai\Attributes\RepairToolCalls;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Promptable;

#[RepairToolCalls]
class SupportAgent implements Agent, HasTools
{
    use Promptable;

    // ...
}
```

Когда Laravel автоматически определяет максимальное количество шагов, этот атрибут добавляет один шаг для исправленного вызова. Явно заданные ограничения `MaxSteps` не изменяются.

<a name="similarity-search"></a>
#### Similarity Search

Инструмент `SimilaritySearch` позволяет агентам искать документы, похожие на заданный запрос, используя векторные представления, хранящиеся в базе данных. Это удобно для генерации с дополненной выборкой (retrieval-augmented generation, RAG), когда агенту требуется доступ к поиску по данным приложения.

Самый простой способ создать инструмент поиска по сходству - использовать метод `usingModel` с Eloquent-моделью, содержащей векторные представления:

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

Первый аргумент - класс Eloquent-модели, а второй - столбец, содержащий векторные представления.

Также можно указать минимальный порог сходства от `0.0` до `1.0`, лимит и замыкание для настройки запроса:

```php
SimilaritySearch::usingModel(
    model: Document::class,
    column: 'embedding',
    minSimilarity: 0.7,
    limit: 10,
    query: fn ($query) => $query->where('published', true),
),
```

Для более гибкой настройки можно создать инструмент поиска по сходству с пользовательским замыканием, возвращающим результаты поиска:

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

Описание инструмента можно настроить методом `withDescription`:

```php
SimilaritySearch::usingModel(Document::class, 'embedding')
    ->withDescription('Search the knowledge base for relevant articles.'),
```

<a name="file-storage-tools"></a>
### Инструменты файлового хранилища

Фабрика инструментов `FileStorage` позволяет предоставить агентам доступ к [диску файловой системы](/docs/{{version}}/filesystem) Laravel. Метод `all` возвращает инструменты, с помощью которых агент может просматривать список файлов, читать и проверять файлы, генерировать URL, а также записывать, удалять и копировать файлы на указанном диске:

```php
use Laravel\Ai\Tools\FileStorage;

public function tools(): iterable
{
    return FileStorage::all('local');
}
```

Если агент должен иметь возможность только просматривать файлы, используйте метод `readOnly`:

```php
return FileStorage::readOnly('local');
```

Эти методы возвращают коллекцию `Illuminate\Support\Collection`, что позволяет дополнительно фильтровать инструменты, предоставляемые агенту:

```php
use Laravel\Ai\Tools\Filesystem\DeleteFile;

return FileStorage::all('s3')
    ->reject(fn ($tool) => $tool instanceof DeleteFile);
```

<a name="mcp-tools"></a>
### MCP Tools

Если приложение использует [Laravel MCP](/docs/{{version}}/mcp), агентам можно предоставить инструменты, опубликованные серверами [Model Context Protocol](https://modelcontextprotocol.io). С помощью [клиента Laravel MCP](/docs/{{version}}/mcp#client) можно подключиться к удалённому или локальному серверу MCP и передать его инструменты непосредственно агенту.

> [!NOTE]
> Для инструментов MCP в приложении должен быть установлен пакет [Laravel MCP](/docs/{{version}}/mcp).

Поскольку метод `tools` клиента MCP возвращает коллекцию, разверните её в массив `tools` агента с помощью оператора `...`:

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

AI SDK автоматически оборачивает каждый инструмент MCP, чтобы агент мог вызывать его как любой другой инструмент. Также можно использовать [именованный клиент MCP](/docs/{{version}}/mcp#named-clients):

```php
use Laravel\Mcp\Facades\Mcp;

public function tools(): iterable
{
    return [
        ...Mcp::client('github')->tools(),
    ];
}
```

Или подключиться к [локальному серверу MCP](/docs/{{version}}/mcp#client-connecting):

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

Инструменты провайдеров - специальные инструменты, нативно реализованные AI-провайдерами. Они предоставляют такие возможности, как веб-поиск, получение содержимого URL и поиск по файлам. В отличие от обычных инструментов, инструменты провайдеров выполняются самим провайдером, а не вашим приложением.

Инструменты провайдеров можно возвращать из метода `tools` агента.

<a name="web-search"></a>
#### Web Search

Инструмент провайдера `WebSearch` позволяет агентам искать в интернете актуальную информацию. Это полезно для вопросов о текущих событиях, свежих данных или темах, которые могли измениться после даты обучения модели.

**Поддерживаемые провайдеры:** Anthropic, OpenAI, Azure, Gemini, OpenRouter

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

Инструмент провайдера `WebFetch` позволяет агентам получать и читать содержимое веб-страниц. Это полезно, когда агент должен анализировать конкретные URL или получить подробную информацию с известных страниц.

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

Количество запросов на получение страниц и перечень доменов можно ограничить:

```php
(new WebFetch)->max(3)->allow(['docs.laravel.com']),
```

<a name="file-search"></a>
#### File Search

Инструмент провайдера `FileSearch` позволяет агентам искать по [файлам](#files), сохранённым в [векторных хранилищах](#vector-stores). Это включает сценарии RAG, в которых агент ищет релевантную информацию в загруженных документах.

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

Можно указать идентификаторы нескольких векторных хранилищ:

```php
new FileSearch(stores: ['store_1', 'store_2']);
```

Если у файлов есть [метаданные](#adding-files-to-stores), результаты можно фильтровать через аргумент `where`:

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
### Субагенты

Агентов также можно возвращать из метода `tools` другого агента. Когда агент возвращается как инструмент, родительский агент может поручить субагенту конкретную задачу и использовать его ответ при формировании ответа на исходный запрос. Это удобно, когда универсальному агенту требуется доступ к специализированным агентам со своими инструкциями, инструментами, настройками модели или предпочтениями провайдера.

Например, агент поддержки клиентов может поручать вопросы о праве на возврат средств отдельному агенту по возвратам:

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

Чтобы настроить представление субагента для родительского агента, реализуйте интерфейс `CanActAsTool` в субагенте и определите имя и описание, с которыми он будет доступен как инструмент:

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

Если субагент не реализует `CanActAsTool`, Laravel использует базовое имя его класса в качестве имени инструмента и общее описание, предлагающее родительскому агенту передать ясное и самодостаточное описание задачи. Каждый вызов субагента выполняется изолированно и не получает историю разговора родительского агента.

<a name="middleware"></a>
### Middleware

Агенты поддерживают посредников, позволяющих перехватывать и изменять запросы до их отправки провайдеру. Посредник создаётся командой `make:agent-middleware`:

```shell
php artisan make:agent-middleware LogPrompts
```

Чтобы добавить посредников к агенту, реализуйте `HasMiddleware` и верните их список:

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

Метод `then` объекта ответа позволяет выполнить код после завершения обработки как для синхронных, так и для потоковых ответов:

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

- `MaxSteps`: максимальное количество шагов агента при использовании инструментов.
- `MaxTokens`: максимальное количество tokens, которое может сгенерировать модель.
- `Model`: модель агента.
- `Provider`: AI-провайдер или список провайдеров для переключения при сбое.
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
#[Model('claude-sonnet-5')]
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
> Базовая модель, выбираемая атрибутами `UseCheapestModel` и `UseSmartestModel`, может меняться между выпусками Laravel AI SDK по мере появления новых моделей у провайдеров. Смена модели способна повлиять на поведение, привести к использованию устаревших параметров и существенно изменить стоимость. Если вам нужна стабильная и предсказуемая модель с известной ценой, явно укажите ее с помощью атрибута `Model`.

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

Метод получает текущего провайдера (вариант перечисления `Lab` или строку), поэтому для каждого провайдера можно возвращать разные опции. Это особенно полезно при [переключении после сбоя](#failover), когда каждый резервный провайдер может иметь собственную конфигурацию.

Пример Anthropic выше также включает [кеширование запросов](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) с помощью `cache_control`.

<a name="human-tool-approval"></a>
## Подтверждение инструментов человеком

> [!WARNING]
> Подтверждение инструментов требует агента `Conversational`, история разговора которого сохраняется, чтобы приостановленный вызов можно было возобновить. Трейт `RemembersConversations` предоставляет необходимое сохранение.

Инструменты, выполняющие чувствительные или необратимые действия, могут требовать подтверждения человеком перед выполнением. Чтобы инструмент можно было подтверждать, реализуйте контракт `Approvable` и используйте трейт `InteractsWithApprovals`. Подтверждаемые инструменты по умолчанию требуют подтверждения:

```php
<?php

namespace App\Ai\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Illuminate\Support\Facades\Storage;
use Laravel\Ai\Concerns\InteractsWithApprovals;
use Laravel\Ai\Contracts\Approvable;
use Laravel\Ai\Contracts\Tool;
use Laravel\Ai\Tools\Request;
use Stringable;

class DeleteFile implements Approvable, Tool
{
    use InteractsWithApprovals;

    /**
     * Get the description of the tool's purpose.
     */
    public function description(): Stringable|string
    {
        return 'Delete a file from storage.';
    }

    /**
     * Execute the tool.
     */
    public function handle(Request $request): Stringable|string
    {
        Storage::delete($request['path']);

        return "Deleted [{$request['path']}].";
    }

    /**
     * Get the tool's schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'path' => $schema->string()->required(),
        ];
    }
}
```

Чтобы определить, нужно ли подтверждение на основе аргументов вызова инструмента, определите в инструменте метод `needsApproval`. Этот метод может вернуть boolean или экземпляр `Approval`, включающий причину запроса подтверждения:

```php
use Laravel\Ai\Approvals\Approval;

/**
 * Determine whether the tool needs approval for the given request.
 */
protected function needsApproval(Request $request): Approval|bool
{
    return str_starts_with($request['path'], 'temporary/')
        ? false
        : Approval::required('This will permanently delete a file.');
}
```

Вы можете переопределить требование подтверждения инструмента при возврате его из метода `tools` агента:

```php
public function tools(): iterable
{
    return [
        (new SendNotification)->withoutApproval(),
        (new DeleteFile)->requireApproval('Deletion review required.'),
    ];
}
```

Когда вызывается требующий подтверждения инструмент, агент приостанавливается перед его выполнением. Ожидающие подтверждения можно изучить в ответе: они содержат идентификатор каждого вызова инструмента, имя инструмента, аргументы и причину подтверждения:

```php
$response = (new FileAssistant)
    ->forUser($user)
    ->prompt('Delete the old invoice.');

if ($response->hasPendingApprovals()) {
    foreach ($response->pendingApprovals as $approval) {
        // $approval->id
        // $approval->tool
        // $approval->arguments
        // $approval->reason
    }
}
```

Чтобы возобновить работу агента, продолжите разговор и передайте экземпляр `Decisions`, содержащий решение для каждого ожидающего подтверждения вызова инструмента. Решения позволяют подтвердить или отклонить вызов либо изменить его аргументы перед выполнением:

```php
use Laravel\Ai\Approvals\Decision;
use Laravel\Ai\Approvals\Decisions;

$response = (new FileAssistant)
    ->continue($conversationId, as: $user)
    ->prompt(Decisions::from([
        'call_abc' => Decision::approve(),
        'call_ghi' => Decision::reject('The invoice must be retained.'),
    ]));
```

Логические значения `true` и `false` можно использовать как сокращения для подтверждения и отклонения. Для каждого ожидающего подтверждения вызова инструмента должно быть принято решение. Неизвестные, отсутствующие или уже обработанные идентификаторы вызовов приведут к исключению `ApprovalMismatchException`. Для вызовов без явного решения можно задать действие по умолчанию с помощью методов `approveRemaining` или `rejectRemaining`:

```php
$decisions = Decisions::from([
    'call_abc' => true,
])->rejectRemaining('Not approved.');

$response = (new FileAssistant)
    ->continue($conversationId, as: $user)
    ->prompt($decisions);
```

Отклонение с результатом, например `Decision::reject('Not approved.')`, возвращается модели, чтобы она могла продолжить ответ. Отклонение без результата останавливает цикл генерации после записи отклонения.

Подтверждение инструментов поддерживается методами `prompt`, `stream`, `queue`, `broadcast`, `broadcastNow` и `broadcastOnQueue`.

Во время потоковой передачи и трансляции пауза представляется событием `tool_approval_request`. При использовании [потокового протокола Vercel AI SDK](#streaming-using-the-vercel-ai-sdk-protocol) запросы подтверждения и результаты отправляются с помощью встроенных частей протокола для подтверждения инструментов.

Для агентов, поставленных в очередь, итоговый ответ передаётся в обратный вызов `then`, а Laravel также отправляет событие `ToolApprovalRequested`.

Laravel сохраняет результат подтверждённого инструмента перед тем, как попросить модель продолжить. Если после этого генерация завершится ошибкой, подтверждение уже будет обработано. Продолжите разговор обычным текстовым запросом вместо повторной отправки тех же решений.

<a name="complete-approval-flow"></a>
### Полный процесс подтверждения

Следующие маршруты демонстрируют полный процесс подтверждения. Маршрут `GET` возвращает экран чата, а маршрут `POST` принимает либо новый текстовый запрос, либо решения о подтверждении с экрана чата. В этом примере предполагается, что модель `User` приложения использует трейт `HasConversations`:

```php
use App\Ai\Agents\FileAssistant;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;
use Illuminate\Support\Facades\Route;
use Illuminate\Validation\Rule;
use Laravel\Ai\Approvals\Decision;
use Laravel\Ai\Approvals\Decisions;
use Laravel\Ai\Models\Conversation;

Route::get('/chat/{conversation}', function (Request $request, Conversation $conversation) {
    Gate::authorize('view', $conversation);

    return view('chat', [
        'conversation' => $conversation,
    ]);
})->middleware('auth');

Route::post('/chat/{conversation}', function (Request $request, Conversation $conversation) {
    Gate::authorize('view', $conversation);

    $validated = $request->validate([
        'message' => ['nullable', 'string', 'required_without:decisions', 'prohibited_with:decisions'],
        'decisions' => ['nullable', 'array', 'required_without:message', 'prohibited_with:message'],
        'decisions.*.action' => ['required_with:decisions', Rule::in(['approve', 'reject'])],
        'decisions.*.result' => ['nullable', 'string'],
    ]);

    $prompt = isset($validated['decisions'])
        ? Decisions::from($validated->collect('decisions')->map(
            fn (array $decision) => match ($decision['action']) {
                'approve' => Decision::approve(),
                'reject' => Decision::reject($decision['result'] ?? null),
            }
        )->all())
        : $validated['message'];

    $response = (new FileAssistant)
        ->continue($conversation->id, as: $request->user())
        ->prompt($prompt);

    return [
        'conversation_id' => $response->conversationId,
        'status' => $response->hasPendingApprovals() ? 'awaiting_approval' : 'complete',
        'message' => $response->text,
        'approvals' => $response->pendingApprovals,
    ];
})->middleware('auth');
```

Когда статус ответа равен `awaiting_approval`, экран чата должен отобразить ожидающие подтверждения и отправить выбор пользователя в тот же эндпоинт, используя идентификатор вызова инструмента как ключ каждого решения:

```json
{
    "decisions": {
        "call_abc": {
            "action": "approve"
        },
        "call_def": {
            "action": "reject",
            "result": "The invoice must be retained."
        }
    }
}
```

Для обычного сообщения чата экран вместо этого может отправить значение `message`:

```json
{
    "message": "Delete the old invoice."
}
```

<a name="images"></a>
## Изображения

Класс `Laravel\Ai\Image` используется для генерации изображений через провайдеры `openai`, `gemini` или `xai`:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

Методы `square`, `portrait` и `landscape` управляют соотношением сторон, `quality` задаёт желаемое качество (`high`, `medium`, `low`), а `timeout` - тайм-аут HTTP-запроса:

```php
$image = Image::of('A donut sitting on the kitchen counter')
    ->quality('high')
    ->landscape()
    ->timeout(120)
    ->generate();
```

Можно прикреплять эталонные изображения:

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

Сгенерированные изображения легко сохранить на диск по умолчанию, настроенный в `config/filesystems.php`:

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

Также можно сгенерировать аудио из строки с помощью метода `toAudio`, доступного в классе Laravel `Stringable`:

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

Сгенерированное аудио можно сохранить на диск по умолчанию, настроенный в конфигурационном файле приложения `config/filesystems.php`:

```php
$audio = Audio::of('I love coding with Laravel.')->generate();

$path = $audio->store();
$path = $audio->storeAs('audio.mp3');
$path = $audio->storePublicly();
$path = $audio->storePubliclyAs('audio.mp3');
```

Генерацию аудио также можно выполнять через очередь:

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Responses\AudioResponse;

Audio::of('I love coding with Laravel.')
    ->queue()
    ->then(function (AudioResponse $audio) {
        $path = $audio->store();

        // ...
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

<a name="text-summarization"></a>
## Суммаризация текста

Текст можно суммаризировать с помощью метода `summarize`, доступного через класс Laravel `Stringable`. По умолчанию summary будет содержать не более трех предложений и будет сгенерирован с использованием самой дешевой текстовой модели настроенного провайдера:

```php
use Illuminate\Support\Str;

$summary = Str::of($article)->summarize();
```

Можно указать максимальное количество предложений, провайдера, модель и timeout, используемые для генерации summary. Класс `Str` также предоставляет статическую версию метода:

```php
use Laravel\Ai\Enums\Lab;

$summary = Str::of($article)->summarize(
    sentences: 4,
    provider: Lab::Anthropic,
    model: 'claude-sonnet-5',
    timeout: 30,
);

$summary = Str::summarize($article, sentences: 4);
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

<a name="multimodal-embeddings"></a>
### Мультимодальные embeddings

Помимо строк, метод `Embeddings::for` принимает изображения, аудио, документы и видео, позволяя генерировать embeddings для нетекстового контента. Gemini поддерживает embeddings для изображений, аудио, документов и видео, а VoyageAI - для изображений и видео:

```php
use Laravel\Ai\Embeddings;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Files\Image;
use Laravel\Ai\Files\Video;

$response = Embeddings::for([
    'A vineyard at sunset.',
    Image::fromStorage('vineyard.jpg'),
    Video::fromPath('/home/laravel/tour.mp4'),
])->generate(Lab::Gemini);
```

Мультимодальные входные данные используют те же [классы файлов, что и attachments](#attachments). Такие файлы можно создать из локального пути, filesystem disk, удаленного URL или Base64-encoded content. Изображения, документы и видео также можно создавать из uploaded files, а документы - из raw string content:

```php
use Laravel\Ai\Files\Audio;
use Laravel\Ai\Files\Document;
use Laravel\Ai\Files\Image;
use Laravel\Ai\Files\Video;

Image::fromPath('/home/laravel/photo.jpg');
Image::fromStorage('photo.jpg');
Image::fromUpload($request->file('photo'));

Audio::fromPath('/home/laravel/clip.mp3');
Audio::fromStorage('clip.mp3');
Audio::fromUpload($request->file('clip.mp3'));

Video::fromPath('/home/laravel/video.mp4');
Video::fromStorage('video.mp4');
Video::fromUpload($request->file('video'));

Document::fromUrl('https://example.com/report.pdf');
Document::fromString('Laravel is a PHP framework.', 'text/plain');
Document::fromUpload($request->file('report'));
```

> [!NOTE]
> VoyageAI не позволяет смешивать медиа по удаленным URL и Base64-encoded media в одном запросе. Локальные, сохраненные и загруженные файлы отправляются как Base64-encoded content, а текстовые входные данные можно комбинировать с любым источником медиа. Обратитесь к документации провайдера, чтобы узнать, какие мультимодальные модели и входные данные доступны.

<a name="querying-embeddings"></a>
### Запросы по embeddings

Обычно векторные представления сохраняются в столбце базы данных типа `vector` для последующих запросов. Laravel поддерживает такие столбцы PostgreSQL с помощью расширения `pgvector`:

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

Чтобы ускорить поиск по сходству, добавьте векторный индекс. При вызове `index` Laravel создаст индекс HNSW с косинусным расстоянием:

```php
$table->vector('embedding', dimensions: 1536)->index();
```

В Eloquent-модели приведите векторный столбец к типу `array`:

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

Если агенту требуется поиск по сходству в виде инструмента, обратитесь к разделу [«Поиск по сходству»](#similarity-search).

> [!NOTE]
> Векторные запросы сейчас поддерживаются только для соединений PostgreSQL с расширением `pgvector`.

<a name="caching-embeddings"></a>
### Кеширование векторных представлений

Генерацию векторных представлений можно кешировать, чтобы избежать повторных вызовов API для одинаковых входных данных. Для включения кеша установите параметр `ai.caching.embeddings.cache` в `true`:

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

Для реранжирования документов можно использовать класс `Laravel\Ai\Reranking`:

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

Также можно ограничить количество результатов и указать провайдера:

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

После сохранения файла на него можно ссылаться при генерации текста с помощью агентов, не загружая файл повторно:

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

По умолчанию `Files` использует AI-провайдера, заданного в `config/ai.php`. Для большинства операций можно указать другого провайдера:

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

После сохранения файла у провайдера на него можно ссылаться в разговорах агентов с помощью метода `fromId` класса `Document` или `Image`:

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
## Векторные хранилища

Векторные хранилища позволяют создавать доступные для поиска коллекции файлов для генерации с дополненной выборкой (RAG). Класс `Laravel\Ai\Stores` предоставляет методы создания, получения и удаления векторных хранилищ:

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

Удалить векторное хранилище можно с помощью `Stores::delete` или его экземпляра:

```php
use Laravel\Ai\Stores;

Stores::delete('store_id');

$store = Stores::get('store_id');

$store->delete();
```

<a name="adding-files-to-stores"></a>
### Добавление файлов в хранилища

После создания векторного хранилища добавьте в него [файлы](#files) методом `add`. Файлы автоматически индексируются для семантического поиска с помощью [инструмента провайдера для поиска файлов](#file-search):

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

> **Примечание:** Обычно при добавлении ранее сохранённого файла в векторное хранилище идентификатор возвращаемого документа совпадает с идентификатором файла, но некоторые провайдеры могут вернуть новый идентификатор документа. Поэтому рекомендуется хранить оба идентификатора в базе данных.

К файлам можно добавить метаданные, чтобы затем фильтровать результаты при использовании [инструмента провайдера для поиска файлов](#file-search):

```php
$store->add(Document::fromPath('/path/to/document.pdf'), metadata: [
    'author' => 'Taylor Otwell',
    'department' => 'Engineering',
    'year' => 2026,
]);
```

Удалить файл из хранилища можно методом `remove`:

```php
$store->remove('file_id');
```

Удаление файла из векторного хранилища не удаляет его из [файлового хранилища](#files) провайдера. Чтобы удалить файл из обоих хранилищ, используйте `deleteFile`:

```php
$store->remove('file_abc123', deleteFile: true);
```

<a name="failover"></a>
## Переключение при сбое

При обращении к агенту или генерации медиа можно передать массив провайдеров или моделей, чтобы автоматически переключаться на резервный вариант при сбое сервиса или превышении ограничения частоты запросов основного провайдера:

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

Переключение происходит только при возникновении `FailoverableException`, например при превышении ограничения частоты запросов (`RateLimitedException`), перегрузке или недоступности провайдера (`ProviderOverloadedException`) либо недостатке средств (`InsufficientCreditsException`). Обычные ошибки, например ошибка валидации или некорректный запрос, не запускают переключение.

При передаче простого списка провайдеров, например `[Lab::OpenAI, Lab::Anthropic]`, каждый из них использует свою модель по умолчанию. Чтобы указать конкретную модель для каждого провайдера в цепочке переключения, передайте ассоциативный массив с ключами провайдеров, используя свойство `value` перечисления `Lab` в качестве ключа (варианты перечисления нельзя напрямую использовать как ключи массива PHP):

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

Чтобы подменить ответы агента в тестах, вызовите метод `fake` его класса. Можно передать список ответов или замыкание:

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

При подмене агента со структурированным выводом можно передавать массивы в качестве ответов. Агент вернёт структурированный ответ с переданными данными:

```php
SalesCoach::fake([
    ['score' => 87],
]);
```

Также можно подменить ответ, ожидающий подтверждения инструмента:

```php
use Laravel\Ai\Approvals\PendingApproval;
use Laravel\Ai\Responses\AgentResponse;

FileAssistant::fake([
    AgentResponse::fakeWithPendingApprovals([
        new PendingApproval(
            id: 'call_abc',
            tool: 'DeleteFile',
            arguments: ['path' => 'invoice.pdf'],
            reason: 'This will permanently delete a file.',
        ),
    ]),
]);

$response = (new FileAssistant)->prompt('Delete the invoice.');

$response->hasPendingApprovals(); // true
```

> **Примечание:** Если `Agent::fake()` вызван для агента со структурированным выводом и поддельный вывод не был передан явно, Laravel автоматически сгенерирует поддельные данные, соответствующие схеме вывода.

После обращения к агенту можно проверить полученные запросы:

```php
use Laravel\Ai\Prompts\AgentPrompt;

SalesCoach::assertPrompted('Analyze this...');

SalesCoach::assertPrompted(function (AgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotPrompted('Missing prompt');

SalesCoach::assertNeverPrompted();
```

При проверке продолжения с подтверждениями можно изучить решения в запросе:

```php
use Laravel\Ai\Approvals\Decisions;
use Laravel\Ai\Prompts\AgentPrompt;

FileAssistant::fake();

(new FileAssistant)->prompt(Decisions::from([
    'call_abc' => true,
]));

FileAssistant::assertPrompted(function (AgentPrompt $prompt) {
    return $prompt->hasApprovalDecisions()
        && $prompt->approvalDecisions->get('call_abc')->isApproved();
});
```

Для вызовов, поставленных в очередь, используйте соответствующие методы проверки:

```php
use Laravel\Ai\QueuedAgentPrompt;

SalesCoach::assertQueued('Analyze this...');

SalesCoach::assertQueued(function (QueuedAgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotQueued('Missing prompt');

SalesCoach::assertNeverQueued();
```

Метод `preventStrayPrompts` гарантирует, что для каждого обращения к агенту задан поддельный ответ:

```php
SalesCoach::fake()->preventStrayPrompts();
```

<a name="testing-images"></a>
### Изображения

Генерацию изображений можно подменить методом `fake` класса `Image`. После этого можно выполнять различные проверки записанных запросов на генерацию изображений:

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

После генерации изображений можно проверить полученные запросы:

```php
Image::assertGenerated(function (ImagePrompt $prompt) {
    return $prompt->contains('sunset') && $prompt->isLandscape();
});

Image::assertNotGenerated('Missing prompt');

Image::assertNothingGenerated();
```

Для генерации изображений через очередь используйте соответствующие методы проверки:

```php
Image::assertQueued(
    fn (QueuedImagePrompt $prompt) => $prompt->contains('sunset')
);

Image::assertNotQueued('Missing prompt');

Image::assertNothingQueued();
```

Чтобы убедиться, что для каждой генерации изображения задан поддельный ответ, используйте метод `preventStrayImages`. Если изображение будет сгенерировано без такого ответа, будет выброшено исключение:

```php
Image::fake()->preventStrayImages();
```

<a name="testing-audio"></a>
### Аудио

Генерацию аудио можно подменить методом `fake` класса `Audio`. После этого можно выполнять различные проверки записанных запросов на генерацию аудио:

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

После генерации аудио можно проверить полученные запросы:

```php
Audio::assertGenerated(function (AudioPrompt $prompt) {
    return $prompt->contains('Hello') && $prompt->isFemale();
});

Audio::assertNotGenerated('Missing prompt');
Audio::assertNothingGenerated();
```

Для генерации аудио через очередь используйте соответствующие методы проверки:

```php
Audio::assertQueued(
    fn (QueuedAudioPrompt $prompt) => $prompt->contains('Hello')
);

Audio::assertNotQueued('Missing prompt');
Audio::assertNothingQueued();
```

Чтобы убедиться, что для каждой генерации аудио задан поддельный ответ, используйте метод `preventStrayAudio`. Если аудио будет сгенерировано без такого ответа, будет выброшено исключение:

```php
Audio::fake()->preventStrayAudio();
```

<a name="testing-transcriptions"></a>
### Транскрипции

Генерацию транскрипций можно подменить методом `fake` класса `Transcription`. После этого можно выполнять различные проверки записанных запросов на создание транскрипций:

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

После создания транскрипций можно проверить полученные запросы:

```php
Transcription::assertGenerated(function (TranscriptionPrompt $prompt) {
    return $prompt->language === 'en' && $prompt->isDiarized();
});

Transcription::assertNotGenerated(
    fn (TranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingGenerated();
```

Для транскрипций, создаваемых через очередь, используйте соответствующие методы проверки:

```php
Transcription::assertQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->isDiarized()
);

Transcription::assertNotQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingQueued();
```

Чтобы убедиться, что для каждой транскрипции задан поддельный ответ, используйте метод `preventStrayTranscriptions`. Если транскрипция будет создана без такого ответа, будет выброшено исключение:

```php
Transcription::fake()->preventStrayTranscriptions();
```

<a name="testing-embeddings"></a>
### Embeddings

Генерацию векторных представлений можно подменить методом `fake` класса `Embeddings`. После этого можно выполнять различные проверки записанных запросов на их генерацию:

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

После генерации векторных представлений можно проверить полученные запросы:

```php
Embeddings::assertGenerated(function (EmbeddingsPrompt $prompt) {
    return $prompt->contains('Laravel') && $prompt->dimensions === 1536;
});

Embeddings::assertNotGenerated(
    fn (EmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingGenerated();
```

Для генерации векторных представлений через очередь используйте соответствующие методы проверки:

```php
Embeddings::assertQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Laravel')
);

Embeddings::assertNotQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingQueued();
```

Чтобы убедиться, что для каждой генерации векторных представлений задан поддельный ответ, используйте метод `preventStrayEmbeddings`. Если они будут сгенерированы без такого ответа, будет выброшено исключение:

```php
Embeddings::fake()->preventStrayEmbeddings();
```

<a name="testing-reranking"></a>
### Реранжирование

Операции реранжирования можно подменить методом `fake` класса `Reranking`:

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

После реранжирования можно проверить выполненные операции:

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

Файловые операции можно подменить методом `fake` класса `Files`:

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
### Векторные хранилища

Операции векторного хранилища можно подменить методом `fake` класса `Stores`. При этом автоматически будут подменены и [файловые операции](#files):

```php
use Laravel\Ai\Stores;

Stores::fake();
```

После этого можно проверять созданные или удалённые хранилища:

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

Чтобы проверить добавление или удаление файлов из хранилища, используйте методы проверки экземпляра `Store`:

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

Если файл одновременно сохраняется у провайдера и добавляется в векторное хранилище, идентификатор провайдера может быть неизвестен. В этом случае передайте замыкание в `assertAdded`:

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
- `ToolApprovalRequested`
- `ToolApprovalResolved`
- `ToolInvoked`
- `TranscriptionGenerated`

Вы можете слушать любые из этих событий, чтобы логировать или сохранять информацию об использовании AI SDK.
