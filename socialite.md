---
git: 57ae1e7dbd4bda3bae24ce93e527f1807ae49a43
---

# Пакет Laravel Socialite

<a name="introduction"></a>
## Введение

Помимо типичной аутентификации на основе форм, Laravel также предлагает простой и удобный способ аутентификации через провайдеров OAuth с помощью [Laravel Socialite](https://github.com/laravel/socialite). Socialite в настоящее время поддерживает аутентификацию через Facebook, X, LinkedIn, Google, GitHub, GitLab, Bitbucket и Slack.

> [!NOTE]
> Адаптеры для других платформ перечислены на веб-сайте [Socialite Providers](https://socialiteproviders.com/), управляемом сообществом.

<a name="installation"></a>
## Установка

Чтобы начать работу с Socialite, используйте менеджер пакетов Composer, чтобы добавить пакет в зависимости вашего проекта:

```shell
composer require laravel/socialite
```

<a name="upgrading-socialite"></a>
## Обновление пакета Socialite

При обновлении Socialite важно внимательно изучить [руководство по обновлению](https://github.com/laravel/socialite/blob/master/UPGRADE).

<a name="configuration"></a>
## Конфигурирование

Перед использованием Socialite вам нужно будет добавить учетные данные для провайдеров OAuth, которые использует ваше приложение. Обычно эти учетные данные можно получить, создав "приложение разработчика" в панели управления службы, с которой вы будете аутентифицироваться.

Эти учетные данные должны быть размещены в файле конфигурации вашего приложения `config/services.php` и должны использовать ключ `facebook`, `x`, `linkedin-openid`, `google`, `github`, `gitlab`, `bitbucket`, `slack` или `slack-openid`, в зависимости от провайдеров, которые требуются вашему приложению:

```php
'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => 'http://example.com/callback-url',
],
```

> [!NOTE]
> Если параметр `redirect` содержит относительный путь, то он будет автоматически преобразован в абсолютный URL.

<a name="authentication"></a>
## Аутентификация

<a name="routing"></a>
### Маршрутизация

Для аутентификации пользователей с помощью провайдера OAuth вам понадобятся два маршрута: один для перенаправления пользователя к провайдеру OAuth, а другой для получения обратного вызова от провайдера после аутентификации. Пример ниже демонстрирует реализацию обоих маршрутов:

```php
use Laravel\Socialite\Socialite;

Route::get('/auth/redirect', function () {
    return Socialite::driver('github')->redirect();
});

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // $user->token
});
```

Метод `redirect` фасада `Socialite`, отвечает за перенаправление пользователя к провайдеру OAuth, в то время как метод `user` обрабатывает входящий запрос и получает информацию о пользователе от провайдера  после того, как запрос на аутентификацию будет подтверждён.

<a name="authentication-and-storage"></a>
### Аутентификация и хранение

После того как пользователь был получен от поставщика OAuth, вы можете определить, существует ли пользователь в базе данных вашего приложения и [аутентифицировать пользователя](/docs/{{version}}/authentication#authenticate-a-user-instance). Если пользователь не существует в базе данных вашего приложения, вы обычно создаете новую запись в своей базе данных:

```php
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Laravel\Socialite\Socialite;

Route::get('/auth/callback', function () {
    $githubUser = Socialite::driver('github')->user();

    $user = User::updateOrCreate([
        'github_id' => $githubUser->id,
    ], [
        'name' => $githubUser->name,
        'email' => $githubUser->email,
        'github_token' => $githubUser->token,
        'github_refresh_token' => $githubUser->refreshToken,
    ]);

    Auth::login($user);

    return redirect('/dashboard');
});
```

> [!NOTE]
> Для получения дополнительной информации о том, какая информация о пользователях доступна от конкретных поставщиков OAuth, обратитесь к документации по [получению сведений о пользователе](#retrieving-user-details).

<a name="access-scopes"></a>
### Права доступа

Перед перенаправлением пользователя вы можете использовать метод `scopes`, чтобы указать "scopes" (права/области) которые должны быть включены в запрос аутентификации.  Этот метод объединит все ранее указанные права с теми, которые указали вы:

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('github')
    ->scopes(['read:user', 'public_repo'])
    ->redirect();
```

Вы можете перезаписать все существующие права в запросе аутентификации, используя метод `setScopes`:

```php
return Socialite::driver('github')
    ->setScopes(['read:user', 'public_repo'])
    ->redirect();
```

<a name="slack-bot-scopes"></a>
### Права Slack Bot

API Slack предоставляет [разные типы токенов доступа](https://api.slack.com/authentication/token-types), каждый с собственным набором [прав](https://api.slack.com/scopes). Socialite совместим с обоими следующими типами токенов доступа Slack:

<!-- <div class="content-list" markdown="1"> -->

- Bot (prefixed with `xoxb-`)
- User (prefixed with `xoxp-`)

<!-- </div> -->

По умолчанию драйвер `slack` создаст токен `user`, и вызов метода `user` этого драйвера вернет данные пользователя.

Токены ботов в основном полезны, если ваше приложение будет отправлять уведомления во внешние рабочие пространства Slack, принадлежащие пользователям вашего приложения. Чтобы сгенерировать токен бота, вызовите метод `asBotUser` перед перенаправлением пользователя в Slack для аутентификации:

```php
return Socialite::driver('slack')
    ->asBotUser()
    ->setScopes(['chat:write', 'chat:write.public', 'chat:write.customize'])
    ->redirect();
```

Кроме того, вы должны вызвать метод `asBotUser` перед вызовом метода `user`, когда Slack перенаправляет пользователя обратно на ваше приложение после аутентификации:

```php
$user = Socialite::driver('slack')->asBotUser()->user();
```

При генерации токена бота метод `user` по-прежнему будет возвращать экземпляр `Laravel\Socialite\Two\User`, однако только свойство `token` будет заполнено. Этот токен можно сохранить, чтобы [отправлять уведомления в рабочие пространства Slack аутентифицированного пользователя](/docs/{{version}}/notifications#notifying-external-slack-workspaces).

<a name="optional-parameters"></a>
### Необязательные параметры

Некоторые провайдеры OAuth поддерживают другие необязательные параметры в запросе перенаправления. Чтобы включить в запрос любые необязательные параметры, вызовите метод `with` с ассоциативным массивом:

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('google')
    ->with(['hd' => 'example.com'])
    ->redirect();
```

> [!WARNING]
> При использовании метода `with` будьте осторожны, чтобы не передавать какие-либо зарезервированные ключевые слова, такие как `state` или `response_type`.

<a name="retrieving-user-details"></a>
## Получение сведений о пользователе

После того как пользователь будет перенаправлен обратно на ваш маршрут `callback` аутентификации вашего приложения, вы можете получить данные пользователя, используя метод `user` пакета Socialite. Объект пользователя, возвращаемый методом `user`, содержит множество свойств и методов, которые вы можете использовать для сохранения информации о пользователе в вашей собственной базе данных.

Различные свойства и методы этого объекта могут быть доступны в зависимости от версии провайдера OAuth, с которым вы выполняете аутентификацию, OAuth 1.0 или OAuth 2.0:

```php
use Laravel\Socialite\Socialite;

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // Провайдер OAuth 2.0 ...
    $token = $user->token;
    $refreshToken = $user->refreshToken;
    $expiresIn = $user->expiresIn;

    // Провайдер OAuth 1.0 ...
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // Все провайдеры ...
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();
});
```

<a name="retrieving-user-details-from-a-token-oauth2"></a>
#### Получение сведений о пользователе из токена

Если у вас уже есть действительный токен доступа пользователя, то вы можете получить его данные с помощью метода `userFromToken` пакета Socialite:

```php
use Laravel\Socialite\Socialite;

$user = Socialite::driver('github')->userFromToken($token);
```

Если вы используете Facebook Limited Login в iOS-приложении, Facebook вернет OIDC-токен вместо токена доступа. Чтобы получить сведения о пользователе по OIDC-токену, передайте методу `userFromToken` значение nonce, использованное при запуске входа:

```php
$user = Socialite::driver('facebook')->userFromToken($token, $nonce);
```

<a name="stateless-authentication"></a>
#### Аутентификация без сохранения состояния

Метод `stateless` отключает проверку состояния сессии. Это полезно при добавлении социальной аутентификации в API без состояния, который не использует сессии на основе cookie:

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('google')->stateless()->user();
```

<a name="testing"></a>
## Тестирование

Laravel Socialite предоставляет удобный способ тестировать OAuth-флоу аутентификации без выполнения реальных запросов к OAuth-провайдерам. Метод `fake` позволяет имитировать поведение OAuth-провайдера и определить данные пользователя, которые должны быть возвращены.

<a name="faking-the-redirect"></a>
#### Имитация перенаправления

Чтобы проверить, что ваше приложение корректно перенаправляет пользователей к OAuth-провайдеру, вы можете вызвать метод `fake` перед выполнением запроса к маршруту перенаправления. В этом случае Socialite вернет перенаправление на фиктивный URL авторизации вместо перенаправления к реальному OAuth-провайдеру:

```php
use Laravel\Socialite\Socialite;

test('user is redirected to github', function () {
    Socialite::fake('github');

    $response = $this->get('/auth/github/redirect');

    $response->assertRedirect();
});
```

<a name="faking-the-callback"></a>
#### Имитация обратного вызова

Чтобы протестировать маршрут обратного вызова вашего приложения, вы можете вызвать метод `fake` и передать экземпляр `User`, который должен быть возвращен, когда приложение запросит сведения о пользователе у провайдера. Экземпляр `User` можно создать с помощью метода `fake`:

```php
use Laravel\Socialite\Socialite;
use Laravel\Socialite\Two\User;

test('user can login with github', function () {
    Socialite::fake('github', User::fake([
        'id' => 'github-123',
        'name' => 'Jason Beggs',
        'email' => 'jason@example.com',
    ]));

    $response = $this->get('/auth/github/callback');

    $response->assertRedirect('/dashboard');

    $this->assertDatabaseHas('users', [
        'name' => 'Jason Beggs',
        'email' => 'jason@example.com',
        'github_id' => 'github-123',
    ]);
});
```

По умолчанию экземпляр `User` будет содержать фиктивные значения OAuth-токенов. При необходимости вы можете переопределить эти значения, передав дополнительные атрибуты методу `fake`:

```php
$fakeUser = User::fake([
    'id' => 'github-123',
    'name' => 'Jason Beggs',
    'email' => 'jason@example.com',
    'token' => 'fake-token',
    'refreshToken' => 'fake-refresh-token',
    'expiresIn' => 3600,
    'approvedScopes' => ['read', 'write'],
]);
```

Пользователей OAuth 1 можно имитировать с помощью класса `Laravel\Socialite\One\User`.
