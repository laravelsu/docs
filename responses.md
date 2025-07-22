---
git: b3c1a5ceea69156c002bab55039e48e28aaae0ad
---

# HTTP-ответы

<a name="creating-responses"></a>
## Создание ответов

<a name="strings-arrays"></a>
#### Строки и массивы

Все маршруты и контроллеры должны возвращать ответ, который будет отправлен обратно в браузер пользователя. Laravel предлагает несколько разных способов вернуть ответы. Самый простой ответ – это возврат строки из маршрута или контроллера. Фреймворк автоматически преобразует строку в полный HTTP-ответ:

```php
Route::get('/', function () {
    return 'Hello World';
});
```

Помимо возврата строк из ваших маршрутов и контроллеров, вы также можете возвращать массивы. Фреймворк автоматически преобразует массив в ответ JSON:

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

> [!NOTE]
> Знаете ли вы, что можете возвращать [коллекции Eloquent](/docs/{{version}}/eloquent-collections) из ваших маршрутов или контроллеров? Они будут автоматически преобразованы в JSON.

<a name="response-objects"></a>
#### Объекты ответа

Как правило, вы не просто будете возвращать строки или массивы из действий маршрута. Вместо этого вы вернете полные экземпляры `Illuminate\Http\Response` или [шаблоны](/docs/{{version}}/views).

Возврат полного экземпляра `Response` позволяет вам настроить код состояния и заголовки HTTP ответа. Экземпляр `Response` наследуется от класса `Symfony\Component\HttpFoundation\Response`, который содержит множество методов для построения ответов HTTP:

```php
Route::get('/home', function () {
    return response('Hello World', 200)
        ->header('Content-Type', 'text/plain');
});
```

<a name="eloquent-models-and-collections"></a>
#### Модели и коллекции Eloquent

По желанию можно вернуть модели и коллекции [Eloquent ORM](/docs/{{version}}/eloquent) прямо из ваших маршрутов и контроллеров. Когда вы это сделаете, Laravel автоматически преобразует модели и коллекции в ответы JSON, учитывая [скрытие атрибутов](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) модели:

```php
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

<a name="attaching-headers-to-responses"></a>
### Добавление заголовков к ответам

Имейте в виду, что большинство методов ответа можно объединять в цепочку вызовов для гибкого создания экземпляров ответа. Например, вы можете использовать метод `header` для добавления серии заголовков к ответу перед его отправкой обратно пользователю:

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

Или вы можете использовать метод `withHeaders`, чтобы указать массив заголовков, которые будут добавлены к ответу:

```php
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```

<a name="cache-control-middleware"></a>
#### Посредник управления кешем

Laravel содержит посредник `cache.headers`, используемый для быстрой установки заголовка `Cache-Control` для группы маршрутов. Директивы должны быть предоставлены с использованием эквивалента "snake case" соответствующей директивы управления кешем и должны быть разделены точкой с запятой. Если в списке директив указан `etag`, то MD5-хеш содержимого ответа будет автоматически установлен как идентификатор ETag:

```php
Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

<a name="attaching-cookies-to-responses"></a>
### Добавление файлов Cookies к ответам

Вы можете добавить Cookies к исходящему экземпляру `Illuminate\Http\Response`, используя метод `cookie`. Вы должны передать этому методу имя, значение и количество минут, в течение которых куки должен считаться действительным:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

Метод `cookie` также принимает еще несколько аргументов, которые используются реже. Как правило, эти аргументы имеют то же назначение и значение, что и аргументы, передаваемые встроенному в PHP методу [`setcookie`](https://www.php.net/manual/ru/function.setcookie.php) method:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

Если вы хотите, чтобы куки отправлялся вместе с исходящим ответом, но у вас еще нет экземпляра этого ответа, вы можете использовать фасад `Cookie`, чтобы «поставить в очередь» файлы Cookies для добавления их к ответу при его отправке. Метод `queue` принимает аргументы, необходимые для создания экземпляра `Cookie`. Эти файлы Cookies будут добавлены к исходящему ответу перед его отправкой в браузер:

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

<a name="generating-cookie-instances"></a>
#### Создание экземпляров `Cookie`

Если вы хотите сгенерировать экземпляр `Symfony\Component\HttpFoundation\Cookie`, который может быть добавлен к экземпляру ответа позже, вы можете использовать глобальный помощник `cookie`. Этот файл Cookies не будет отправлен обратно клиенту, если он не прикреплен к экземпляру ответа:

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

<a name="expiring-cookies-early"></a>
#### Досрочное окончание срока действия файлов Cookies

Вы можете удалить куки, обнулив срок его действия с помощью метода `withoutCookie` исходящего ответа:

```php
return response('Hello World')->withoutCookie('name');
```

Если у вас еще нет экземпляра исходящего ответа, вы можете использовать метод `expire` фасада `Cookie` для обнуления срока действия кук:

```php
Cookie::expire('name');
```

<a name="cookies-and-encryption"></a>
### Файлы Cookies и шифрование

По умолчанию, благодаря middleware `Illuminate\Cookie\Middleware\EncryptCookies` все файлы Cookies, генерируемые Laravel, зашифрованы и подписаны, поэтому клиент не может их изменить или прочитать. Если вы хотите отключить шифрование для подмножества куки, созданных вашим приложением, вы можете использовать метод `encryptCookies` в файле `bootstrap/app.php` вашего приложения:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```

<a name="redirects"></a>
## Перенаправления

Ответы с перенаправлением являются экземплярами класса `Illuminate\Http\RedirectResponse` и содержат корректные заголовки, необходимые для перенаправления пользователя на другой URL. Есть несколько способов сгенерировать экземпляр `RedirectResponse`. Самый простой способ – использовать глобальный помощник `redirect`:

```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

По желанию можно перенаправить пользователя в его предыдущее местоположение, например, когда отправленная форма является недействительной. Вы можете сделать это с помощью глобального помощника `back`. Поскольку эта функция использует [сессии](/docs/{{version}}/session), убедитесь, что маршрут, вызывающий функцию `back`, использует группу посредников `web`:

```php
Route::post('/user/profile', function () {
    // Валидация запроса...

    return back()->withInput();
});
```

<a name="redirecting-named-routes"></a>
### Перенаправление на именованные маршруты

Когда вы вызываете помощник `redirect` без параметров, возвращается экземпляр `Illuminate\Routing\Redirector`, что позволяет вам вызывать любой метод экземпляра `Redirector`. Например, чтобы сгенерировать `RedirectResponse` на именованный маршрут, вы можете использовать метод `route`:

```php
return redirect()->route('login');
```

Если ваш маршрут имеет параметры, вы можете передать их в качестве второго аргумента методу `route`:

```php
// Для маршрута со следующим URI: /profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

<a name="populating-parameters-via-eloquent-models"></a>
#### Заполнение параметров с моделей Eloquent

Если вы перенаправляете на маршрут с параметром `ID`, который извлекается из модели Eloquent, то вы можете просто передать саму модель. ID будет извлечен автоматически:

```php
// Для маршрута со следующим URI: /profile/{id}

return redirect()->route('profile', [$user]);
```

Если вы хотите настроить значение, которое соответствует параметру маршрута, то вы можете указать столбец при определении параметра маршрута (`/profile/{id:slug}`) или переопределить метод `getRouteKey` в вашей модели Eloquent:

```php
/**
 * Получить значение ключа маршрута модели.
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
### Перенаправление к действиям контроллера

Вы также можете генерировать перенаправления на [действия контроллера](/docs/{{version}}/controllers). Для этого передайте имя контроллера и действия методу `action`:

```php
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```

Если ваш маршрут контроллера требует параметров, вы можете передать их в качестве второго аргумента методу `action`:

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

<a name="redirecting-external-domains"></a>
### Перенаправление на внешние домены

Иногда может потребоваться перенаправление на домен за пределами вашего приложения. Вы можете сделать это, вызвав метод `away`, который создает `RedirectResponse` без какой-либо дополнительной кодировки URL, валидации или проверки:

```php
return redirect()->away('https://www.google.com');
```

<a name="redirecting-with-flashed-session-data"></a>
### Перенаправление с кратковременным сохранением данных в сессии

Перенаправление на новый URL-адрес и [краткосрочная запись данных в сессию](/docs/{{version}}/session#flash-data) обычно выполняются одновременно. Обычно это делается после успешного выполнения действия, когда вы отправляете сообщение об успешном завершении в сессию. Для удобства вы можете создать экземпляр `RedirectResponse` и передать данные в сессию в единой текучей цепочке методов:

```php
Route::post('/user/profile', function () {
    // ...

    return redirect('/dashboard')->with('status', 'Profile updated!');
});
```

После перенаправления пользователя, вы можете отобразить сохраненное из [сессии](/docs/{{version}}/session) сообщение. Например, используя [синтаксис Blade](/docs/{{version}}/blade):

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

<a name="redirecting-with-input"></a>
#### Перенаправление с кратковременным сохранением входных данных

Вы можете использовать метод `withInput` экземпляра `RedirectResponse`, для передачи входных данных текущего запроса в сессию перед перенаправлением пользователя в новое место. Обычно это делается, если пользователь спровоцировал ошибку валидации. После того как входные данные были переданы в сессию, вы можете легко [получить их](/docs/{{version}}/requests#retrieving-old-input) во время следующего запроса для повторного автозаполнения формы:

```php
return back()->withInput();
```

<a name="other-response-types"></a>
## Другие типы ответов

Помощник `response` используется для генерации других типов экземпляров ответа. Когда помощник `response` вызывается без аргументов, возвращается реализация [контракта](/docs/{{version}}/contracts) `Illuminate\Contracts\Routing\ResponseFactory`. Этот контракт содержит несколько полезных методов для генерации ответов.

<a name="view-responses"></a>
### Ответы с HTML-шаблонами

Если вам нужен контроль над статусом и заголовками ответа, но также необходимо вернуть [HTML-шаблон](/docs/{{version}}/views) в качестве содержимого ответа, то вы должны использовать метод `view`:

```php
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```

Конечно, вы можете использовать глобальный помощник `view`, даже если вам не нужно передавать собственные код состояния или заголовки HTTP.

<a name="json-responses"></a>
### Ответы JSON

Метод `json` автоматически установит заголовок `Content-Type` в `application/json`, а также преобразует переданный массив в JSON с помощью функции `json_encode` PHP:

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

Если вы хотите создать ответ JSONP, вы можете использовать метод `json` в сочетании с методом `withCallback`:

```php
return response()
    ->json(['name' => 'Abigail', 'state' => 'CA'])
    ->withCallback($request->input('callback'));
```

<a name="file-downloads"></a>
### Ответы для загрузки файлов

Метод `download` используется для генерации ответа, который заставляет браузер пользователя загружать файл по указанному пути. Метод `download` принимает имя файла в качестве второго аргумента метода, определяющий имя файла, которое видит пользователь, загружающий файл. Наконец, вы можете передать массив заголовков HTTP в качестве третьего аргумента метода:

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

> [!WARNING]
> Symfony HttpFoundation, управляющий загрузкой файлов, требует, чтобы имя загружаемого файла было в кодировке ASCII.

<a name="file-responses"></a>
### Ответы на файлы

Метод `file` может использоваться для отображения файла, например изображения или PDF-файла, непосредственно в браузере пользователя вместо запуска загрузки. Этот метод принимает абсолютный путь к файлу в качестве первого аргумента и массив заголовков в качестве второго аргумента:

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

<a name="streamed-responses"></a>
## Потоковые ответы

Передавая данные клиенту по мере их создания, вы можете значительно сократить использование памяти и повысить производительность, особенно для очень больших ответов. Потоковые ответы позволяют клиенту начать обработку данных до того, как сервер завершит их отправку:

```php
Route::get('/stream', function () {
    return response()->stream(function (): void {
        foreach (['developer', 'admin'] as $string) {
            echo $string;
            ob_flush();
            flush();
            sleep(2); // Simulate delay between chunks...
        }
    }, 200, ['X-Accel-Buffering' => 'no']);
});
```

Для удобства, если замыкание, которое вы предоставляете методу `stream`, возвращает [Генератор](https://www.php.net/manual/en/language.generators.overview.php), Laravel автоматически очистит выходной буфер между строками, возвращаемыми генератором, а также отключит буферизацию вывода Nginx:

```php
Route::post('/chat', function () {
    return response()->stream(function (): Generator {
        $stream = OpenAI::client()->chat()->createStreamed(...);

         foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

<a name="consuming-streamed-responses"></a>
### Использование потоковых ответов

Потоковые ответы могут быть получены с помощью пакета `stream` npm Laravel, который предоставляет удобный API для взаимодействия с потоками ответов и событий Laravel. Для начала установите пакет `@laravel/stream-react` или `@laravel/stream-vue`:

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

Затем `useStream` может быть использован для обработки потока событий. После предоставления URL-адреса вашего потока перехватчик автоматически обновит `data` с помощью объединенного ответа по мере возврата содержимого из вашего приложения Laravel:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, isFetching, isStreaming, send } = useStream("chat");

    const sendMessage = () => {
        send({
            message: `Current timestamp: ${Date.now()}`,
        });
    };

    return (
        <div>
            <div>{data}</div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
            <button onClick={sendMessage}>Send Message</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
const { data, isFetching, isStreaming, send } = useStream("chat");
const sendMessage = () => {
    send({
        message: `Current timestamp: ${Date.now()}`,
    });
};
</script>
<template>
    <div>
        <div>{{ data }}</div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
        <button @click="sendMessage">Send Message</button>
    </div>
</template>
```

При отправке данных обратно в поток через `send`, активное соединение с потоком отменяется перед отправкой новых данных. Все запросы отправляются как запросы JSON `POST`.

> [!WARNING]
> Поскольку хук `useStream` делает запрос `POST` к вашему приложению, требуется действительный токен CSRF. Самый простой способ предоставить токен CSRF — это [включить его через тег meta в head макета вашего приложения](/docs/{{version}}/csrf#csrf-x-csrf-token).
Второй аргумент, заданный для `useStream`, — это объект параметров, который вы можете использовать для настройки поведения потребления потока. Значения по умолчанию для этого объекта показаны ниже:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        id: undefined,
        initialInput: undefined,
        headers: undefined,
        csrfToken: undefined,
        onResponse: (response: Response) => void,
        onData: (data: string) => void,
        onCancel: () => void,
        onFinish: () => void,
        onError: (error: Error) => void,
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
const { data } = useStream("chat", {
    id: undefined,
    initialInput: undefined,
    headers: undefined,
    csrfToken: undefined,
    onResponse: (response: Response) => void,
    onData: (data: string) => void,
    onCancel: () => void,
    onFinish: () => void,
    onError: (error: Error) => void,
});
</script>
<template>
    <div>{{ data }}</div>
</template>
```

`onResponse` срабатывает после успешного первоначального ответа от потока, и необработанный [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) передается в обратный вызов. `onData` вызывается при получении каждого фрагмента — текущий фрагмент передается в обратный вызов. `onFinish` вызывается, когда поток завершен и когда во время цикла выборки/чтения возникает ошибка.

По умолчанию запрос к потоку при инициализации не делается. Вы можете передать начальную полезную нагрузку потоку, используя опцию `initialInput`:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        initialInput: {
            message: "Introduce yourself.",
        },
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
const { data } = useStream("chat", {
    initialInput: {
        message: "Introduce yourself.",
    },
});
</script>
<template>
    <div>{{ data }}</div>
</template>
```

Чтобы отменить поток вручную, вы можете использовать метод `cancel`, возвращаемый из хука:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, cancel } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <button onClick={cancel}>Cancel</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
const { data, cancel } = useStream("chat");
</script>
<template>
    <div>
        <div>{{ data }}</div>
        <button @click="cancel">Cancel</button>
    </div>
</template>
```

Каждый раз, когда используется хук `useStream`, генерируется случайный `id` для идентификации потока. Он отправляется обратно на сервер с каждым запросом в заголовке `X-STREAM-ID`. При потреблении одного и того же потока из нескольких компонентов вы можете читать и писать в поток, предоставляя свой собственный `id`:

```tsx tab=React
// App.tsx
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, id } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <StreamStatus id={id} />
        </div>
    );
}

// StreamStatus.tsx
import { useStream } from "@laravel/stream-react";

function StreamStatus({ id }) {
    const { isFetching, isStreaming } = useStream("chat", { id });

    return (
        <div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
        </div>
    );
}
```

```vue tab=Vue
<!-- App.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
import StreamStatus from "./StreamStatus.vue";
const { data, id } = useStream("chat");
</script>
<template>
    <div>
        <div>{{ data }}</div>
        <StreamStatus :id="id" />
    </div>
</template>
<!-- StreamStatus.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
const props = defineProps<{
    id: string;
}>();
const { isFetching, isStreaming } = useStream("chat", { id: props.id });
</script>
<template>
    <div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
    </div>
</template>
```

<a name="streamed-json-responses"></a>
### Потоковые ответы JSON

Если вам нужно поэтапно передавать данные JSON, вы можете использовать метод `streamJson`. Этот метод особенно полезен для больших наборов данных, которые необходимо постепенно отправлять в браузер в формате, который можно легко проанализировать с помощью JavaScript:
```php
use App\Models\User;

Route::get('/users.json', function () {
    return response()->streamJson([
        'users' => User::cursor(),
    ]);
});
```

Хук `useJsonStream` идентичен хуку [useStream](#sumption-streamed-responses), за исключением того, что он попытается проанализировать данные как JSON после завершения потоковой передачи:

```tsx tab=React
import { useJsonStream } from "@laravel/stream-react";

type User = {
    id: number;
    name: string;
    email: string;
};

function App() {
    const { data, send } = useJsonStream<{ users: User[] }>("users");

    const loadUsers = () => {
        send({
            query: "taylor",
        });
    };

    return (
        <div>
            <ul>
                {data?.users.map((user) => (
                    <li>
                        {user.id}: {user.name}
                    </li>
                ))}
            </ul>
            <button onClick={loadUsers}>Load Users</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useJsonStream } from "@laravel/stream-vue";

type User = {
    id: number;
    name: string;
    email: string;
};
const { data, send } = useJsonStream<{ users: User[] }>("users");
const loadUsers = () => {
    send({
        query: "taylor",
    });
};
</script>
<template>
    <div>
        <ul>
            <li v-for="user in data?.users" :key="user.id">
                {{ user.id }}: {{ user.name }}
            </li>
        </ul>
        <button @click="loadUsers">Load Users</button>
    </div>
</template>
```

<a name="event-streams"></a>
### Потоки событий (SSE)

Метод `eventStream` может использоваться для возврата потокового ответа событий, отправленных сервером (SSE), с использованием типа контента `text/event-stream`. Метод `eventStream` принимает замыкание, которое должно [выдавать](https://www.php.net/manual/en/language.generators.overview.php) ответы потоку по мере их доступности:

```php
Route::get('/chat', function () {
    return response()->eventStream(function () {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

Если вы хотите настроить имя события, вы можете создать экземпляр класса `StreamedEvent`:

```php
use Illuminate\Http\StreamedEvent;

yield new StreamedEvent(
    event: 'update',
    data: $response->choices[0],
);
```

<a name="consuming-event-streams"></a>
#### Использование потоков событий

Потоки событий можно использовать с помощью пакета Laravel `stream` npm, который предоставляет удобный API для взаимодействия с потоками событий Laravel. Для начала установите пакет `@laravel/stream-react` или `@laravel/stream-vue`:

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

Затем для использования потока событий можно использовать `useEventStream`. После предоставления URL-адреса вашего потока перехватчик автоматически обновит `message` с помощью объединенного ответа по мере того, как сообщения будут возвращаться из вашего приложения Laravel:

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/chat");

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";
const { message } = useEventStream("/chat");
</script>
<template>
  <div>{{ message }}</div>
</template>
```

Второй аргумент, заданный параметру `useEventStream`, - это объект параметров, который вы можете использовать для настройки поведения при использовании потока. Значения по умолчанию для этого объекта приведены ниже:

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/stream", {
    eventName: "update",
    onMessage: (message) => {
      //
    },
    onError: (error) => {
      //
    },
    onComplete: () => {
      //
    },
    endSignal: "</stream>",
    glue: " ",
  });

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";
const { message } = useEventStream("/chat", {
  eventName: "update",
  onMessage: (message) => {
    // ...
  },
  onError: (error) => {
    // ...
  },
  onComplete: () => {
    // ...
  },
  endSignal: "</stream>",
  glue: " ",
});
</script>
```

Потоки событий также могут быть вручную использованы через объект [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) вашим фронтендом приложения. Метод `eventStream` автоматически отправит обновление `</stream>` в поток событий, когда поток завершится:

```js
const source = new EventSource('/chat');

source.addEventListener('update', (event) => {
    if (event.data === '</stream>') {
        source.close();

        return;
    }

    console.log(event.data);
});
```

Чтобы настроить конечное событие, отправляемое в поток событий, вы можете предоставить экземпляр `StreamedEvent` аргументу `endStreamWith` метода `eventStream`:

```php
return response()->eventStream(function () {
    // ...
}, endStreamWith: new StreamedEvent(event: 'update', data: '</stream>'));
```

<a name="streamed-downloads"></a>
### Потоковые загрузки

По желанию можно превратить строковый ответ переданной функции в загружаемый ответ без необходимости записывать результирующее содержимое на диск. В этом сценарии вы можете использовать метод `streamDownload`. Этот метод принимает в качестве аргументов замыкание, имя файла и необязательный массив заголовков:

```php
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
        ->contents()
        ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

<a name="response-macros"></a>
## Макрокоманды ответа

Если вы хотите определить собственный ответ, который вы можете повторно использовать в различных маршрутах и контроллерах, то вы можете использовать метод `macro` фасада `Response`. Как правило, этот метод следует вызывать в методе `boot` одного из [поставщиков служб](/docs/{{version}}/providers) вашего приложения, например, `App\Providers\AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Загрузка любых служб приложения.
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

Метод `macro` принимает имя как свой первый аргумент и замыкание – как второй аргумент. Замыкание макрокоманды будет выполнено при вызове имени макрокоманды из реализации `ResponseFactory` или глобального помощника `response`:

```php
return response()->caps('foo');
```
