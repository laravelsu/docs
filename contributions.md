---
git: 89e91b5cff48e1b9b1a7921300653eb1ceb7bfcb
---

# Рекомендации по участию

<a name="bug-reports"></a>
## Отчеты об ошибках

Чтобы поощрять активное сотрудничество, Laravel настоятельно рекомендует отправлять Pull Request с исправлением проблемы, а не создавать issue на GitHub. В большинстве официальных пакетов Laravel раздел GitHub Issues отключён.

Если вы обнаружили проблему, создайте Pull Request с её исправлением. Он должен содержать заголовок и ясное описание проблемы и предложенного решения. Также добавьте как можно больше относящейся к делу информации и пример кода, демонстрирующий проблему. Цель Pull Request - помочь вам и другим участникам понять проблему и проверить исправление.

Если вы не знаете, как исправить проблему, опишите её агенту для работы с кодом и попробуйте с его помощью подготовить Pull Request.

Pull Request рассматриваются только после перевода в состояние «готов к проверке» (то есть не как черновик) и при успешном прохождении всех тестов нового функционала. Неактивные Pull Request, долго остающиеся черновиками, будут закрыты через несколько дней.

В управлении исходным кодом Laravel используется GitHub, и для каждого проекта есть репозитории:

<!-- <div class="content-list" markdown="1"> -->

- [Laravel AI SDK](https://github.com/laravel/ai)
- [Приложение Laravel](https://github.com/laravel/laravel)
- [Логотипы Laravel](https://github.com/laravel/art)
- [Пакет Laravel Boost](https://github.com/laravel/boost)
- [Документация Laravel](https://github.com/laravel/docs)
- [Пакет Laravel Dusk](https://github.com/laravel/dusk)
- [Пакет Laravel Cashier Stripe](https://github.com/laravel/cashier)
- [Пакет Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)
- [Пакет Laravel Echo](https://github.com/laravel/echo)
- [Пакет Laravel Envoy](https://github.com/laravel/envoy)
- [Пакет Laravel Folio](https://github.com/laravel/folio)
- [Фреймворк Laravel](https://github.com/laravel/framework)
- [Пакет Laravel Horizon](https://github.com/laravel/horizon)
- [Пакет Laravel Passport](https://github.com/laravel/passport)
- [Пакет Laravel Pennant](https://github.com/laravel/pennant)
- [Пакет Laravel Pint](https://github.com/laravel/pint)
- [Пакет Laravel Prompts](https://github.com/laravel/prompts)
- [Пакет Laravel Reverb](https://github.com/laravel/reverb)
- [Пакет Laravel Sail](https://github.com/laravel/sail)
- [Пакет Laravel Sanctum](https://github.com/laravel/sanctum)
- [Пакет Laravel Scout](https://github.com/laravel/scout)
- [Пакет Laravel Socialite](https://github.com/laravel/socialite)
- [Пакет Laravel Telescope](https://github.com/laravel/telescope)
- [Пакет Laravel Livewire Starter Kit](https://github.com/laravel/livewire-starter-kit)
- [Пакет Laravel React Starter Kit](https://github.com/laravel/react-starter-kit)
- [Пакет Laravel Svelte Starter Kit](https://github.com/laravel/svelte-starter-kit)
- [Пакет Laravel Vue Starter Kit](https://github.com/laravel/vue-starter-kit)

<!-- </div> -->

<a name="support-questions"></a>
## Вопросы поддержки

Трекеры с тикетами проблем Laravel на GitHub не предназначены для предоставления помощи или поддержки Laravel. Вместо этого используйте один из следующих каналов:

<!-- <div class="content-list" markdown="1"> -->

- [Обсуждения на GitHub](https://github.com/laravel/framework/discussions)
- [Форум Laracasts](https://laracasts.com/discuss)
- [Форум Laravel.io](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discord.gg/laravel)
- [Larachat](https://larachat.co)
- [IRC](https://web.libera.chat/?nick=artisan&channels=#laravel)

<!-- </div> -->

<a name="which-branch"></a>
## Какую ветку выбрать при запросах слияния?

**Все** исправления ошибок должны быть отправлены в последнюю версию, которая поддерживает исправления ошибок (на данный момент `13.x`). Исправления ошибок **никогда** не должны отправляться в ветку `master`, если они не исправляют функции, которые существуют только в предстоящем выпуске.

**Минорный** функционал, **полностью обратно совместимый** с текущим релизом, может быть отправлен в последнюю стабильную ветку (в настоящее время `13.x`).

**Мажорный** новый функционал или функционал с изменениями, приводящими к нарушению обратной совместимости, должен всегда отправляться в ветку `master`, содержащую предстоящий релиз.

<a name="compiled-assets"></a>
## Скомпилированные ресурсы исходников

Если вы отправляете изменение, которое повлияет на скомпилированные файлы, например, касательно файлов в `resources/css` или` resources/js` репозитория `laravel/laravel`, то не включайте в коммит эти скомпилированные файлы. Из-за большого размера они не могут быть реально рассмотрены сопровождающим. Это может быть использовано как способ внедрения вредоносного кода в Laravel. Чтобы предотвратить это, все скомпилированные файлы будут сгенерированы и включены в коммит сопровождающими Laravel.

<a name="ai-generated-contributions"></a>
## Вклады, сгенерированные ИИ

Мы ценим каждый Pull Request, отправленный в Laravel. Однако существенные изменения, преимущественно сгенерированные ИИ без вдумчивой человеческой проверки и осмысления, неприемлемы.

Если вы решили использовать инструменты ИИ для подготовки крупных или сложных изменений фреймворка, получившийся код **должен** быть вами тщательно проверен, протестирован и понят перед отправкой.

Описание Pull Request **должно** быть полностью написано участником. Pull Request с описаниями, сгенерированными ИИ, будут закрыты.

**Массовое открытие issues или Pull Request, полностью сгенерированных ИИ, не допускается.** Такие Pull Request будут закрыты без рассмотрения, а пользователь, отправивший их, может быть заблокирован в репозитории.

Мы рекомендуем участникам познакомиться с существующей кодовой базой, взаимодействовать с сообществом и отправлять Pull Request, которые отражают их собственное понимание и внимательное рассмотрение решаемой проблемы.

<a name="security-vulnerabilities"></a>
## Уязвимости безопасности

Если вы обнаружите уязвимость в системе безопасности Laravel, отправьте письмо нашей команде безопасности по адресу <a href="mailto:security@laravel.com">security@laravel.com</a>. Все уязвимости безопасности будут незамедлительно устранены.

<a name="coding-style"></a>
## Стиль кодирования

Laravel следует стандарту кодирования [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide) и стандарту автозагрузки [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader).

<a name="phpdoc"></a>
### PHPDoc

Ниже приведен пример валидного блока документации Laravel. Обратите внимание, что за атрибутом `@param` идут два пробела, тип аргумента, еще два пробела и, наконец, имя переменной:

```php
/**
 * Регистрация привязки к контейнеру.
 *
 * @param  string|array  $abstract
 * @param  \Closure|string|null  $concrete
 * @param  bool  $shared
 * @return void
 *
 * @throws \Exception
 */
public function bind($abstract, $concrete = null, $shared = false)
{
    // ...
}
```

Когда атрибуты `@param` или `@return` являются избыточными из-за использования нативных типов, их можно удалить:

```php
/**
 * Выполнение задания.
 * [tl! remove]
 * @return void [tl! remove]
 */
public function handle(AudioProcessor $processor): void
{
    // ...
}
```

Однако, когда нативный тип является обобщенным, укажите его через использование атрибутов `@param` или `@return`:

```php
/**
 * Получение вложения к сообщению.
 * [tl! add]
 * @return array<int, \Illuminate\Mail\Mailables\Attachment> [tl! add]
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

<a name="styleci"></a>
### StyleCI

Не волнуйтесь, если стиль вашего кода не идеален! [StyleCI](https://styleci.io/) автоматически объединит любые исправления стиля после слияния вашего запроса с репозиторием Laravel. Это позволяет нам сосредоточиться на содержании вашего вклада, а не на стиле кода.

<a name="code-of-conduct"></a>
## Нормы поведения

Кодекс поведения Laravel основан на кодексе поведения Ruby. О любых нарушениях кодекса поведения можно сообщить Тейлору Отвеллу (taylor@laravel.com):

<!-- <div class="content-list" markdown="1"> -->

- Участники будут терпимо относиться к противоположным взглядам.
- Участники должны гарантировать, что их язык и действия не содержат личных нападок и пренебрежительных личных замечаний.
- Толкуя слова и действия других, участники всегда должны исходить из добрых намерений.
- Не допускается поведение, которое можно обоснованно считать преследованием.

<!-- </div> -->
