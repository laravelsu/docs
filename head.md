---
git: 3ebbcbf54a8ed7dcb72610d04e6780737ccb462b
---

# Laravel Head

- [Введение](#introduction)
- [Установка](#installation)
- [Быстрый старт](#quickstart)
- [Приоритет разрешения](#resolution-precedence)
- [Определение метаданных](#defining-metadata)
    - [Значения по умолчанию](#defaults)
    - [Метаданные маршрутов](#route-metadata)
    - [Метаданные во время выполнения](#runtime-metadata)
    - [Страницы ошибок](#error-pages)
- [Open Graph](#open-graph)
    - [Карточки X / Twitter](#twitter-cards)
- [Цвета темы](#theme-colors)
- [Метаданные приложения и иконки](#app-metadata-and-icons)
- [Progressive Web Apps](#progressive-web-apps)
- [Производительность и обнаружение](#performance-and-discovery)
- [Пользовательские теги](#custom-tags)
- [Схемы](#schemas)
    - [Breadcrumbs](#breadcrumbs)
    - [FAQs](#faqs)
    - [Пользовательские схемы](#custom-schemas)
- [Рендеринг](#rendering)
    - [Blade](#blade)
    - [Livewire](#livewire)
    - [Inertia](#inertia)

<a name="introduction"></a>
## Введение

[Laravel Head](https://github.com/laravel/head) предоставляет fluent API для управления элементом `<head>` документа вашего приложения, включая title и meta-теги, метаданные Open Graph, canonical URLs, директивы robots, performance hints и structured data. Он работает с Blade, Livewire и Inertia.

<a name="installation"></a>
## Установка

Вы можете установить Laravel Head с помощью менеджера пакетов Composer:

```shell
composer require laravel/head
```

<a name="quickstart"></a>
## Быстрый старт

Зарегистрируйте общие для сайта значения по умолчанию в сервис-провайдере:

```php
use Laravel\Head\Facades\Head;
use Laravel\Head\HeadBuilder;

Head::defaults(fn (HeadBuilder $head) => $head
    ->title('Laravel', suffix: ' - Laravel')
    ->description('Build something great.'));
```

Задайте метаданные конкретной страницы во время выполнения:

```php
Head::title($post->title)
    ->description($post->description);
```

Отрендерите разрешенные теги в layout:

```blade
<head>
    @head
</head>
```

<a name="resolution-precedence"></a>
## Приоритет разрешения

Метаданные страницы разрешаются из пяти слоев, перечисленных от самого низкого к самому высокому приоритету:

1. Значения страницы по умолчанию
2. Метаданные группы маршрутов
3. Метаданные маршрута
4. Метаданные во время выполнения
5. Метаданные ошибок

Более высокие слои заменяют более низкие отдельно для каждого поля. Например, заголовок, заданный во время выполнения, заменит заголовок маршрута, но не заменит описание маршрута. Следующие разделы описывают, как задавать метаданные на каждом слое. Подробнее о рендеринге разрешенных метаданных в Blade, Livewire и Inertia смотрите в разделе [Рендеринг](#rendering).

<a name="defining-metadata"></a>
## Определение метаданных

Laravel Head позволяет определять метаданные с помощью общих значений по умолчанию, метаданных маршрутов, вызовов во время выполнения и определений страниц ошибок.

<a name="defaults"></a>
### Значения по умолчанию

Зарегистрируйте значения страницы по умолчанию в сервис-провайдере:

```php
use Laravel\Head\Enums\OgType;
use Laravel\Head\Facades\Head;
use Laravel\Head\HeadBuilder;

Head::defaults(function (HeadBuilder $head) {
    $head
        ->title('Laravel', suffix: ' - Laravel')
        ->description('Build something great.')
        ->canonical()
        ->og(siteName: 'Laravel', type: OgType::Website)
        ->searchableByRobots()
        ->preconnect('https://fonts.example.com');
});
```

Значения по умолчанию являются слоем метаданных страницы с самым низким приоритетом. Если ни маршрут, ни runtime-код, ни метаданные ошибки не задают title, `Laravel` будет отрендерен как есть. Когда более высокий слой задает заголовок страницы, к нему применяется унаследованный suffix, поэтому `Head::title('About')` отрендерит `About - Laravel`. Передайте `exact: true` для заголовков, которые должны игнорировать унаследованные prefix или suffix.

Вызов `Head::canonical()` рендерит canonical URL на основе URL текущего запроса. Чтобы задать явный URL, передайте строку, например `Head::canonical('/about')`. Canonical URLs по умолчанию нормализуются к `https`; передайте `forceHttps: false`, чтобы сохранить scheme запроса.

Директивы robots можно передавать как строку, как cases enum `RobotsRule` или как список, смешивающий обе формы. Списки рендерятся как директивы, разделенные запятыми, поэтому `Head::robots([RobotsRule::NoIndex, RobotsRule::NoFollow])` отрендерит `noindex, nofollow`.

Для удобства метод `searchableByRobots` рендерит `all`, а метод `hiddenFromRobots` рендерит `none`.

<a name="route-metadata"></a>
### Метаданные маршрутов

Вы можете определять метаданные прямо на маршрутах. Это особенно полезно для полустатических страниц, метаданные которых известны заранее.

<a name="routes-and-groups"></a>
#### Маршруты и группы

```php
Route::view('/contact', 'contact')
    ->name('contact')
    ->withHead(
        title: 'Contact Us',
        description: 'Get in touch.',
    );
```

Общие метаданные маршрутов можно применить к группе в любой позиции цепочки:

```php
Route::withHead(robots: 'noindex, nofollow')
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', DashboardController::class)
            ->name('dashboard')
            ->withHead(title: 'Dashboard');
    });
```

Также можно определить метаданные для resource и singleton маршрутов:

```php
Route::resource('posts', PostController::class)->withHead(
    robots: 'index, follow',
);

Route::singleton('profile', ProfileController::class)->withHead(
    title: 'Your Profile',
);
```

Метод `withHead` сохраняет обычные массивы через нативный Laravel API метаданных маршрутов. Это эквивалентно вызову метода `metadata` с атрибутами, вложенными под ключ `head`, поэтому метаданные остаются совместимыми с кэшированными маршрутами.

Именованные аргументы намеренно ограничены встроенными свойствами маршрутов Laravel Head, чтобы редакторы и статический анализ могли находить опечатки в именах. Атрибуты маршрута, зарегистрированные пользовательскими построителями тегов, можно передать через `extensions`:

```php
Route::get('/article', ArticleController::class)->withHead(
    title: 'Article',
    extensions: ['readingTime' => 4],
);
```

<a name="supported-properties"></a>
#### Поддерживаемые свойства

Поддерживаемые свойства маршрутов соответствуют тем же именам, что и методы fluent builder:

| Категория | Свойства |
| --- | --- |
| Документ | `title`, `description`, `canonical`, `robots` |
| Метаданные приложения | `themeColor`, `applicationName`, `colorScheme`, `referrer`, `viewport`, `appleWebAppTitle`, `webAppCapable`, `appleWebAppStatusBarStyle` |
| Social | `og`, `ogImage`, `ogVideo`, `ogAudio`, `twitter`, `twitterImage` |
| Производительность | `preload`, `prefetch`, `preconnect`, `dnsPrefetch` |
| Обнаружение | `alternates`, `feed`, `icon`, `favicon`, `appleTouchIcon`, `appleTouchStartupImage`, `maskIcon`, `manifest` |
| Структурированные данные | `schema` |
| Пользовательские теги | `meta`, `link` |

Имена вложенных опций используют тот же `camelCase`, что и fluent API, например `forceHttps`, `siteName` и `secureUrl`.

Повторяемые свойства, такие как `ogImage`, `preload`, `feed`, `schema`, `icon` и `appleTouchStartupImage`, принимают либо одно значение, либо список.

<a name="runtime-metadata"></a>
### Метаданные во время выполнения

Когда значение неизвестно до прихода запроса, например title просматриваемой записи, его можно задать во время выполнения:

```php
use Laravel\Head\Facades\Head;

public function __invoke(Post $post): Response
{
    Head::title($post->title);

    // ...
}
```

Runtime-вызовы через facade `Head` переопределяют метаданные маршрута для данных, зависящих от запроса. Controllers и actions являются самыми распространенными местами для таких вызовов:

```php
use App\Models\Post;
use Laravel\Head\Facades\Head;

public function show(Post $post)
{
    Head::title($post->title)
        ->description($post->description);

    return view('posts.show', ['post' => $post]);
}
```

Несколько runtime-вызовов объединяются в порядке выполнения. Для полей с одним значением, таких как title, description, canonical URL и директивы robots, более поздний вызов имеет приоритет. Повторяемые поля сохраняют несколько записей, но повторное добавление того же ключа обновляет более раннюю запись. Для метода `ogImage` ключом является URL:

```php
Head::ogImage('/images/cover.jpg', alt: 'Draft cover')
    ->ogImage('/images/gallery.jpg', alt: 'Gallery image')
    ->ogImage('/images/cover.jpg', alt: 'Final cover', width: 1200, height: 630);
```

```html
<meta property="og:image" content="/images/cover.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:image:alt" content="Final cover">
<meta property="og:image" content="/images/gallery.jpg">
<meta property="og:image:alt" content="Gallery image">
```

Open Graph media, унаследованные из defaults, выступают как fallback. Когда маршрут, runtime-код или метаданные ошибки определяют собственные media того же типа, default media заменяются, а не объединяются, поэтому `og:image` страницы имеет приоритет над общим изображением сайта.

Условные метаданные можно определять fluent-цепочкой с помощью методов `when` и `unless`:

```php
Head::title($post->title)
    ->when($post->isDraft(), fn ($head) => $head->hiddenFromRobots());
```

<a name="error-pages"></a>
### Страницы ошибок

Обычно метаданные ошибок следует регистрировать в методе `boot` класса `AppServiceProvider` вашего приложения:

```php
use Laravel\Head\ErrorPages;
use Laravel\Head\Facades\Head;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Head::errors(function (ErrorPages $errors) {
        $errors->defaults(robots: 'noindex, follow');

        $errors->status(
            404,
            title: 'Page Not Found',
            description: 'The page you are looking for could not be found.',
        );
    });
}
```

Методы `defaults` и `status` также принимают тот же callback fluent builder, который используется в `Head::defaults()`:

```php
use Laravel\Head\ErrorPages;
use Laravel\Head\Facades\Head;
use Laravel\Head\HeadBuilder;

Head::errors(function (ErrorPages $errors) {
    $errors->status(404, fn (HeadBuilder $head) => $head
        ->title('Page Not Found')
        ->description('The page you are looking for could not be found.'));
});
```

Когда response рендерится для зарегистрированного status ошибки, эти метаданные получают приоритет над всеми остальными слоями.

Laravel автоматически определяет status response при рендеринге error view или выполнении respond-phase hook, например метода Inertia `handleExceptionsUsing()`. Если вы рендерите error response внутри callback `$exceptions->render()`, вызовите `Head::status(404)` перед рендерингом, чтобы применить метаданные ошибки.

<a name="open-graph"></a>
## Open Graph

Свойства Open Graph можно задавать с помощью метода `og`. Повторяемые media можно добавлять с помощью top-level методов, которые принимают именованные аргументы напрямую:

```php
use Laravel\Head\Enums\ImageType;
use Laravel\Head\Enums\OgType;

Head::og(type: OgType::Article, title: $post->title)
    ->ogImage($post->hero_image_url)
    ->ogImage(
        $post->gallery_image_url,
        alt: $post->gallery_image_alt,
        width: 1200,
        height: 630,
        type: ImageType::Jpeg,
    );
```

Методы `ogImage`, `ogVideo` и `ogAudio` принимают URL первым аргументом, а также optional named arguments, такие как `alt`, `width`, `height`, `type` и `secureUrl`, где они поддерживаются спецификацией Open Graph.

MIME-типы изображений можно передавать как cases enum `ImageType` везде, где API принимает image `type`, например `ImageType::Svg`, `ImageType::Png`, `ImageType::Jpeg` и `ImageType::Webp`.

> [!NOTE]
> Document `title` и `description` автоматически заполняют отсутствующие значения `og:title` и `og:description`.

Для одного Open Graph image без других атрибутов можно передать именованный аргумент `image` в метод `og`:

```php
Head::og(
    type: OgType::Website,
    title: $page->title,
    description: $page->description,
    image: $page->og_image_url,
);
```

Вызовы `og(image: ...)` и `ogImage(...)` записывают данные в один и тот же базовый список изображений, поэтому можно использовать тот вариант, который выразительнее в конкретном месте вызова. Метод [`meta`](#custom-tags) можно использовать для пользовательских расширений Open Graph, например свойств product или article.

<a name="twitter-cards"></a>
### Карточки X / Twitter

Чтобы рендерить карточки X / Twitter из тех же title, description и image, которые используются Open Graph, зарегистрируйте `twitter()` в defaults:

```php
use Laravel\Head\Enums\TwitterCard;
use Laravel\Head\Facades\Head;
use Laravel\Head\HeadBuilder;

Head::defaults(fn (HeadBuilder $head) => $head->twitter(
    card: TwitterCard::SummaryWithLargeImage,
));
```

Затем задайте метаданные на уровне страницы:

```php
Head::title('Introducing Laravel Head')
    ->description('A fluent API for Laravel document head metadata.')
    ->ogImage('https://example.com/social.jpg', alt: 'Introducing Laravel Head');
```

Это отрендерит соответствующие Twitter tags:

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Introducing Laravel Head">
<meta name="twitter:description" content="A fluent API for Laravel document head metadata.">
<meta name="twitter:image" content="https://example.com/social.jpg">
<meta name="twitter:image:alt" content="Introducing Laravel Head">
```

Отдельные страницы можно настраивать явными Twitter values:

```php
Head::twitter(title: $post->social_title)
    ->twitterImage($post->social_image_url, alt: $post->title);
```

Метаданные маршрутов принимают `twitter` и `twitterImage`.

<a name="theme-colors"></a>
## Цвета темы

Цвета темы можно задавать глобально, для маршрута или во время выполнения:

```php
Head::themeColor('#0f172a');
```

Это рендерит тег `<meta name="theme-color">`. Для theme colors, зависящих от media, можно использовать enum `Media`:

```php
use Laravel\Head\Enums\Media;

Head::themeColor('#ffffff', media: Media::Light)
    ->themeColor('#111827', media: Media::Dark);
```

Enum `Media` также включает `Portrait` и `Landscape`. Аргумент `media` также принимает пользовательскую media query string.

Метаданные маршрута поддерживают один theme color через тот же ключ `camelCase`:

```php
Route::view('/dashboard', 'dashboard')->withHead(
    themeColor: '#0f172a',
);
```

<a name="app-metadata-and-icons"></a>
## Метаданные приложения и иконки

Laravel Head включает методы для распространенных метаданных браузера и приложения:

```php
use Laravel\Head\Enums\ImageType;
use Laravel\Head\Enums\Media;

Head::applicationName('Laravel')
    ->colorScheme('light dark')
    ->referrer('strict-origin-when-cross-origin')
    ->viewport('width=device-width, initial-scale=1')
    ->appleWebAppTitle('Laravel')
    ->webAppCapable()
    ->appleWebAppStatusBarStyle('black')
    ->favicon('/favicon.svg', type: ImageType::Svg)
    ->icon('/favicon-32x32.png', type: ImageType::Png, sizes: '32x32')
    ->appleTouchIcon('/apple-touch-icon.png', sizes: '180x180')
    ->appleTouchStartupImage('/launch.png', media: Media::Portrait)
    ->maskIcon('/safari-pinned-tab.svg', color: '#111827')
    ->manifest('/site.webmanifest');
```

Метод `favicon` является псевдонимом метода `icon` и принимает те же аргументы `type`, `sizes` и `media`.

Метаданные маршрута используют те же имена:

```php
use Laravel\Head\Enums\ImageType;
use Laravel\Head\Enums\Media;

Route::view('/dashboard', 'dashboard')->withHead(
    applicationName: 'Laravel',
    colorScheme: 'light dark',
    appleWebAppTitle: 'Laravel',
    webAppCapable: true,
    appleWebAppStatusBarStyle: 'black',
    favicon: [
        ['href' => '/favicon.svg', 'type' => ImageType::Svg],
        ['href' => '/favicon-32x32.png', 'type' => ImageType::Png, 'sizes' => '32x32'],
    ],
    appleTouchIcon: ['href' => '/apple-touch-icon.png', 'sizes' => '180x180'],
    appleTouchStartupImage: ['href' => '/launch.png', 'media' => Media::Portrait],
    manifest: '/site.webmanifest',
);
```

<a name="progressive-web-apps"></a>
## Progressive Web Apps

Метод `pwa` настраивает распространенные теги `<head>` документа, необходимые для installable web app:

```php
Head::pwa(
    name: 'Laravel',
    manifest: '/site.webmanifest',
    themeColor: '#0f172a',
    appleTouchIcon: '/apple-touch-icon.png',
    appleWebAppStatusBarStyle: 'black',
);
```

Это рендерит имя приложения, ссылку на web application manifest и iOS standalone metadata. Если переданы theme color, Apple status bar style и Apple touch icon, они также будут отрендерены. Создание web application manifest и регистрация service worker остаются ответственностью вашего приложения.

Метод `pwa` можно использовать в defaults или runtime metadata. Метаданные маршрутов поддерживают отдельные свойства, показанные выше.

<a name="performance-and-discovery"></a>
## Производительность и обнаружение

Laravel Head рендерит performance hints, pagination links, locale alternates и feed discovery:

```php
Head::preload(asset('fonts/inter.woff2'), as: 'font', crossorigin: true)
    ->prefetch(asset('images/next.webp'))
    ->preconnect('https://cdn.example.com')
    ->dnsPrefetch('https://analytics.example.com')
    ->paginate($posts)
    ->alternates([
        'en' => 'https://example.com/en/about',
        'fr' => 'https://example.com/fr/about',
        'x-default' => 'https://example.com/about',
    ])
    ->feed('/feed', title: 'Laravel RSS')
    ->feed('/feed.atom', type: 'atom', title: 'Laravel Atom');
```

Для локальных assets методы `preloadAsset()` и `prefetchAsset()` разрешают URL через helper `asset()` и определяют атрибут `as` по расширению файла. Preload для fonts автоматически включает `crossorigin`, что требует спецификация preload даже для same-origin fonts:

```php
Head::preloadAsset('fonts/inter.woff2')
    ->prefetchAsset('images/next.webp');
```

```html
<link rel="preload" href="https://example.com/fonts/inter.woff2" as="font" crossorigin>
<link rel="prefetch" href="https://example.com/images/next.webp" as="image">
```

Можно явно передать `as`, чтобы переопределить automatic detection. Метод `preloadAsset` выбросит исключение, если атрибут `as` невозможно определить по расширению, потому что браузеры игнорируют preloads без этого атрибута; метод `prefetchAsset` в таком случае просто опустит его.

<a name="custom-tags"></a>
## Пользовательские теги

Для тегов без отдельного метода используйте `meta()` и `link()`:

```php
Head::meta('format-detection', 'telephone=no')
    ->meta('article:author', $post->author->name)
    ->link('search', '/opensearch.xml', [
        'type' => 'application/opensearchdescription+xml',
        'title' => 'Laravel Search',
    ])
    ->link('me', 'https://social.example.com/@laravel');
```

В meta tag можно включить media query, когда браузер должен применять тег только при совпадающих условиях:

```php
use Laravel\Head\Enums\Media;

Head::meta('theme-color', '#ffffff', media: Media::Light)
    ->meta('theme-color', '#111827', media: Media::Dark);
```

Метод `meta` использует атрибут `name` для обычных meta tags. Для ключей, которые обычно используют атрибут `property`, например Open Graph (`og:`) или article metadata (`article:`), метод переключается автоматически:

```php
Head::meta('description', 'About Laravel')
    ->meta('og:title', 'About Laravel');
```

```html
<meta name="description" content="About Laravel">
<meta property="og:title" content="About Laravel">
```

Можно передать `property: true` или `property: false`, чтобы явно выбрать один из атрибутов.

<a name="schemas"></a>
## Схемы

Встроенные schema builders покрывают распространенные типы JSON-LD:

```php
use Laravel\Head\Enums\OfferAvailability;
use Laravel\Head\Facades\Schema;

Head::schema(
    Schema::product()
        ->name($product->name)
        ->offers(
            Schema::offer()
                ->price($product->price)
                ->currency('USD')
                ->availability(OfferAvailability::InStock)
        )
);
```

Встроенные factory methods: `article`, `blogPosting`, `product`, `offer`, `brand`, `breadcrumbs`, `faq`, `organization`, `person`, `webPage` и `webSite`. Неизвестные factory methods создают универсальный schema object, поэтому вы можете описывать пользовательские типы schema.org.

Когда данные JSON-LD schema невалидны, Laravel Head выбрасывает исключение в non-production окружениях и пишет warning в production.

<a name="breadcrumbs"></a>
### Breadcrumbs

Breadcrumb items можно добавлять по одному или массово. Positions назначаются автоматически в порядке добавления элементов:

```php
Head::schema(
    Schema::breadcrumbs()->items([
        'Home' => route('home'),
        'Shop' => route('shop.index'),
        'Shoes' => route('shop.category', 'shoes'),
    ])
);
```

Метод `item` можно использовать, чтобы добавить один breadcrumb item:

```php
Schema::breadcrumbs()
    ->item('Home', route('home'))
    ->item('Shop', route('shop.index'));
```

<a name="faqs"></a>
### FAQs

FAQ entries следуют той же схеме. Их можно добавлять по одному с помощью метода `question` или массово с помощью метода `questions`:

```php
Head::schema(
    Schema::faq()->questions([
        'What is Laravel Head?' => 'A fluent API for managing the document head.',
        'Is it free?' => 'Yes, it is open source.',
    ])
);
```

<a name="custom-schemas"></a>
### Пользовательские схемы

Пользовательские schema types можно регистрировать явно:

```php
use DateTimeInterface;
use Laravel\Head\Facades\Schema;
use Laravel\Head\Schema\SchemaObject;
use Laravel\Head\SchemaType;

#[SchemaType('JobPosting')]
class JobPosting extends SchemaObject
{
    public function title(string $title): static
    {
        return $this->set('title', $title);
    }

    public function datePosted(DateTimeInterface|string $date): static
    {
        return $this->date('datePosted', $date);
    }
}

Schema::register(JobPosting::class);

Head::schema(
    Schema::jobPosting()
        ->title('Senior Laravel Developer')
        ->datePosted(now())
);
```

<a name="rendering"></a>
## Рендеринг

Laravel Head разрешает метаданные страницы в теги для текущего response. То, как эти теги рендерятся, зависит от стека вашего приложения.

HTML renderer обеспечивает директиву `@head` и rendered elements, которые Laravel Head передает Inertia через prop `head`. Array renderer обеспечивает `Head::toArray()` для приложений, которым нужны разрешенные метаданные в виде structured data.

<a name="blade"></a>
### Blade

Отрендерите накопленные теги в `<head>` вашего layout с помощью директивы `@head`:

```blade
<head>
    <meta charset="utf-8">
    @head
</head>
```

Директива `@head` рендерится синхронно, поэтому метаданные страницы следует определить до рендеринга layout.

<a name="livewire"></a>
### Livewire

Приложения Livewire используют ту же директиву `@head` в document layout:

```blade
<head>
    @head
</head>

<body>
    {{ $slot }}

    @livewireScripts
</body>
```

Специальная настройка Livewire не требуется. Метаданные Laravel Head разрешаются для каждого запроса, а resolver ограничен областью запроса. Поэтому каждый переход `wire:navigate` получает свежий документ, вывод `@head` которого отражает метаданные целевого маршрута. Страницы, посещенные через `wire:navigate`, получают подходящие route, runtime и error metadata без необходимости писать head-код на уровне компонента.

<a name="inertia"></a>
### Inertia

Используйте ту же директиву `@head` в корневом шаблоне Inertia вместе с собственными компонентами Inertia:

```blade
<html>
<head>
    <meta charset="utf-8">
    @head

    @viteReactRefresh
    @vite(['resources/css/app.css', 'resources/js/app.tsx'])
    <x-inertia::head />
</head>
<body>
    <x-inertia::app />
</body>
</html>
```

Когда Inertia установлена, Laravel Head автоматически передает управляемый страницей head как массив rendered element strings под prop `head` в каждом page object:

```json
{
    "props": {
        "head": [
            "<title data-inertia=\"title\">Dashboard - Laravel</title>",
            "<meta data-inertia=\"description\" name=\"description\" content=\"Your application overview.\">"
        ]
    }
}
```

Включите опцию Inertia `serverHead` там, где приложение вызывает `createInertiaApp()`. Опция доступна в Inertia 3.5 и новее:

```js
createInertiaApp({
    // ...
    serverHead: true,
});
```

Каждый управляемый страницей элемент имеет стабильный ключ `data-inertia`. Директива `@head` рендерит initial document, после чего Inertia принимает эти элементы и синхронизирует их во время стандартных visits, [instant visits](https://inertiajs.com/docs/v3/the-basics/instant-visits), а также навигации назад и вперед. Эти элементы присутствуют в initial HTML response, поэтому crawlers и link-preview bots могут прочитать их без выполнения JavaScript. Клиентский компонент `<Head>` не требуется.

Это работает как с [server-side rendering (SSR)](https://inertiajs.com/docs/v3/advanced/server-side-rendering), так и без него. Если у приложения есть отдельный SSR entry point, включите `serverHead` и там. Laravel Head автоматически дедуплицирует управляемые страницей элементы между `@head` и `<x-inertia::head />` независимо от их порядка, сохраняя другие head elements, созданные JavaScript SSR.

> [!NOTE]
> При добавлении Laravel Head в существующее Inertia-приложение удалите title callbacks из `resources/js/app.tsx` и `resources/js/ssr.tsx`, чтобы Laravel Head мог управлять итоговым document title, и перенесите теги, которыми управляет Inertia-компонент [`<Head>`](https://inertiajs.com/docs/v3/the-basics/title-and-meta), в Laravel Head, чтобы они никогда не определяли один и тот же element.

Prop `head` отсутствует в responses partial reload, поэтому Inertia сохраняет head последней полной страницы. Instant visits аналогично сохраняют текущий head до получения background response. Если приложение уже использует prop `head`, измените его имя в сервис-провайдере:

```php
use Laravel\Head\Facades\Head;

public function boot(): void
{
    Head::inertia(prop: '_head');
}
```

Затем укажите Inertia тот же prop через `serverHead: '_head'`.

<a name="static-inertia-tags"></a>
#### Статические теги Inertia

Большинство тегов должны находиться в defaults, route metadata или runtime metadata, чтобы Laravel Head мог разрешать правильное значение для каждой страницы. Используйте Inertia globals только для document tags, которые рендерятся в первом HTML response и остаются неизменными в Inertia до конца session.

Зарегистрируйте их в сервис-провайдере с помощью `Head::inertiaGlobals()`:

```php
use Laravel\Head\Facades\Head;
use Laravel\Head\HeadBuilder;

Head::inertiaGlobals(function (HeadBuilder $head) {
    $head
        ->viewport('width=device-width, initial-scale=1')
        ->colorScheme('light dark')
        ->icon('/favicon.svg', type: 'image/svg+xml')
        ->appleTouchIcon('/apple-touch-icon.png', sizes: '180x180')
        ->manifest('/site.webmanifest');
});
```

Inertia globals исключаются из prop `head`, рендерятся без ownership-атрибутов `data-inertia` и никогда не обновляются после первого response. Эти globals подходят для стабильных browser hints, таких как viewport, color scheme, favicons, touch icons и manifests. Если тег относится к конкретной странице, важен для SEO или может быть переопределен позже, поместите его в `defaults`, route metadata или runtime metadata.

Приложения, которым нужны разрешенные метаданные в виде structured data вместо rendered tags, могут вызвать `Head::toArray()`. Возвращаемые данные включают titles, значения Open Graph, JSON-LD schemas и другие разрешенные метаданные.
