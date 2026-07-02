---
git: e5a60cd792282de178478ca5bb678945e266b00b
---

# Стартовые комплекты

<a name="introduction"></a>
## Введение

Чтобы помочь вам быстрее начать работу с новым приложением Laravel, мы рады предложить [стартовые наборы приложений](https://laravel.com/starter-kits). Эти стартовые наборы дают вам фору при создании вашего следующего приложения Laravel и включают маршруты, контроллеры и представления, необходимые для регистрации и аутентификации пользователей вашего приложения. Стартовые наборы используют [Laravel Fortify](/docs/{{version}}/fortify) для аутентификации.

Использование стартовых комплектов необязательно — вы можете создать приложение с нуля, просто установив свежую копию Laravel. В любом случае, мы уверены, что у вас получится что-то замечательное!

<a name="creating-an-application"></a>
## Создание приложения с использованием стартового набора

Чтобы создать новое приложение Laravel с помощью одного из наших стартовых наборов, вам сначала следует [установить PHP и инструмент Laravel CLI](/docs/{{version}}/installation#installing-php). Если у вас уже установлены PHP и Composer, вы можете установить инструмент Laravel installer CLI через Composer:

```shell
composer global require laravel/installer
```

Затем создайте новое приложение Laravel с помощью CLI установщика Laravel. Установщик Laravel предложит вам выбрать предпочитаемый вами стартовый набор:

```shell
laravel new my-app
```

После создания приложения Laravel вам нужно только установить его зависимости внешнего интерфейса через NPM и запустить сервер разработки Laravel:

```shell
cd my-app
npm install && npm run build
composer run dev
```

После запуска сервера разработки Laravel ваше приложение будет доступно в веб-браузере по адресу [http://localhost:8000](http://localhost:8000).

<a name="available-starter-kits"></a>
## Доступные стартовые наборы

<a name="react"></a>
### React

Наш стартовый комплект React обеспечивает надежную, современную отправную точку для создания приложений Laravel с фронтендом React, использующим [Inertia](https://inertiajs.com).

Inertia позволяет вам создавать современные одностраничные приложения React, используя классическую маршрутизацию и контроллеры на стороне сервера. Это позволяет вам наслаждаться мощью фронтенда React в сочетании с невероятной производительностью бэкенда Laravel и молниеносной компиляцией Vite.

Стартовый комплект React использует React 19, TypeScript, Tailwind и библиотеку компонентов [shadcn/ui](https://ui.shadcn.com).

<a name="svelte"></a>
### Svelte

Наш стартовый комплект Svelte предоставляет надежную современную отправную точку для создания приложений Laravel с фронтендом Svelte, использующим [Inertia](https://inertiajs.com).

Inertia позволяет создавать современные одностраничные приложения Svelte с классической серверной маршрутизацией и контроллерами. Это позволяет объединить возможности Svelte на фронтенде с продуктивностью Laravel на бэкенде и быстрой компиляцией Vite.

Стартовый комплект Svelte использует Svelte 5, TypeScript, Tailwind и библиотеку компонентов [shadcn-svelte](https://www.shadcn-svelte.com/).

<a name="vue"></a>
### Vue

Наш стартовый комплект Vue представляет собой прекрасную отправную точку для создания приложений Laravel с интерфейсом Vue, использующим [Inertia](https://inertiajs.com).

Inertia позволяет вам создавать современные одностраничные приложения Vue, используя классическую маршрутизацию и контроллеры на стороне сервера. Это позволяет вам наслаждаться мощью фронтенда Vue в сочетании с невероятной производительностью бэкенда Laravel и молниеносной компиляцией Vite.

Стартовый комплект Vue использует API Vue Composition, TypeScript, Tailwind и библиотеку компонентов [shadcn-vue](https://www.shadcn-vue.com/).

<a name="livewire"></a>
### Livewire

Наш стартовый комплект Livewire представляет собой идеальную отправную точку для создания приложений Laravel с помощью интерфейса [Laravel Livewire](https://livewire.laravel.com).

Livewire — это мощный способ создания динамических, реактивных интерфейсов frontend с использованием только PHP. Он отлично подходит для команд, которые в основном используют шаблоны Blade и ищут более простую альтернативу фреймворкам SPA на основе JavaScript, таким как React, Svelte и Vue.

Стартовый комплект Livewire использует Livewire, Tailwind и библиотеку компонентов [Flux UI](https://fluxui.dev).

<a name="starter-kit-customization"></a>
## Настройка стартового набора

<a name="react-customization"></a>
### React

Наш стартовый набор React создан с использованием Inertia 3, React 19, Tailwind 4 и [shadcn/ui](https://ui.shadcn.com). Как и во всех наших стартовых наборах, весь код бэкенда и фронтенда находится в вашем приложении, что позволяет выполнять полную настройку.

Большая часть фронтенд-кода находится в каталоге `resources/js`. Вы можете свободно изменять любой код, чтобы настроить внешний вид и поведение вашего приложения:

```text
resources/js/
├── components/    # Повторно используемые компоненты React
├── hooks/         # React hooks
├── layouts/       # Макеты приложений
├── lib/           # Вспомогательные функции и конфигурация
├── pages/         # Компоненты страниц
└── types/         # Объявления типов
```

Чтобы опубликовать дополнительные компоненты shadcn, сначала [найдите компонент, который вы хотите опубликовать](https://ui.shadcn.com). Затем опубликуйте компонент с помощью `npx`:

```shell
npx shadcn@latest add switch
```

В этом примере команда опубликует компонент Switch в `resources/js/components/ui/switch.tsx`. После публикации компонента вы сможете использовать его на любой из своих страниц:

```jsx
import { Switch } from "@/components/ui/switch"

const MyPage = () => {
  return (
    <div>
      <Switch />
    </div>
  );
};

export default MyPage;
```

<a name="react-available-layouts"></a>
#### Доступные макеты React

Стартовый набор React включает в себя два различных основных макета на выбор: макет "sidebar" и макет "header". Макет "sidebar" используется по умолчанию, но вы можете переключиться на макет "header", изменив макет, импортированный в верхней части файла `resources/js/layouts/app-layout.tsx` вашего приложения:

```js
import AppLayoutTemplate from '@/layouts/app/app-sidebar-layout'; // [tl! remove]
import AppLayoutTemplate from '@/layouts/app/app-header-layout'; // [tl! add]
```

<a name="react-sidebar-variants"></a>
#### Варианты боковой панели React

Макет боковой панели включает три различных варианта: вариант "sidebar" по умолчанию, "inset" и "floating". Вы можете выбрать вариант, который вам больше нравится, изменив компонент `resources/js/components/app-sidebar.tsx`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="react-authentication-page-layout-variants"></a>
#### Варианты макета страницы аутентификации React

Страницы аутентификации, входящие в стартовый набор React, такие как страница входа и страница регистрации, также предлагают три различных варианта макета: "simple", "card" и "split".

Чтобы изменить макет аутентификации, измените макет, импортированный в верхней части файла `resources/js/layouts/auth-layout.tsx` вашего приложения:

```js
import AuthLayoutTemplate from '@/layouts/auth/auth-simple-layout'; // [tl! remove]
import AuthLayoutTemplate from '@/layouts/auth/auth-split-layout'; // [tl! add]
```

<a name="svelte-customization"></a>
### Svelte

Наш стартовый комплект Svelte создан с использованием Inertia 3, Svelte 5, Tailwind и [shadcn-svelte](https://www.shadcn-svelte.com/). Как и во всех стартовых комплектах, весь код бэкенда и фронтенда находится внутри вашего приложения, поэтому его можно полностью настраивать.

Большая часть фронтенд-кода находится в каталоге `resources/js`. Вы можете свободно изменять любой код, чтобы настроить внешний вид и поведение приложения:

```text
resources/js/
├── components/    # Повторно используемые компоненты Svelte
├── layouts/       # Макеты приложения
├── lib/           # Вспомогательные функции, конфигурация и rune-модули Svelte
├── pages/         # Компоненты страниц
└── types/         # Объявления TypeScript
```

Чтобы опубликовать дополнительные компоненты shadcn-svelte, сначала [найдите компонент, который хотите опубликовать](https://www.shadcn-svelte.com). Затем опубликуйте компонент с помощью `npx`:

```shell
npx shadcn-svelte@latest add switch
```

В этом примере команда опубликует компонент Switch в `resources/js/components/ui/switch/switch.svelte`. После публикации компонента вы сможете использовать его на любой странице:

```svelte
<script lang="ts">
    import { Switch } from '@/components/ui/switch'
</script>

<div>
    <Switch />
</div>
```

<a name="svelte-available-layouts"></a>
#### Доступные макеты Svelte

Стартовый комплект Svelte включает два основных макета: "sidebar" и "header". Макет "sidebar" используется по умолчанию, но вы можете переключиться на "header", изменив импорт в верхней части файла `resources/js/layouts/AppLayout.svelte`:

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.svelte'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.svelte'; // [tl! add]
```

<a name="svelte-sidebar-variants"></a>
#### Варианты боковой панели Svelte

Макет боковой панели включает три варианта: "sidebar" по умолчанию, "inset" и "floating". Вы можете выбрать нужный вариант, изменив компонент `resources/js/components/AppSidebar.svelte`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="svelte-authentication-page-layout-variants"></a>
#### Варианты макета страниц аутентификации Svelte

Страницы аутентификации стартового комплекта Svelte, такие как страница входа и регистрации, также предлагают три варианта макета: "simple", "card" и "split".

Чтобы изменить макет аутентификации, измените импорт в верхней части файла `resources/js/layouts/AuthLayout.svelte`:

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.svelte'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.svelte'; // [tl! add]
```

<a name="vue-customization"></a>
### Vue

Наш стартовый комплект Vue создан с использованием Inertia 3, Vue 3 Composition API, Tailwind и [shadcn-vue](https://www.shadcn-vue.com/). Как и во всех наших стартовых комплектах, весь код бэкенда и фронтенда находится в вашем приложении, что позволяет выполнять полную настройку.

Большая часть фронтенд-кода находится в каталоге `resources/js`. Вы можете свободно изменять любой код, чтобы настроить внешний вид и поведение вашего приложения:

```text
resources/js/
├── components/    # Повторно используемые компоненты Vue
├── composables/   # Vue composables / hooks
├── layouts/       # Макеты приложений
├── lib/           # Вспомогательные функции и конфигурация
├── pages/         # Компоненты страниц
└── types/         # Объявления типов
```

Чтобы опубликовать дополнительные компоненты shadcn-vue, сначала [найдите компонент, который вы хотите опубликовать](https://www.shadcn-vue.com). Затем опубликуйте компонент с помощью `npx`:

```shell
npx shadcn-vue@latest add switch
```

В этом примере команда опубликует компонент Switch в `resources/js/components/ui/Switch.vue`. После публикации компонента вы сможете использовать его на любой из своих страниц:

```vue
<script setup lang="ts">
import { Switch } from '@/components/ui/switch'
</script>

<template>
    <div>
        <Switch />
    </div>
</template>
```

<a name="vue-available-layouts"></a>
#### Доступные макеты Vue

Стартовый набор Vue включает в себя два различных основных макета на выбор: макет "sidebar" и макет "header". Макет "sidebar" используется по умолчанию, но вы можете переключиться на макет "header", изменив макет, импортированный в верхней части файла `resources/js/layouts/AppLayout.vue` вашего приложения:

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.vue'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.vue'; // [tl! add]
```

<a name="vue-sidebar-variants"></a>
#### Варианты боковой панели Vue

Макет боковой панели включает три различных варианта: вариант "sidebar" по умолчанию, "inset" и "floating". Вы можете выбрать вариант, который вам больше нравится, изменив компонент `resources/js/components/AppSidebar.vue`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="vue-authentication-page-layout-variants"></a>
#### Варианты макета страницы аутентификации Vue

Страницы аутентификации, входящие в стартовый комплект Vue, такие как страница входа и страница регистрации, также предлагают три различных варианта макета: "simple", "card" и "split".

Чтобы изменить макет аутентификации, измените макет, импортированный в верхней части файла `resources/js/layouts/AuthLayout.vue` вашего приложения:

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.vue'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.vue'; // [tl! add]
```

<a name="livewire-customization"></a>
### Livewire

Наш стартовый комплект Livewire создан с использованием Livewire 4, Tailwind и [Flux UI](https://fluxui.dev/). Как и во всех наших стартовых комплектах, весь код бэкенда и фронтенда находится в вашем приложении, что позволяет выполнять полную настройку.

Большая часть фронтенд-кода находится в каталоге `resources/views`. Вы можете свободно изменять любой код, чтобы настроить внешний вид и поведение вашего приложения:

```text
resources/views
├── components            # Повторно используемые компоненты
├── flux                  # Индивидуальные компоненты Flux
├── layouts               # Макеты приложения
├── pages                 # Страницы Livewire
├── partials              # Часто используемые представления Blade
├── dashboard.blade.php   # Панель управления аутентифицированного пользователя
├── welcome.blade.php     # Приветственная страница для гостей
```

<a name="livewire-available-layouts"></a>
#### Доступные макеты Livewire

Стартовый комплект Livewire включает в себя два различных основных макета на выбор: макет "sidebar" и макет "header". Макет "sidebar" используется по умолчанию, но вы можете переключиться на макет "header", изменив макет, используемый файлом `resources/views/layouts/app.blade.php` вашего приложения. Кроме того, вам следует добавить атрибут `container` к основному компоненту Flux:

```blade
<x-layouts::app.header>
    <flux:main container>
        {{ $slot }}
    </flux:main>
</x-layouts::app.header>
```

<a name="livewire-authentication-page-layout-variants"></a>
#### Варианты макета страницы аутентификации Livewire

Страницы аутентификации, входящие в стартовый комплект Livewire, такие как страница входа и страница регистрации, также предлагают три различных варианта макета: "simple", "card" и "split".

Чтобы изменить макет аутентификации, измените макет, используемый файлом `resources/views/layouts/auth.blade.php` вашего приложения:

```blade
<x-layouts::auth.split>
    {{ $slot }}
</x-layouts::auth.split>
```

<a name="authentication"></a>
## Аутентификация

Все стартовые комплекты используют [Laravel Fortify](/docs/{{version}}/fortify) для обработки аутентификации. Fortify предоставляет маршруты, контроллеры и логику входа, регистрации, сброса пароля, подтверждения email и других возможностей.

Fortify автоматически регистрирует маршруты аутентификации на основе возможностей, включенных в файле конфигурации `config/fortify.php` вашего приложения:

<div class="overflow-auto">

| Маршрут                  | Метод | Описание                           |
| ------------------------ | ----- | ---------------------------------- |
| `/login`                 | `GET` | Отображение формы входа            |
| `/login`                 | `POST` | Аутентификация пользователя        |
| `/logout`                | `POST` | Выход пользователя из системы      |
| `/register`              | `GET` | Отображение формы регистрации      |
| `/register`              | `POST` | Создание нового пользователя       |
| `/forgot-password`       | `GET` | Отображение формы запроса ссылки для сброса пароля |
| `/forgot-password`       | `POST` | Отправка ссылки для сброса пароля по email |
| `/reset-password/{token}` | `GET` | Отображение формы сброса пароля    |
| `/reset-password`        | `POST` | Сброс пароля                       |
| `/user/confirm-password` | `GET` | Отображение формы подтверждения пароля |
| `/user/confirm-password` | `POST` | Подтверждение пароля               |
| `/two-factor-challenge`  | `GET` | Отображение формы двухфакторной аутентификации |
| `/two-factor-challenge`  | `POST` | Проверка кода двухфакторной аутентификации |

</div>

Для просмотра всех маршрутов приложения можно использовать Artisan-команду `php artisan route:list`.

<a name="enabling-and-disabling-features"></a>
### Включение и отключение возможностей

Вы можете управлять включенными возможностями Fortify в файле `config/fortify.php`:

```php
use Laravel\Fortify\Features;

'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

Чтобы отключить возможность, закомментируйте или удалите соответствующую запись из массива `features`. Например, удалите `Features::registration()`, чтобы отключить публичную регистрацию.

При использовании стартовых комплектов [React](#react), [Svelte](#svelte) или [Vue](#vue) также удалите ссылки на маршруты отключенной возможности во фронтенд-коде. Эти комплекты используют Wayfinder для типобезопасной маршрутизации, и ссылки на несуществующие маршруты приведут к ошибке сборки.

<a name="customizing-actions"></a>
### Настройка создания пользователей и сброса пароля

Когда пользователь регистрируется или сбрасывает пароль, Fortify вызывает классы действий из каталога `app/Actions/Fortify` вашего приложения:

<div class="overflow-auto">

| Файл                          | Описание                              |
| ----------------------------- | ------------------------------------- |
| `CreateNewUser.php`           | Проверяет и создает новых пользователей |
| `ResetUserPassword.php`       | Проверяет и обновляет пароли пользователей |
| `PasswordValidationRules.php` | Определяет правила проверки пароля    |

</div>

Например, чтобы настроить логику регистрации вашего приложения, отредактируйте action `CreateNewUser`:

```php
public function create(array $input): User
{
    Validator::make($input, [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'email', 'max:255', 'unique:users'],
        'phone' => ['required', 'string', 'max:20'], // [tl! add]
        'password' => $this->passwordRules(),
    ])->validate();

    return User::create([
        'name' => $input['name'],
        'email' => $input['email'],
        'phone' => $input['phone'], // [tl! add]
        'password' => Hash::make($input['password']),
    ]);
}
```

<a name="two-factor-authentication"></a>
### Двухфакторная аутентификация

Стартовые комплекты включают встроенную двухфакторную аутентификацию (2FA), позволяющую пользователям защищать учетные записи с помощью любого TOTP-совместимого приложения-аутентификатора. 2FA включается через `Features::twoFactorAuthentication()` в файле `config/fortify.php`.

<a name="rate-limiting"></a>
### Ограничение частоты запросов

Ограничение частоты запросов помогает защитить эндпоинты аутентификации от перебора и повторяющихся попыток входа. Настроить его можно в `FortifyServiceProvider`:

```php
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Cache\RateLimiting\Limit;

RateLimiter::for('login', function ($request) {
    return Limit::perMinute(5)->by($request->email.$request->ip());
});
```

<a name="teams"></a>
## Команды

Стартовые комплекты React, Svelte, Vue и Livewire также могут быть сгенерированы с поддержкой команд. Когда функция команд включена, каждый пользователь принадлежит одной или нескольким командам и имеет текущую команду. При регистрации новые пользователи автоматически получают личную команду. Стартовые комплекты также включают экраны управления командами для создания команд, переключения между командами, приглашения участников и обновления сведений о команде.

Когда маршрут ограничен текущей командой, slug текущей команды включается в URL. Например, маршрут панели управления становится `/{current_team}/dashboard`, а страницы управления командами используют маршруты вроде `settings/teams/{team}`. При использовании параметров маршрута `{current_team}` и `{team}` стартовые комплекты автоматически гарантируют, что аутентифицированный пользователь принадлежит запрошенной команде, прежде чем разрешить доступ к маршруту.

Чтобы упростить генерацию URL, учитывающих команды, стартовые комплекты регистрируют значения URL по умолчанию для текущей команды аутентифицированного пользователя. Это позволяет вызовам помощников, таких как `route('dashboard')`, автоматически включать slug текущей команды. Когда пользователь входит в систему, регистрируется или переключает команду, стартовые комплекты обновляют текущую команду и эти значения URL по умолчанию, чтобы сгенерированные ссылки продолжали использовать правильный контекст команды.

При создании или переименовании команды стартовые комплекты также не позволяют пользователям выбирать зарезервированные имена, которые могут привести к небезопасным или конфликтующим сегментам маршрутов. Например, нельзя использовать имена, конфликтующие с префиксами маршрутов вроде `settings`, `login` или `dashboard`.

<a name="workos"></a>
## Аутентификация WorkOS AuthKit

По умолчанию стартовые наборы React, Svelte, Vue и Livewire используют встроенную систему аутентификации Laravel для входа, регистрации, сброса пароля, проверки электронной почты и т. д. Кроме того, мы также предлагаем вариант каждого стартового набора на основе [WorkOS AuthKit](https://authkit.com), который предлагает:

<div class="content-list" markdown="1">

- Социальная аутентификация (Google, Microsoft, GitHub и Apple)
- Аутентификация с помощью пароля
- "Magic Auth" на основе электронной почты
- SSO

</div>

Использование WorkOS в качестве поставщика аутентификации [требуется учетная запись WorkOS](https://workos.com). WorkOS предлагает бесплатную аутентификацию для приложений с ежемесячным количеством активных пользователей до 1 миллиона.

Чтобы использовать WorkOS AuthKit в качестве поставщика аутентификации вашего приложения, выберите опцию WorkOS при создании нового приложения на базе стартового набора с помощью `laravel new`.

### Настройка стартового комплекта WorkOS

После создания нового приложения с использованием стартового набора WorkOS, вам следует задать переменные окружения `WORKOS_CLIENT_ID`, `WORKOS_API_KEY` и `WORKOS_REDIRECT_URL` в файле `.env` вашего приложения. Эти переменные должны соответствовать значениям, предоставленным вам на панели инструментов WorkOS для вашего приложения:

```ini
WORKOS_CLIENT_ID=your-client-id
WORKOS_API_KEY=your-api-key
WORKOS_REDIRECT_URL="${APP_URL}/authenticate"
```

Кроме того, вам следует настроить URL домашней страницы приложения в панели управления WorkOS. Этот URL-адрес — это то место, куда будут перенаправляться пользователи после выхода из вашего приложения.

<a name="configuring-authkit-authentication-methods"></a>
#### Настройка методов аутентификации AuthKit

При использовании стартового комплекта на базе WorkOS мы рекомендуем отключить аутентификацию "Email + Password" в настройках конфигурации WorkOS AuthKit вашего приложения, что позволит пользователям проходить аутентификацию только через поставщиков социальных аутентификаций, пароли, "Magic Auth" и SSO. Это позволит вашему приложению полностью избежать обработки паролей пользователей.

<a name="configuring-authkit-session-timeouts"></a>
#### Настройка тайм-аутов сеанса AuthKit

Кроме того, мы рекомендуем вам настроить время бездействия сеанса WorkOS AuthKit в соответствии с настроенным пороговым значением времени бездействия сеанса вашего приложения Laravel, которое обычно составляет два часа.

<a name="inertia-ssr"></a>
### Inertia SSR

Стартовые наборы React, Svelte и Vue совместимы с возможностями [серверного рендеринга](https://inertiajs.com/server-side-rendering) Inertia. Чтобы создать совместимый с Inertia SSR-пакет для вашего приложения, выполните команду `build:ssr`:

```shell
npm run build:ssr
```

Для удобства также доступна команда `composer dev:ssr`. Эта команда запустит сервер разработки Laravel и сервер Inertia SSR после сборки SSR-совместимого пакета для вашего приложения, что позволит вам локально протестировать ваше приложение с помощью серверного движка рендеринга Inertia:

```shell
composer dev:ssr
```

<a name="community-maintained-starter-kits"></a>
### Стартовые наборы, поддерживаемые сообществом

При создании нового приложения Laravel с помощью установщика Laravel вы можете указать любой поддерживаемый сообществом стартовый набор, доступный на Packagist, в флаге `--using`:

```shell
laravel new my-app --using=example/starter-kit
```

<a name="creating-starter-kits"></a>
#### Создание стартовых наборов

Чтобы ваш стартовый набор был доступен другим, вам нужно опубликовать его на [Packagist](https://packagist.org). Ваш стартовый набор должен определять необходимые переменные среды в файле `.env.example`, а все необходимые команды после установки должны быть перечислены в массиве `post-create-project-cmd` файла `composer.json` стартового набора.

<a name="faqs"></a>
### Часто задаваемые вопросы

<a name="faq-upgrade"></a>
#### Как мне обновиться?

Каждый стартовый набор дает вам надежную отправную точку для вашего следующего приложения. Имея полное право собственности на код, вы можете настраивать и компоновать свое приложение именно так, как вы себе представляете. Однако нет необходимости обновлять сам стартовый набор.

<a name="faq-enable-email-verification"></a>
#### Как включить проверку электронной почты?

Проверку электронной почты можно добавить, раскомментировав импорт `MustVerifyEmail` в модели `App/Models/User.php` и убедившись, что модель реализует интерфейс `MustVerifyEmail`:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
// ...

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

После регистрации пользователи получат письмо с подтверждением. Чтобы ограничить доступ к определенным маршрутам до тех пор, пока адрес электронной почты пользователя не будет проверен, добавьте промежуточное ПО `verified` к маршрутам:

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('dashboard', function () {
        return Inertia::render('dashboard');
    })->name('dashboard');
});
```

> [!NOTE]
> При использовании варианта стартовых наборов [WorkOS](#workos) подтверждение адреса электронной почты не требуется.

<a name="faq-modify-email-template"></a>
#### Как изменить шаблон электронной почты по умолчанию?

Вы можете настроить шаблон электронной почты по умолчанию, чтобы он лучше соответствовал брендингу вашего приложения. Чтобы изменить этот шаблон, вам следует опубликовать представления электронной почты в вашем приложении с помощью следующей команды:

```
php artisan vendor:publish --tag=laravel-mail
```

Это сгенерирует несколько файлов в `resources/views/vendor/mail`. Вы можете изменить любой из этих файлов, а также файл `resources/views/vendor/mail/themes/default.css`, чтобы изменить внешний вид и внешний вид шаблона электронной почты по умолчанию.
