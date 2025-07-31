---
git: 8035e585737b0e942bd6a92289e5bdb0cfa25823
---

# Laravel Passport

<a name="introduction"></a>
## Введение

[Laravel Passport](https://github.com/laravel/passport) обеспечивает полную реализацию сервера OAuth2 для вашего приложения Laravel за считанные минуты. Passport построен на основе [League OAuth2](https://github.com/thephpleague/oauth2-server), который поддерживается Энди Миллингтоном (Andy Millington) и Саймоном Хэмпом (Simon Hamp).

> [!NOTE]
> В этой документации предполагается, что вы уже знакомы с OAuth2. Если вы ничего не знаете о OAuth2, перед продолжением ознакомьтесь с общей [терминологией](https://oauth2.thephpleague.com/terminology/) и функциями OAuth2.

<a name="passport-or-sanctum"></a>
### Passport или Sanctum?

Прежде чем начать, вы можете определиться, будет ли ваше приложение лучше обслуживаться через Laravel Passport или [Laravel Sanctum](/docs/{{version}}/sanctum). Если вашему приложению необходима поддержка OAuth2, то следует использовать Laravel Passport.

Однако, если вы пытаетесь аутентифицировать одностраничное приложение, мобильное приложение или выдавать токены API, вам следует использовать [Laravel Sanctum](/docs/{{version}}/sanctum). Laravel Sanctum не поддерживает OAuth2; однако он обеспечивает гораздо более простой опыт разработки аутентификации API.

<a name="installation"></a>
## Установка

Вы можете установить Laravel Passport с помощью Artisan-команды `install:api`:

```shell
php artisan install:api --passport
```

Эта команда опубликует и запустит миграцию базы данных для создания таблиц, необходимых вашему приложению для хранения клиентов OAuth2 и токенов доступа. Команда также создаст ключи шифрования, необходимые для создания токенов безопасного доступа.

После выполнения команды `install:api` добавьте трейт `Laravel\Passport\HasApiTokens` и интерфейс `Laravel\Passport\Contracts\OAuthenticatable` в модель `App\Models\User`. Этот трейт предоставит вашей модели несколько вспомогательных методов, которые позволят вам проверять токен и области действия аутентифицированного пользователя:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

Наконец, в файле конфигурации приложения `config/auth.php` вы должны установить для параметра `driver` раздела `api` значение `passport`. Это укажет вашему приложению использовать Passport `TokenGuard` при аутентификации входящих запросов API:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

<a name="deploying-passport"></a>
### Развертывание Passport

При первом развертывании Passport на серверах вашего приложения вам, вероятно, потребуется выполнить команду `passport:keys`. Эта команда генерирует ключи шифрования, необходимые Passport для создания токенов доступа. Сгенерированные ключи обычно не хранятся в системе контроля версий:

```shell
php artisan passport:keys
```

При необходимости вы можете указать путь, откуда должны быть загружены ключи Passport. Для этого вы можете использовать метод `Passport::loadKeysFrom`. Обычно этот метод следует вызывать из метода `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
/**
 * Запустите любые службы приложений.
 */
public function boot(): void
{
    Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
}
```

<a name="loading-keys-from-the-environment"></a>
#### Загрузка ключей из окружения

В качестве альтернативы вы можете опубликовать файл конфигурации Passport с помощью Artisan-команды `vendor:publish`:

```shell
php artisan vendor:publish --tag=passport-config
```

После публикации файла конфигурации вы можете загрузить ключи шифрования вашего приложения, определив их как переменные среды:

```ini
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

<a name="upgrading-passport"></a>
### Обновление Passport

При обновлении до новой основной версии Passport важно внимательно изучить [руководство по обновлению](https://github.com/laravel/passport/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Настройка

<a name="token-lifetimes"></a>
### Срок жизни токена

По умолчанию Passport выдает долговременные токены доступа, срок действия которых истекает через год. Если вы хотите настроить более длительный / более короткий срок жизни токена, вы можете использовать методы `tokensExpireIn`, `refreshTokensExpireIn` и `personalAccessTokensExpireIn`. Эти методы следует вызывать из метода `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
use Carbon\CarbonInterval;

/**
 * Запустите любые службы приложений.
 */
public function boot(): void
{
    Passport::tokensExpireIn(CarbonInterval::days(15));
    Passport::refreshTokensExpireIn(CarbonInterval::days(30));
    Passport::personalAccessTokensExpireIn(CarbonInterval::months(6));
}
```

> [!WARNING]
> Поля `expires_at` в таблицах базы данных Passport доступны только для чтения и только для отображения. При выпуске токенов Passport сохраняет информацию об истечении срока действия в подписанных и зашифрованных токенах. Если вам нужно сделать токен недействительным, вы должны [отозвать его](#revoking-tokens).

<a name="overriding-default-models"></a>
### Переопределение моделей по умолчанию

Вы можете свободно расширять модели, используемые внутри Passport, определяя свою собственную модель и расширяя соответствующую модель Passport:

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

После определения модели вы можете указать Passport использовать вашу пользовательскую модель через класс `Laravel\Passport\Passport`. Как правило, вы должны сообщить Passport о ваших пользовательских моделях в методе `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\DeviceCode;
use App\Models\Passport\RefreshToken;
use App\Models\Passport\Token;
use Laravel\Passport\Passport;

/**
 * Запустите любые службы приложений.
 */
public function boot(): void
{
    Passport::useTokenModel(Token::class);
    Passport::useRefreshTokenModel(RefreshToken::class);
    Passport::useAuthCodeModel(AuthCode::class);
    Passport::useClientModel(Client::class);
    Passport::useDeviceCodeModel(DeviceCode::class);
}
```

<a name="overriding-routes"></a>
### Переопределение маршрутов

Иногда может возникнуть необходимость настроить маршруты, определенные Passport. Для этого сначала нужно игнорировать маршруты, зарегистрированные Passport, добавив `Passport::ignoreRoutes()` в метод `register` класса `AppServiceProvider` вашего приложения:

```php
use Laravel\Passport\Passport;

/**
 * Регистрация любых сервисов приложения.
 */
public function register(): void
{
    Passport::ignoreRoutes();
}
```

Затем вы можете скопировать маршруты, определенные Passport, из [его файла маршрутов](https://github.com/laravel/passport/blob/master/routes/web.php) в файл `routes/web.php` вашего приложения и изменить их по своему усмотрению:

```php
Route::group([
    'as' => 'passport.',
    'prefix' => config('passport.path', 'oauth'),
    'namespace' => '\Laravel\Passport\Http\Controllers',
], function () {
    // Маршруты Passport...
});
```

Этот подход позволяет вам полностью контролировать маршруты, связанные с Passport, и настраивать их в соответствии с потребностями вашего приложения.

<a name="authorization-code-grant"></a>
## Предоставление кода авторизации

Использование OAuth2 через коды авторизации — это то, через что большинство разработчиков знакомится с OAuth2. При использовании кодов авторизации клиентское приложение перенаправит пользователя на ваш сервер, где он либо утвердит, либо отклонит запрос на выдачу токена доступа клиенту.

Для начала нам нужно указать Passport, как возвращать наше «авторизованное» представление.

Вся логика рендеринга представления авторизации может быть настроена с помощью соответствующих методов, доступных в классе `Laravel\Passport\Passport`. Как правило, этот метод следует вызывать из метода `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // By providing a view name...
    Passport::authorizationView('auth.oauth.authorize');

    // By providing a closure...
    Passport::authorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );
}
```

Passport автоматически определит маршрут `/oauth/authorize`, возвращающий это представление. Ваш шаблон `auth.oauth.authorize` должен включать форму, которая отправляет POST-запрос к маршруту `passport.authorizations.approve` для подтверждения авторизации, и форму, которая отправляет DELETE-запрос к маршруту `passport.authorizations.deny` для отклонения авторизации. Маршруты `passport.authorizations.approve` и `passport.authorizations.deny` ожидают поля `state`, `client_id` и `auth_token`.

<a name="managing-clients"></a>
### Управление клиентами

Разработчикам приложений, которым необходимо взаимодействовать с API вашего приложения, необходимо зарегистрировать своё приложение в вашем приложении, создав «клиента». Обычно это включает в себя указание имени приложения и URI, на который ваше приложение будет перенаправлять пользователей после одобрения их запроса на авторизацию.

<a name="managing-first-party-clients"></a>
#### Собственные клиенты

Самый простой способ создать клиент — использовать команду Artisan `passport:client`. Эта команда может использоваться для создания собственных клиентов или тестирования функциональности OAuth2. При запуске команды `passport:client` Passport запросит у вас дополнительную информацию о клиенте и предоставит идентификатор и секретный ключ клиента:

```shell
php artisan passport:client
```

Если вы хотите разрешить несколько URI перенаправления для вашего клиента, вы можете указать их, разделив запятыми, в запросе URI командой `passport:client`. Все URI, содержащие запятые, должны быть закодированы в URI:

```shell
https://third-party-app.com/callback,https://example.com/oauth/redirect
```

<a name="managing-third-party-clients"></a>
#### Сторонние клиенты

Поскольку пользователи вашего приложения не смогут использовать команду `passport:client`, вы можете использовать метод `createAuthorizationCodeGrantClient` класса `Laravel\Passport\ClientRepository` для регистрации клиента для заданного пользователя:

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

// Создание клиента приложения OAuth, принадлежащего данному пользователю...
$client = app(ClientRepository::class)->createAuthorizationCodeGrantClient(
    user: $user,
    name: 'Example App',
    redirectUris: ['https://third-party-app.com/callback'],
    confidential: false,
    enableDeviceFlow: true
);

// Получение всех клиентов приложения OAuth, принадлежащих пользователю...
$clients = $user->oauthApps()->get();
```

Метод `createAuthorizationCodeGrantClient` возвращает экземпляр `Laravel\Passport\Client`. Вы можете отобразить `$client->id` в качестве идентификатора клиента, а `$client->plainSecret` — в качестве секретного ключа клиента.

<a name="requesting-tokens"></a>
### Запрос токенов

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### Перенаправление для авторизации

После создания клиента разработчики могут использовать свой идентификатор клиента и секретный ключ, чтобы запросить код авторизации и токен доступа из вашего приложения. Во-первых, приложение-потребитель должно сделать запрос перенаправления на маршрут вашего приложения `/oauth/authorize` следующим образом:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('http://passport-app.test/oauth/authorize?'.$query);
});
```

Параметр `prompt` может использоваться для определения поведения аутентификации в приложении Passport.

Если значение `prompt` равно `none`, Passport всегда будет выдавать ошибку аутентификации, если пользователь не аутентифицирован в приложении Passport. Если значение равно `consent`, Passport всегда будет отображать экран одобрения авторизации, даже если все разрешения были ранее предоставлены потребляющему приложению. Когда значение равно `login`, приложение Passport всегда будет предлагать пользователю повторно войти в систему, даже если у него уже есть активная сессия.

Если значение `prompt` не указано, пользователь будет приглашен к авторизации только в том случае, если он ранее не авторизовал доступ потребляющему приложению для запрашиваемых разрешений.

> [!NOTE]
> Помните, что маршрут `/oauth/authorize` уже определен в Passport. Вам не нужно вручную определять этот маршрут.

<a name="approving-the-request"></a>
#### Подтверждение запроса

При получении запросов на авторизацию Passport автоматически реагирует в соответствии со значением параметра `prompt` (если он присутствует) и может отображать пользователю шаблон, позволяющий одобрить или отклонить запрос на авторизацию. Если пользователь одобряет запрос, он будет перенаправлен обратно на `redirect_uri`, который был указан потребляющим приложением. `redirect_uri` должен соответствовать URL-адресу перенаправления, который был указан при создании клиента.

Иногда вам может понадобиться пропустить запрос на авторизацию, например, при авторизации основного клиента. Вы можете сделать это, [расширив модель `Client`](#overriding-default-models) и определив метод `skipsAuthorization`. Если `skipsAuthorization` возвращает `true`, клиент будет автоматически одобрен, и пользователь будет немедленно перенаправлен обратно на `redirect_uri`, за исключением случаев, когда потребляющее приложение явно установило параметр `prompt` при перенаправлении на авторизацию:

```php
<?php

namespace App\Models\Passport;

use Illuminate\Contracts\Auth\Authenticatable;
use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * Определите, должен ли клиент пропускать запрос авторизации.
     *
     * @param  \Laravel\Passport\Scope[]  $scopes
     */
    public function skipsAuthorization(Authenticatable $user, array $scopes): bool
    {
        return $this->firstParty();
    }
}
```

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### Преобразование кодов авторизации в токены доступа

Если пользователь одобряет запрос авторизации, он будет перенаправлен обратно в приложение-потребитель. Потребитель должен сначала сверить параметр `state` со значением, которое было сохранено до перенаправления. Если параметр `state` совпадает, то потребитель должен отправить вашему приложению запрос `POST`, чтобы запросить токен доступа. Запрос должен включать код авторизации, который был выдан вашим приложением, когда пользователь утвердил запрос авторизации:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class,
        'Invalid state value.'
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json();
});
```

Маршрут `/oauth/token` вернет ответ JSON, содержащий атрибуты `access_token`, `refresh_token` и `expires_in`. Атрибут `expires_in` содержит количество секунд до истечения срока действия токена доступа.

> [!NOTE]
> Как и маршрут `/oauth/authorize`, маршрут `/oauth/token` определяется для вас методом `Passport::routes`. Нет необходимости определять этот маршрут вручную.

<a name="managing-tokens"></a>
### Управление токенами

Вы можете получить авторизованные токены пользователя, используя метод `tokens` трейта `Laravel\Passport\HasApiTokens`. Например, это может быть использовано, чтобы предоставить вашим пользователям панель управления для отслеживания их подключений к сторонним приложениям:

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// Получение всех действительных токенов для пользователя...
$tokens = $user->tokens()
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get();

// Получение всех подключений пользователя к сторонним клиентам приложений OAuth...
$connections = $tokens->load('client')
    ->reject(fn (Token $token) => $token->client->firstParty())
    ->groupBy('client_id')
    ->map(fn (Collection $tokens) => [
        'client' => $tokens->first()->client,
        'scopes' => $tokens->pluck('scopes')->flatten()->unique()->values()->all(),
        'tokens_count' => $tokens->count(),
    ])
    ->values();
```

<a name="refreshing-tokens"></a>
### Обновление токенов

Если ваше приложение выдает недолговечные токены доступа, пользователям потребуется обновить свои токены доступа с помощью токена обновления, предоставленного им при выдаче токена доступа:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

Этот маршрут `/oauth/token` вернет ответ JSON, содержащий атрибуты `access_token`, `refresh_token` и `expires_in`. Атрибут `expires_in` содержит количество секунд до истечения срока действия токена доступа.

<a name="revoking-tokens"></a>
### Отзыв токенов

Вы можете отозвать токен, используя метод `revoke` модели `Laravel\Passport\Token`. Вы можете отозвать токен обновления токена, используя метод `revoke` модели `Laravel\Passport\RefreshToken`:

```php
use Laravel\Passport\Passport;
use Laravel\Passport\Token;

$token = Passport::token()->find($tokenId);

// Отозвать токен доступа...
$token->revoke();

// Отозвать токен обновления токена...
$token->refreshToken?->revoke();

// Отозвать все токены пользователя...
User::find($userId)->tokens()->each(function (Token $token) {
    $token->revoke();
    $token->refreshToken?->revoke();
});
```

<a name="purging-tokens"></a>
### Удаление токенов

Когда токены были отозваны или срок их действия истек, вы можете удалить их из базы данных. Команда `passport:purge` Artisan, содержащаяся в Passport, может сделать это за вас:

```shell
# Удалить отозванные и просроченные токены, коды авторизации и коды устройств...
php artisan passport:purge

# Удалить токены срок действия которых истек более чем на 6 часов назад...
php artisan passport:purge --hours=6

# Удалить только отозванные токены, коды авторизации и коды устройств...
php artisan passport:purge --revoked

# Удалить только просроченные токены, коды авторизации и коды устройств...
php artisan passport:purge --expired
```

Вы также можете настроить [запланированное задание](/docs/{{version}}/scheduling) в файле вашего приложения `routes/console.php` для автоматического удаления токенов по расписанию:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('passport:purge')->hourly();
```

<a name="code-grant-pkce"></a>
## Предоставление кода авторизации с помощью PKCE

Предоставление кода авторизации с `Proof Key for Code Exchange` (PKCE) - это безопасный способ аутентификации одностраничных или мобильных приложений для доступа к вашему API. Это разрешение следует использовать, когда вы не можете гарантировать, что секретный ключ клиента будет храниться конфиденциально, или, чтобы уменьшить угрозу перехвата кода авторизации злоумышленником. Комбинация `code verifier` и `code challenge` заменяет секретный ключ клиента при замене кода авторизации на токен доступа.

<a name="creating-a-auth-pkce-grant-client"></a>
### Создание клиента

Прежде чем ваше приложение сможет выдавать токены через предоставление кода авторизации с помощью PKCE, вам необходимо создать клиента с поддержкой PKCE. Вы можете сделать это с помощью Artisan-команды `passport:client` с параметром `--public`:

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### Запрос токенов

<a name="code-verifier-code-challenge"></a>
#### Code Verifier & Code Challenge

Поскольку это разрешение на авторизацию не предоставляет секретный ключ клиента, разработчикам необходимо сгенерировать комбинацию `code verifier` и `code challenge`, чтобы запросить токен.

Средство проверки кода должно представлять собой случайную строку от 43 до 128 символов, содержащую буквы, цифры и символы `"-"`, `"."`, `"_"`, `"~"`, как определено в [спецификации RFC 7636](https://tools.ietf.org/html/rfc7636).

Итоговым результатом должна быть строка в кодировке Base64 с URL-адресом и безопасными для имени файла символами. Завершающие символы '=' должны быть удалены, и не должно быть разрывов строк, пробелов или других дополнительных символов.

```php
$encoded = base64_encode(hash('sha256', $code_verifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### Перенаправление для авторизации

После создания клиента вы можете использовать идентификатор клиента и сгенерированный `code verifier` и `code challenge`, чтобы запросить код авторизации и токен доступа из вашего приложения. Во-первых, приложение-потребитель должно сделать запрос перенаправления на маршрут вашего приложения `/oauth/authorize`:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $request->session()->put(
        'code_verifier', $code_verifier = Str::random(128)
    );

    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $code_verifier, true))
    , '='), '+/', '-_');

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        'code_challenge' => $codeChallenge,
        'code_challenge_method' => 'S256',
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('http://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### Преобразование кодов авторизации в токены доступа

Если пользователь одобряет запрос авторизации, он будет перенаправлен обратно в приложение-потребитель. Потребитель должен сверить параметр `state` со значением, которое было сохранено до перенаправления, как в стандартном предоставлении кода авторизации.

Если параметр состояния совпадает, потребитель должен отправить вашему приложению запрос `POST`, чтобы запросить токен доступа. Запрос должен включать код авторизации, который был выдан вашим приложением, когда пользователь утвердил запрос авторизации, вместе с первоначально сгенерированным верификатором кода:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    $codeVerifier = $request->session()->pull('code_verifier');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code_verifier' => $codeVerifier,
        'code' => $request->code,
    ]);

    return $response->json();
});
```

<a name="device-authorization-grant"></a>
## Предоставление авторизации устройству

Разрешение на авторизацию устройства OAuth2 позволяет устройствам ввода без браузера или с ограниченными возможностями, таким как телевизоры и игровые консоли, получать токен доступа путем обмена «кодом устройства». При использовании Device Flow клиент устройства предложит пользователю использовать дополнительное устройство, например компьютер или смартфон, и подключиться к вашему серверу, где он введет предоставленный «код пользователя» и либо одобрит, либо отклонит запрос на доступ.

Для начала нам нужно указать Passport, как возвращать наши представления «код пользователя» и «авторизация».

Вся логика рендеринга представления авторизации может быть настроена с помощью соответствующих методов, доступных в классе `Laravel\Passport\Passport`. Как правило, этот метод следует вызывать из метода `boot` класса `App\Providers\AppServiceProvider` вашего приложения.

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // Указав имя представления...
    Passport::deviceUserCodeView('auth.oauth.device.user-code');
    Passport::deviceAuthorizationView('auth.oauth.device.authorize');

    // Указав замыкание...
    Passport::deviceUserCodeView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/UserCode')
    );

    Passport::deviceAuthorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );

    // ...
}
```

Passport автоматически определит маршруты, возвращающие эти представления. Ваш шаблон `auth.oauth.device.user-code` должен включать форму, которая отправляет GET-запрос к маршруту `passport.device.authorizations.authorize`. Маршрут `passport.device.authorizations.authorize` ожидает параметр запроса `user_code`.

Ваш шаблон `auth.oauth.device.authorize` должен включать форму, которая отправляет POST-запрос к маршруту `passport.device.authorizations.approve` для подтверждения авторизации, и форму, которая отправляет DELETE-запрос к маршруту `passport.device.authorizations.deny` для отклонения авторизации. Маршруты `passport.device.authorizations.approve` и `passport.device.authorizations.deny` ожидают поля `state`, `client_id` и `auth_token`.

<a name="creating-a-device-authorization-grant-client"></a>
### Создание клиента предоставления авторизации устройства

Прежде чем ваше приложение сможет выдавать токены через предоставление авторизации устройству, вам необходимо создать клиент с поддержкой Device Flow. Это можно сделать с помощью команды Artisan `passport:client` с опцией `--device`. Эта команда создаст клиент с поддержкой Device Flow и предоставит вам идентификатор и секретный ключ клиента:

```shell
php artisan passport:client --device
```

Кроме того, вы можете использовать метод `createDeviceAuthorizationGrantClient` класса `ClientRepository` для регистрации стороннего клиента, принадлежащего данному пользователю:

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

$client = app(ClientRepository::class)->createDeviceAuthorizationGrantClient(
    user: $user,
    name: 'Example Device',
    confidential: false,
);
```

<a name="requesting-device-authorization-grant-tokens"></a>
### Запрос токенов

<a name="device-code"></a>
#### Запрос кода устройства

После создания клиента разработчики могут использовать идентификатор клиента для запроса кода устройства из вашего приложения. Сначала устройство-потребитель должно отправить POST-запрос к маршруту `/oauth/device/code` вашего приложения, чтобы запросить код устройства:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/device/code', [
    'client_id' => 'your-client-id',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

В результате будет возвращён JSON-ответ, содержащий атрибуты `device_code`, `user_code`, `verification_uri`, `interval` и `expires_in`. Атрибут `expires_in` содержит количество секунд до истечения срока действия кода устройства. Атрибут `interval` содержит количество секунд, которое устройство-потребитель должно ждать между запросами при опросе маршрута `/oauth/token`, чтобы избежать ошибок ограничения скорости.

> [!NOTE]
> Помните, что маршрут `/oauth/device/code` уже определён в Passport. Вам не нужно определять этот маршрут вручную.

<a name="user-code"></a>
#### Отображение URI проверки и кода пользователя

После получения запроса на код устройства потребляющее устройство должно проинструктировать пользователя использовать другое устройство, посетить предоставленный `verification_uri` и ввести `user_code`, чтобы одобрить запрос на авторизацию.

<a name="polling-token-request"></a>
#### Запрос опросного токена

Поскольку пользователь будет использовать отдельное устройство для предоставления (или запрета) доступа, устройство-потребитель должно опрашивать маршрут `/oauth/token` вашего приложения, чтобы определить, когда пользователь ответил на запрос. Устройство-потребитель должно использовать минимальный интервал опроса, указанный в JSON-ответе, при запросе кода устройства, чтобы избежать ошибок, связанных с ограничением частоты запросов:

```php
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Sleep;

$interval = 5;

do {
    Sleep::for($interval)->seconds();

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'urn:ietf:params:oauth:grant-type:device_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret', // Required for confidential clients only...
        'device_code' => 'the-device-code',
    ]);

    if ($response->json('error') === 'slow_down') {
        $interval += 5;
    }
} while (in_array($response->json('error'), ['authorization_pending', 'slow_down']));

return $response->json();
```

Если пользователь одобрил запрос авторизации, будет возвращён JSON-ответ, содержащий атрибуты `access_token`, `refresh_token` и `expires_in`. Атрибут `expires_in` содержит количество секунд до истечения срока действия токена доступа.

<a name="password-grant"></a>
## Предоставление пароля

> [!WARNING]
> Мы больше не рекомендуем использовать токены для предоставления пароля. Вместо этого вам следует выбрать [тип гаранта, который в настоящее время рекомендуется OAuth2 Server](https://oauth2.thephpleague.com/authorization-server/which-grant/).

Предоставление пароля OAuth2 позволяет другим сторонним клиентам, таким как мобильное приложение, получать токен доступа, используя адрес электронной почты / имя пользователя и пароль. Это позволяет вам безопасно выдавать токены доступа своим основным клиентам, не требуя от пользователей прохождения всего потока перенаправления кода авторизации OAuth2.

Чтобы включить предоставление пароля, вызовите метод `enablePasswordGrant` в методе `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
/**
 * Запустите любые службы приложения.
 */
public function boot(): void
{
    Passport::enablePasswordGrant();
}
```

<a name="creating-a-password-grant-client"></a>
### Создание токенов

Прежде чем ваше приложение сможет выдавать токены с помощью предоставления пароля, вам необходимо создать клиент предоставления пароля. Вы можете сделать это с помощью Artisan-команды `passport:client` с параметром `--password`.

```shell
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### Запрос токенов

После включения предоставления пароля и создания клиента с предоставлением пароля вы можете запросить токен доступа, отправив `POST` запрос на маршрут `/oauth/token` с адресом электронной почты и паролем пользователя. Помните, что этот маршрут уже зарегистрирован Passport, поэтому нет необходимости определять его вручную. Если запрос будет успешным, вы получите `access_token` и `refresh_token` в JSON-ответе от сервера:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Требуется только для конфиденциальных клиентов...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

> [!NOTE]
> Помните, токены доступа по умолчанию являются долгоживущими. Однако вы можете [настроить максимальное время жизни токена доступа](#configuration), если это необходимо.

<a name="requesting-all-scopes"></a>
### Запрос токена для всех областей

При использовании доступа по паролю или доступа с учетными данными клиента вы можете авторизовать токен для всех областей, поддерживаемых вашим приложением. Вы можете сделать это, указав `*` в параметре `scope`. При этом метод `can` экземпляра токена всегда будет возвращать `true`. Эта расширенная область может быть назначена только токену, который выпущен с использованием разрешений `password` или `client_credentials`:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Требуется только для конфиденциальных клиентов...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => '*',
]);
```

<a name="customizing-the-user-provider"></a>
### Настройка пользовательского провайдера

Если ваше приложение использует более одного [провайдера аутентификации пользователя](/docs/{{version}}/authentication#introduction), вы можете указать, какой провайдер использует клиент предоставления пароля, указав параметр `--provider` при создании клиента через команду `artisan passport:client --password`. Указанное имя провайдера должно соответствовать допустимому провайдеру, определенному в файле конфигурации приложения `config/auth.php`. Затем вы можете [защитить свой маршрут с помощью посредника](#multiple-authentication-guards), чтобы гарантировать, что авторизованы только пользователи из указанного провайдера.

<a name="customizing-the-username-field"></a>
### Настройка поля имени пользователя

При аутентификации с использованием предоставления пароля Passport будет использовать атрибут `email` вашей аутентифицируемой модели в качестве "username". Однако вы можете настроить это поведение, определив метод `findForPassport` в своей модели:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Возвращает экземпляр пользователя для переданного имени.
     */
    public function findForPassport(string $username): User
    {
        return $this->where('username', $username)->first();
    }
}
```

<a name="customizing-the-password-validation"></a>
### Настройка проверки пароля пользователя

При аутентификации с использованием предоставления пароля Passport будет использовать атрибут `password` модели для проверки пароля. Если модель не имеет атрибута `password` или вы хотите настроить логику проверки пароля, вы можете определить метод `validateForPassportPasswordGrant` в своей модели:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Проверьте пароль пользователя для предоставления разрешения.
     */
    public function validateForPassportPasswordGrant(string $password): bool
    {
        return Hash::check($password, $this->password);
    }
}
```

<a name="implicit-grant"></a>
## Неявное разрешение

> [!WARNING]
> Мы больше не рекомендуем использовать токены для предоставления пароля. Вместо этого вам следует выбрать [тип гаранта, который в настоящее время рекомендуется OAuth2 Server](https://oauth2.thephpleague.com/authorization-server/which-grant/).

Неявное разрешение аналогично предоставлению кода авторизации; однако токен возвращается клиенту без обмена кодом авторизации. Это разрешение чаще всего используется для JavaScript или мобильных приложений, где учетные данные клиента не могут быть надежно сохранены. Чтобы включить разрешение, вызовите метод `enableImplicitGrant` в методе `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
/**
 * Запустите любые службы приложения.
 */
public function boot(): void
{
    Passport::enableImplicitGrant();
}
```

Прежде чем ваше приложение сможет выдавать токены через неявное предоставление, вам необходимо создать клиент неявного предоставления. Это можно сделать с помощью Artisan-команды `passport:client` с опцией `--implicit`.

```shell
php artisan passport:client --implicit
```

После включения разрешения и создания неявного клиента разработчики могут использовать свой идентификатор клиента для запроса токена доступа из вашего приложения. Приложение-потребитель должно выполнить перенаправление на маршрут `/oauth/authorize` вашего приложения следующим образом:

```php
use Illuminate\Http\Request;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'token',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

> [!NOTE]
> Помните, что маршрут `/oauth/authorize` уже определен методом `Passport::routes`. Вам не нужно вручную определять этот маршрут.

<a name="client-credentials-grant"></a>
## Разрешение учетных данных

Предоставление учетных данных клиента подходит для межмашинной (machine-to-machine) аутентификации. Например, вы можете использовать это разрешение в запланированном задании, которое выполняет задачи обслуживания через API.

Прежде чем ваше приложение сможет выдавать токены с помощью предоставления учетных данных клиента, вам необходимо создать клиента предоставления учетных данных. Вы можете сделать это, используя параметр `--client` в Artisan-команде `passport:client`:

```shell
php artisan passport:client --client
```

Затем назначьте посредника `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner` маршруту:

```php
use Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner;

Route::get('/orders', function (Request $request) {
    // Токен доступа действителен, и клиент является владельцем ресурса...
})->middleware(EnsureClientIsResourceOwner::class);
```

Чтобы ограничить доступ к маршруту определенными областями, вы можете предоставить список требуемых областей методу `using`:

```php
Route::get('/orders', function (Request $request) {
    // Токен доступа действителен, клиент является владельцем ресурса и имеет обе области действия: «servers:read» и «servers:create»...
})->middleware(EnsureClientIsResourceOwner::using('servers:read', 'servers:create'));
```

<a name="retrieving-tokens"></a>
### Получение токенов

Чтобы получить токен с использованием этого типа разрешения, сделайте запрос к конечной точке `oauth/token`:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'scope' => 'servers:read servers:create',
]);

return $response->json()['access_token'];
```

<a name="personal-access-tokens"></a>
## Токены персонального доступа

Иногда ваши пользователи могут захотеть выдать себе токены доступа, не проходя типичный поток перенаправления кода авторизации. Разрешение пользователям выдавать себе токены через пользовательский интерфейс вашего приложения может быть полезно для предоставления пользователям возможности экспериментировать с вашим API или может служить более простым подходом к выдаче токенов доступа в целом.

> [!NOTE]
> Если ваше приложение использует Passport в основном для выдачи токенов личного доступа, рассмотрите возможность использования [Laravel Sanctum](/docs/{{version}}/sanctum), облегченной собственной библиотеки Laravel для выдачи токенов доступа к API.

<a name="creating-a-personal-access-client"></a>
### Создание токенов персонального доступа

Прежде чем ваше приложение сможет выдавать токены персонального доступа, вам необходимо создать клиента личного доступа. Вы можете сделать это, выполнив Artisan-команду `passport:client` с параметром `--personal`. Если вы уже выполнили команду `passport:install`, вам не нужно запускать эту команду:

```shell
php artisan passport:client --personal
```

<a name="customizing-the-user-provider-for-pat"></a>
### Настройка поставщика услуг для пользователей

Если ваше приложение использует более одного [поставщика аутентификации пользователей](/docs/{{version}}/authentication#introduction), вы можете указать, какой поставщик используется клиентом предоставления личного доступа, указав параметр `--provider` при создании клиента командой `artisan passport:client --personal`. Указанное имя поставщика должно соответствовать допустимому имени поставщика, указанному в файле конфигурации `config/auth.php` вашего приложения. После этого вы можете [защитить свой маршрут с помощью посредника](#multiple-authentication-guards), чтобы гарантировать авторизацию только пользователей указанного провайдера.

<a name="managing-personal-access-tokens"></a>
### Управление токенами персонального доступа

После того как вы создали клиент персонального доступа, вы можете выдавать токены для данного пользователя, используя метод `createToken` в экземпляре модели `App\Models\User`. Метод `createToken` принимает имя токена в качестве первого аргумента и необязательный массив [области](#token-scopes) в качестве второго аргумента:

```php
use App\Models\User;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// Создание токена без областей действия...
$token = $user->createToken('My Token')->accessToken;

// Создание токена с областями действия...
$token = $user->createToken('My Token', ['user:read', 'orders:create'])->accessToken;

// Создание токена со всеми областями действия...
$token = $user->createToken('My Token', ['*'])->accessToken;

// Извлечение всех действительных персональных токенов доступа, принадлежащих пользователю...
$tokens = $user->tokens()
    ->with('client')
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get()
    ->filter(fn (Token $token) => $token->client->hasGrantType('personal_access'));
```

<a name="protecting-routes"></a>
## Защита маршрутов

<a name="via-middleware"></a>
### Защита маршрутов через посредников

Паспорт включает в себя [защиту аутентификации](/docs/{{version}}/authentication#adding-custom-guards), которая проверяет токены доступа при входящих запросах. После того как вы настроили защиту `api` для использования драйвера `passport`, вам нужно указать посредника `auth:api` на всех маршрутах, для которых требуется действующий токен доступа:

```php
Route::get('/user', function () {
    // Доступ к этому маршруту могут получить только пользователи, прошедшие аутентификацию API...
})->middleware('auth:api');
```

> [!WARNING]
> Если вы используете [токены учетных данных](#client-credentials-grant), вы должны вместо этого использовать [посредник `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner`](#client-credentials-grant) для защиты ваших маршрутов `auth:api`.

<a name="multiple-authentication-guards"></a>
#### Множественная защита аутентификации

Если ваше приложение аутентифицирует разные типы пользователей, которые, возможно, используют совершенно разные модели Eloquent, вам, вероятно, потребуется определить конфигурацию защиты для каждого типа провайдера пользователей в вашем приложении. Это позволяет защитить запросы, предназначенные для конкретных поставщиков услуг. Например, при следующей конфигурации защиты конфигурационный файл `config/auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],
],
```

Следующий маршрут будет использовать защиту `api-customers`, которая использует провайдера пользователей `customers` для аутентификации входящих запросов:

```php
Route::get('/customer', function () {
    // ...
})->middleware('auth:api-customers');
```

> [!NOTE]
> Для получения дополнительной информации об использовании нескольких поставщиков пользователей с Passport обратитесь к [документации по персональным токенам доступа](#customizing-the-user-provider-for-pat) и [документации по предоставлению пароля](#customizing-the-user-provider).

<a name="passing-the-access-token"></a>
### Защита маршрутов через передачу токена

При вызове маршрутов, защищенных Passport, пользователи API вашего приложения должны указать свой токен доступа как токен Bearer в заголовке Authorization своего запроса. Например, при использовании фасада `Http`:

```php
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => "Bearer $accessToken",
])->get('https://passport-app.test/api/user');

return $response->json();
```

<a name="token-scopes"></a>
## Области токенов

Области позволяют вашим клиентам API запрашивать определенный набор разрешений при запросе авторизации для доступа к учетной записи. Например, если вы создаете приложение для электронной коммерции, не всем потребителям API потребуется возможность размещать заказы. Вместо этого вы можете разрешить потребителям запрашивать авторизацию только для доступа к статусам отгрузки заказа. Другими словами, области позволяют пользователям вашего приложения ограничивать действия, которые стороннее приложение может выполнять от их имени.

<a name="defining-scopes"></a>
### Определение областей

Вы можете определить области своего API, используя метод `Passport::tokensCan` в методе `boot` класса `App\Providers\AppServiceProvider` вашего приложения. Метод `tokensCan` принимает массив имен и описаний областей видимости. Описание области действия может быть любым, и оно будет отображаться для пользователей на экране утверждения авторизации:

```php
/**
 * Запустите любые службы приложения.
 */
public function boot(): void
{
    Passport::tokensCan([
        'user:read' => 'Retrieve the user info',
        'orders:create' => 'Place orders',
        'orders:read:status' => 'Check order status',
    ]);
}
```

<a name="default-scope"></a>
### Области по-умолчанию

Если клиент не запрашивает какие-либо конкретные области действия, вы можете настроить сервер Passport для присоединения областей действия по умолчанию к токену с помощью метода `defaultScopes`. Как правило, этот метод следует вызывать из метода `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
use Laravel\Passport\Passport;

Passport::tokensCan([
    'user:read' => 'Retrieve the user info',
    'orders:create' => 'Place orders',
    'orders:read:status' => 'Check order status',
]);

Passport::defaultScopes([
    'user:read',
    'orders:create',
]);
```

<a name="assigning-scopes-to-tokens"></a>
### Назначение областей токенам

<a name="when-requesting-authorization-codes"></a>
#### При запросе кодов авторизации

При запросе токена доступа с использованием предоставления кода авторизации потребители должны указать свои желаемые области в качестве параметра строки запроса `scope`. Параметр `scope` должен быть списком областей, разделенных пробелами:

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="when-issuing-personal-access-tokens"></a>
#### При выдаче токенов личного доступа

Если вы выдаете токены личного доступа с помощью метода `createToken` модели `App\Models\User`, вы можете передать массив желаемых областей в качестве второго аргумента метода:

```php
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

<a name="checking-scopes"></a>
### Проверка областей

В состав Passport входят два посредника, которые можно использовать для проверки подлинности входящего запроса с помощью токена, которому предоставлена ​​определенная область действия.

<a name="check-for-all-scopes"></a>
#### Проверка всех областей

Посредник `Laravel\Passport\Http\Middleware\CheckToken` может быть назначен маршруту для проверки того, что токен доступа входящего запроса имеет все перечисленные области действия:

```php
use Laravel\Passport\Http\Middleware\CheckToken;

Route::get('/orders', function () {
    // Access token has both "orders:read" and "orders:create" scopes...
})->middleware(['auth:api', CheckToken::using('orders:read', 'orders:create')]);
```

<a name="check-for-any-scopes"></a>
#### Проверка любых областей

Посредник `Laravel\Passport\Http\Middleware\CheckTokenForAnyScope` может быть назначен маршруту для проверки того, что токен доступа входящего запроса имеет *по крайней мере одну* из перечисленных областей:

```php
use Laravel\Passport\Http\Middleware\CheckTokenForAnyScope;

Route::get('/orders', function () {
    // Access token has either "orders:read" or "orders:create" scope...
})->middleware(['auth:api', CheckTokenForAnyScope::using('orders:read', 'orders:create')]);
```

<a name="checking-scopes-on-a-token-instance"></a>
#### Проверка областей на экземпляре токена

После того как запрос с аутентификацией токена доступа поступил в ваше приложение, вы все равно можете проверить, имеет ли токен заданную область действия, используя метод `tokenCan` в экземпляре `App\Models\User`:

```php
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    if ($request->user()->tokenCan('orders:create')) {
        // ...
    }
});
```

<a name="additional-scope-methods"></a>
#### Дополнительные методы области

Метод `scopeIds` вернет массив всех определенных идентификаторов / имен:

```php
use Laravel\Passport\Passport;

Passport::scopeIds();
```

Метод `scopes` вернет массив всех определенных областей как экземпляры `Laravel\Passport\Scope`:

```php
Passport::scopes();
```

Метод `scopesFor` вернет массив экземпляров `Laravel\Passport\Scope`, соответствующих указанным идентификаторам / именам:

```php
Passport::scopesFor(['user:read', 'orders:create']);
```

Вы можете определить, была ли определена область, используя метод `hasScope`:

```php
Passport::hasScope('orders:create');
```

<a name="spa-authentication"></a>
## Аутентификация SPA

При создании API может быть чрезвычайно полезно иметь возможность использовать собственный API из приложения JavaScript. Такой подход к разработке API позволяет вашему собственному приложению использовать тот же API, которым вы делитесь со всем миром. Один и тот же API может использоваться вашим веб-приложением, мобильными приложениями, сторонними приложениями и любыми SDK, которые вы можете публиковать в различных менеджерах пакетов.

Как правило, если вы хотите использовать свой API из своего приложения JavaScript, вам необходимо вручную отправить токен доступа в приложение и передавать его с каждым запросом к вашему приложению. Однако Passport включает в себя посредника, которое может сделать это за вас. Все, что вам нужно сделать, это добавить посредника `CreateFreshApiToken` в группу посредников `web` в файле `bootstrap/app.php` вашего приложения:

```php
use Laravel\Passport\Http\Middleware\CreateFreshApiToken;

->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        CreateFreshApiToken::class,
    ]);
})
```

> [!WARNING]
> Вы должны убедиться, что посредник `CreateFreshApiToken` является последним в списке ваших посредников указанных ранее.

Это посредник будет прикреплять файл cookie `laravel_token` к вашим исходящим ответам. Этот файл cookie содержит зашифрованный JWT, который Passport будет использовать для аутентификации запросов API от вашего приложения JavaScript. Время жизни JWT равно вашему значению конфигурации session.lifetime. Теперь, поскольку браузер автоматически отправляет cookie со всеми последующими запросами, вы можете делать запросы к API вашего приложения без явной передачи токена доступа:

```js
axios.get('/api/user')
    .then(response => {
        console.log(response.data);
    });
```

<a name="customizing-the-cookie-name"></a>
#### Настройка имени Cookie

При необходимости вы можете настроить имя файла cookie `laravel_token`, используя метод `Passport::cookie`. Обычно этот метод следует вызывать из метода `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
/**
 * Запустите любые службы приложения.
 */
public function boot(): void
{
    Passport::cookie('custom_name');
}
```

<a name="csrf-protection"></a>
#### CSRF защита

При использовании этого метода аутентификации вам необходимо убедиться, что в ваши запросы включен действительный заголовок токена CSRF. Стандартный шаблон Laravel JavaScript, входящий в состав скелетного приложения и всех стартовых наборов, включает экземпляр [Axios](https://github.com/axios/axios), который будет автоматически использовать зашифрованное значение cookie `XSRF-TOKEN` для отправки заголовка `X-XSRF-TOKEN` в запросах.

> [!NOTE]
> Если вы решите отправить заголовок `X-CSRF-TOKEN` вместо `X-XSRF-TOKEN`, вам нужно использовать незашифрованный токен, предоставленный `csrf_token()`.

<a name="events"></a>
## События

Passport вызывает события при выдаче токенов доступа и обновлении токенов. Вы можете [прослушивать эти события](/docs/{{version}}/events), чтобы сократить или отозвать другие токены доступа в вашей базе данных:

| Наименование события                          |
| --------------------------------------------- |
| `Laravel\Passport\Events\AccessTokenCreated`  |
| `Laravel\Passport\Events\AccessTokenRevoked`  |
| `Laravel\Passport\Events\RefreshTokenCreated` |

<a name="testing"></a>
## Тестирование

Метод `actingAs` Passport может использоваться для указания аутентифицированного в данный момент пользователя, а также его областей действия. Первым аргументом, передаваемым методу `actingAs`, является экземпляр пользователя, а вторым - массив областей видимости, которые должны быть предоставлены токену пользователя:

```php tab=Pest
use App\Models\User;
use Laravel\Passport\Passport;

test('orders can be created', function () {
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
});
```

```php tab=PHPUnit
use App\Models\User;
use Laravel\Passport\Passport;

public function test_orders_can_be_created(): void
{
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
}
```

Метод `actingAsClient` Passport может использоваться для указания аутентифицированного в данный момент клиента, а также его областей. Первым аргументом, передаваемым методу `actingAsClient`, является экземпляр клиента, а вторым — массив областей видимости, которые должны быть предоставлены токену клиента:

```php tab=Pest
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

test('servers can be retrieved', function () {
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function test_servers_can_be_retrieved(): void
{
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
}
```
