---
git: 4b842431d1c68b4bd92200432d7c42c6bab44175
---

# Стартовые комплекты

<a name="introduction"></a>
## Введение

Чтобы помочь вам быстрее начать работу с новым приложением Laravel, мы рады предложить [стартовые наборы приложений](https://laravel.com/starter-kits). Эти стартовые наборы дают вам фору при создании вашего следующего приложения Laravel и включают маршруты, контроллеры и представления, необходимые для регистрации и аутентификации пользователей вашего приложения.

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

<a name="vue"></a>
### Vue

Наш стартовый комплект Vue представляет собой прекрасную отправную точку для создания приложений Laravel с интерфейсом Vue, использующим [Inertia](https://inertiajs.com).

Inertia позволяет вам создавать современные одностраничные приложения Vue, используя классическую маршрутизацию и контроллеры на стороне сервера. Это позволяет вам наслаждаться мощью фронтенда Vue в сочетании с невероятной производительностью бэкенда Laravel и молниеносной компиляцией Vite.

Стартовый комплект Vue использует API Vue Composition, TypeScript, Tailwind и библиотеку компонентов [shadcn-vue](https://www.shadcn-vue.com/).

<a name="livewire"></a>
### Livewire

Наш стартовый комплект Livewire представляет собой идеальную отправную точку для создания приложений Laravel с помощью интерфейса [Laravel Livewire](https://livewire.laravel.com).

Livewire — это мощный способ создания динамических, реактивных интерфейсов frontend с использованием только PHP. Он отлично подходит для команд, которые в основном используют шаблоны Blade и ищут более простую альтернативу фреймворкам SPA на основе JavaScript, таким как React и Vue.

Стартовый комплект Livewire использует Livewire, Tailwind и библиотеку компонентов [Flux UI](https://fluxui.dev).

<a name="starter-kit-customization"></a>
## Настройка стартового набора

<a name="react-customization"></a>
### React

Наш стартовый набор React создан с использованием Inertia 2, React 19, Tailwind 4 и [shadcn/ui](https://ui.shadcn.com). Как и во всех наших стартовых наборах, весь код бэкэнда и фронтэнда находится в вашем приложении, что позволяет выполнять полную настройку.

Большая часть кода frontend находится в каталоге `resources/js`. Вы можете свободно изменять любой код, чтобы настроить внешний вид и поведение вашего приложения:

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

<a name="vue-customization"></a>
### Vue

Наш стартовый комплект Vue создан с использованием Inertia 2, Vue 3 Composition API, Tailwind и [shadcn-vue](https://www.shadcn-vue.com/). Как и во всех наших стартовых комплектах, весь код бэкэнда и фронтэнда находится в вашем приложении, что позволяет выполнять полную настройку.

Большая часть кода frontend находится в каталоге `resources/js`. Вы можете свободно изменять любой код, чтобы настроить внешний вид и поведение вашего приложения:

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
import { Switch } from '@/Components/ui/switch'
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

Наш стартовый комплект Livewire создан с использованием Livewire 3, Tailwind и [Flux UI](https://fluxui.dev/). Как и во всех наших стартовых комплектах, весь код бэкэнда и фронтэнда находится в вашем приложении, что позволяет выполнять полную настройку.

#### Livewire и Volt

Большая часть кода frontend находится в каталоге `resources/views`. Вы можете свободно изменять любой код, чтобы настроить внешний вид и поведение вашего приложения:

```text
resources/views
├── components            # Повторно используемые компоненты Livewire
├── flux                  # Индивидуальные компоненты Flux
├── livewire              # Страницы Livewire
├── partials              # Часто используемые представления Blade
├── dashboard.blade.php   # Панель управления аутентифицированного пользователя
├── welcome.blade.php     # Приветственная страница для гостей
```

#### Традиционные компоненты Livewire

Код интерфейса находится в каталоге `resources/views`, а каталог `app/Livewire` содержит соответствующую внутреннюю логику для компонентов Livewire.

<a name="livewire-available-layouts"></a>
#### Доступные макеты Livewire

Стартовый комплект Livewire включает в себя два различных основных макета на выбор: макет "sidebar" и макет "header". Макет "sidebar" используется по умолчанию, но вы можете переключиться на макет "header", изменив макет, используемый файлом `resources/views/components/layouts/app.blade.php` вашего приложения. Кроме того, вам следует добавить атрибут `container` к основному компоненту Flux:

```blade
<x-layouts.app.header>
    <flux:main container>
        {{ $slot }}
    </flux:main>
</x-layouts.app.header>
```

<a name="livewire-authentication-page-layout-variants"></a>
#### Варианты макета страницы аутентификации Livewire

Страницы аутентификации, входящие в стартовый комплект Livewire, такие как страница входа и страница регистрации, также предлагают три различных варианта макета: "simple", "card" и "split".

Чтобы изменить макет аутентификации, измените макет, используемый файлом `resources/views/components/layouts/auth.blade.php` вашего приложения:

```blade
<x-layouts.auth.split>
    {{ $slot }}
</x-layouts.auth.split>
```

<a name="workos"></a>
## Аутентификация WorkOS AuthKit

По умолчанию стартовые наборы React, Vue и Livewire используют встроенную систему аутентификации Laravel для входа, регистрации, сброса пароля, проверки электронной почты и т. д. Кроме того, мы также предлагаем вариант каждого стартового набора на основе [WorkOS AuthKit](https://authkit.com), который предлагает:

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

Стартовые наборы React и Vue совместимы с возможностями [серверного рендеринга](https://inertiajs.com/server-side-rendering) Inertia. Чтобы создать совместимый с Inertia SSR-пакет для вашего приложения, выполните команду `build:ssr`:

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
