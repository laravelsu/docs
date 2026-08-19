---
git: a656f28557de9c3b98926eebd87bc1e578b0436e
---

# Prompts (Подсказки)

<a name="introduction"></a>
## Введение

[Laravel Prompts](https://github.com/laravel/prompts) - это PHP-пакет, который позволяет добавлять красивые и удобные формы в ваши приложения командной строки, с функциями, подобными браузеру, включая плейсхолдеры и валидацию.

<img src="https://laravel.com/img/docs/prompts-example.png">

Laravel Prompts идеально подходит для приема пользовательского ввода в ваших [командах Artisan консоли](/docs/{{version}}/artisan#writing-commands), но его также можно использовать в любом проекте с командной строкой на PHP.

> [!NOTE]
> Laravel Prompts поддерживает macOS, Linux и Windows с WSL. Для получения дополнительной информации, пожалуйста, ознакомьтесь с нашей документацией по [не поддерживаемым средам и резервным вариантам](#fallbacks).

<a name="installation"></a>
## Установка

Laravel Prompts уже включен в последний релиз Laravel.

Вы также можете установить Laravel Prompts в другие проекты PHP, используя менеджер пакетов Composer:

```shell
composer require laravel/prompts
```

<a name="available-prompts"></a>
## Доступные Prompts

<a name="text"></a>
### Текст

Функция `text` предложит пользователю указанный вопрос, примет введённый текст и затем вернет его:

```php
use function Laravel\Prompts\text;

$name = text('Как вас зовут?');
```

Вы также можете добавить плейсхолдер, значение по умолчанию и информационную подсказку:

```php
$name = text(
    label: 'Как вас зовут?',
    placeholder: 'Например: Тейлор Отвелл',
    default: $user?->name,
    hint: 'Это будет отображаться в вашем профиле.'
);
```

<a name="text-required"></a>
#### Обязательные значения

Если вам необходимо, чтобы было введено значение, вы можете передать аргумент `required`:

```php
$name = text(
    label: 'Как вас зовут?',
    required: true
);
```

Если вы хотите настроить сообщение об ошибке валидации, вы также можете передать строку:

```php
$name = text(
    label: 'Как вас зовут?',
    required: 'Требуется ваше имя.'
);
```

<a name="text-validation"></a>
#### Дополнительная Валидация

Наконец, если вы хотите выполнить дополнительную логику валидации, вы можете передать замыкание в аргумент `validate`:

```php
$name = text(
    label: 'Как вас зовут?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'Имя должно состоять минимум из 3 символов.',
        strlen($value) > 255 => 'Имя не должно превышать 255 символов.',
        default => null
    }
);
```

Замыкание получит введенное значение и может вернуть сообщение об ошибке или `null`, если валидация прошла успешно.

Альтернативно вы можете использовать возможности [валидатора](/docs/{{version}}/validation) Laravel. Для этого предоставьте в аргумент `validate` массив, содержащий имя атрибута и желаемые правила проверки:

```php
$name = text(
    label: 'Как вас зовут?',
    validate: ['name' => 'required|max:255|unique:users']
);
```

<a name="textarea"></a>
### Textarea

Функция `textarea` предложит пользователю задать заданный вопрос, примет его ввод через многострочную текстовую область, а затем вернет его:

```php
use function Laravel\Prompts\textarea;

$story = textarea('Расскажи мне историю.');
```

Вы также можете включить текст-заполнитель, значение по умолчанию и информационную подсказку:

```php
$story = textarea(
    label: 'Расскажи мне историю.',
    placeholder: 'Это история о...',
    hint: 'Это будет отображаться в вашем профиле.'
);
```

<a name="textarea-required"></a>
#### Обязательные значения

Если вам требуется, чтобы значение было введено, то вы можете передать аргумент `required`:

```php
$story = textarea(
    label: 'Расскажи мне историю.',
    required: true
);
```

Если вы хотите настроить сообщение проверки, вы также можете передать строку:

```php
$story = textarea(
    label: 'Расскажи мне историю.',
    required: 'Требуется история.'
);
```

<a name="textarea-validation"></a>
#### Дополнительная проверка

Наконец, если вы хотите выполнить дополнительную логику проверки, вы можете передать замыкание аргументу `validate`:

```php
$story = textarea(
    label: 'Расскажи мне историю.',
    validate: fn (string $value) => match (true) {
        strlen($value) < 250 => 'В рассказе должно быть не менее 250 символов.',
        strlen($value) > 10000 => 'Рассказ не должен превышать 10 000 символов.',
        default => null
    }
);
```

Замыкание получит введенное значение и может вернуть сообщение об ошибке или значение `null`, если проверка пройдет успешно.

Альтернативно вы можете использовать возможности [валидатора](/docs/{{version}}/validation) Laravel. Для этого предоставьте в аргумент `validate` массив, содержащий имя атрибута и желаемые правила проверки:

```php
$story = textarea(
    label: 'Расскажи мне историю.',
    validate: ['story' => 'required|max:10000']
);
```

<a name="number"></a>
### Число

Функция `number` задает пользователю вопрос, принимает числовой ввод и возвращает его. Пользователь может изменять значение с помощью клавиш со стрелками вверх и вниз:

```php
use function Laravel\Prompts\number;

$number = number('Сколько копий вы хотите?');
```

Вы также можете указать placeholder, значение по умолчанию и информационную подсказку:

```php
$copies = number(
    label: 'Сколько копий вы хотите?',
    placeholder: '5',
    default: 1,
    hint: 'Это определит, сколько копий будет создано.'
);
```

<a name="number-required"></a>
#### Обязательные значения

Если значение обязательно, передайте аргумент `required`:

```php
$copies = number(
    label: 'Сколько копий вы хотите?',
    required: true
);
```

Если вы хотите настроить сообщение об ошибке валидации, вы также можете передать строку:

```php
$copies = number(
    label: 'Сколько копий вы хотите?',
    required: 'Количество копий обязательно.'
);
```

<a name="number-validation"></a>
#### Дополнительная валидация

Для дополнительной логики валидации передайте замыкание в аргумент `validate`:

```php
$copies = number(
    label: 'Сколько копий вы хотите?',
    validate: fn (?int $value) => match (true) {
        $value < 1 => 'Нужна хотя бы одна копия.',
        $value > 100 => 'Нельзя создать больше 100 копий.',
        default => null
    }
);
```

Также можно использовать [валидатор](/docs/{{version}}/validation) Laravel:

```php
$copies = number(
    label: 'Сколько копий вы хотите?',
    validate: ['copies' => 'required|integer|min:1|max:100']
);
```

<a name="password"></a>
### Пароль

Функция `password` аналогична функции `text`, но ввод пользователя будет маскироваться при вводе в консоли. Это полезно при запросе чувствительной информации, такой как пароли:

```php
use function Laravel\Prompts\password;

$password = password('Какой у вас пароль?');
```

Вы также можете включить плейсхолдер и информационную подсказку:

```php
$password = password(
    label: 'Какой у вас пароль?',
    placeholder: 'пароль',
    hint: 'Минимум 8 символов.'
);
```

<a name="password-required"></a>
#### Обязательные знаяения

Если вам необходимо, чтобы было введено значение, вы можете передать аргумент `required`:

```php
$password = password(
    label: 'Какой у вас пароль?',
    required: true
);
```

Если вы хотите настроить сообщение об ошибке валидации, вы также можете передать строку:

```php
$password = password(
    label: 'Какой у вас пароль?',
    required: 'Требуется пароль.'
);
```

<a name="password-validation"></a>
#### Дополнительная Валидация

Наконец, если вы хотите выполнить дополнительную логику валидации, вы можете передать замыкание в аргумент `validate`:

```php
$password = password(
    label: 'Какой у вас пароль?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 8 => 'Пароль должен состоять не менее 8 символов.',
        default => null
    }
);
```

Замыкание получит введенное значение и может вернуть сообщение об ошибке или `null`, если валидация проходит успешно.

Альтернативно вы можете использовать возможности [валидатора](/docs/{{version}}/validation) Laravel. Для этого предоставьте в аргумент `validate` массив, содержащий имя атрибута и желаемые правила проверки:

```php
$password = password(
    label: 'Какой у вас пароль?',
    validate: ['password' => 'min:8']
);
```

<a name="confirm"></a>
### Подтверждение

Если вам нужно запросить у пользователя подтверждение "да или нет", вы можете использовать функцию `confirm`. Пользователи могут использовать стрелки или нажать `y` или `n`, чтобы выбрать свой ответ. Эта функция вернет либо `true`, либо `false`.

```php
use function Laravel\Prompts\confirm;

$confirmed = confirm('Вы принимаете условия?');
```

Вы также можете включить значение по умолчанию, настраиваемые названия для меток "Да" и "Нет" и информационную подсказку:

```php
$confirmed = confirm(
    label: 'Вы принимаете условия?',
    default: false,
    yes: 'Я принимаю',
    no: 'Я отказываюсь',
    hint: 'Чтобы продолжить, необходимо принять условия.'
);
```

<a name="confirm-required"></a>
#### Обязательное "Да"

При необходимости вы можете потребовать от ваших пользователей выбрать "Да", передав аргумент `required`:

```php
$confirmed = confirm(
    label: 'Вы принимаете условия?',
    required: true
);
```

Если вы хотите настроить сообщение об ошибке валидации, вы также можете передать строку:

```php
$confirmed = confirm(
    label: 'Вы принимаете условия?',
    required: 'Вы должны принять условия, чтобы продолжить.'
);
```

<a name="select"></a>
### Выбор

Если вам нужно, чтобы пользователь выбрал из предопределенного набора вариантов, вы можете использовать функцию `select`:

```php
use function Laravel\Prompts\select;

$role = select(
    label: 'Какая роль должна быть у пользователя?',
    options: ['Member', 'Contributor', 'Owner']
);
```

Вы также можете указать значение по умолчанию и информационную подсказку:

```php
$role = select(
    label: 'Какая роль должна быть у пользователя?',
    options: ['Member', 'Contributor', 'Owner'],
    default: 'Owner',
    hint: 'Роль может быть изменена в любой момент.'
);
```

Вы также можете передать ассоциативный массив в аргументе `options`, чтобы вернуть выбранный ключ вместо его значения:

```php
$role = select(
    label: 'Какая роль должна быть у пользователя?',
    options: [
        'member' => 'Участник',
        'contributor' => 'Автор',
        'owner' => 'Владелец',
    ],
    default: 'owner'
);
```

При наличии более пяти вариантов будет использоваться прокрутка списка.  Вы можете настроить это, передав аргумент scroll:

```php
$role = select(
    label: 'Какую категорию вы хотели бы присвоить?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="select-info"></a>
#### Дополнительная информация

Аргумент `info` можно использовать для отображения дополнительной информации о текущем выделенном варианте. Если передано замыкание, оно получит значение текущего выделенного варианта и должно вернуть строку или `null`:

```php
$role = select(
    label: 'Какая роль должна быть у пользователя?',
    options: [
        'member' => 'Участник',
        'contributor' => 'Автор',
        'owner' => 'Владелец',
    ],
    info: fn (string $value) => match ($value) {
        'member' => 'Может просматривать и комментировать.',
        'contributor' => 'Может просматривать, комментировать и редактировать.',
        'owner' => 'Полный доступ ко всем ресурсам.',
        default => null,
    }
);
```

Вы также можете передать в аргумент `info` статическую строку, если информация не зависит от выделенного варианта:

```php
$role = select(
    label: 'Какая роль должна быть у пользователя?',
    options: ['Member', 'Contributor', 'Owner'],
    info: 'Роль может быть изменена в любой момент.'
);
```

<a name="select-validation"></a>
#### Дополнительная валидация

В отличие от других, функция `select` не принимает аргумент `required`, потому что невозможно выбрать ничего. Однако, вы можете передать замыкание в аргумент `validate`, если вам нужно представить вариант, но предотвратить его выбор:

```php
$role = select(
    label: 'Какая роль должна быть у пользователя?',
    options: [
        'member' => 'Учаастник',
        'contributor' => 'Автор',
        'owner' => 'Владелец',
    ],
    validate: fn (string $value) =>
        $value === 'owner' && User::where('role', 'owner')->exists()
            ? 'Владелец уже существует.'
            : null
);
```

Если аргумент `options` является ассоциативным массивом, то замыкание получит выбранный ключ, в противном случае оно получит выбранное значение. Замыкание может вернуть сообщение об ошибке или `null`, если валидация прошла успешно.

<a name="multiselect"></a>
### Множественный выбор

Если вам нужно, чтобы пользователь мог выбирать несколько вариантов, вы можете использовать функцию `multiselect`:

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'Какие разрешения следует назначить?',
    options: ['Read', 'Create', 'Update', 'Delete']
);
```

Вы также можете указать значения по умолчанию и информационную подсказку:

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'Какие разрешения следует назначить?',
    options: ['Read', 'Create', 'Update', 'Delete'],
    default: ['Read', 'Create'],
    hint: 'Разрешения могут быть обновлены в любое время.'
);
```

Вы также можете передать ассоциативный массив в аргумент `options`, чтобы возвращались ключи выбранных вариантов вместо их значений:

```php
$permissions = multiselect(
    label: 'Какие разрешения следует назначить?',
    options: [
        'read' => 'Читать',
        'create' => 'Создавать',
        'update' => 'Обновлять',
        'delete' => 'Удалять',
    ],
    default: ['read', 'create']
);
```

При наличии более пяти вариантов будет использоваться прокрутка списка.  Вы можете настроить это, передав аргумент scroll:

```php
$categories = multiselect(
    label: 'Какие категории следует присвоить?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="multiselect-info"></a>
#### Дополнительная информация

Аргумент `info` можно использовать для отображения дополнительной информации о текущем выделенном варианте. Если передано замыкание, оно получит значение текущего выделенного варианта и должно вернуть строку или `null`:

```php
$permissions = multiselect(
    label: 'Какие разрешения следует назначить?',
    options: [
        'read' => 'Читать',
        'create' => 'Создавать',
        'update' => 'Обновлять',
        'delete' => 'Удалять',
    ],
    info: fn (string $value) => match ($value) {
        'read' => 'Просмотр ресурсов и их свойств.',
        'create' => 'Создание новых ресурсов.',
        'update' => 'Изменение существующих ресурсов.',
        'delete' => 'Безвозвратное удаление ресурсов.',
        default => null,
    }
);
```

<a name="multiselect-required"></a>
#### Обязательное значение

По умолчанию пользователь может выбирать ноль или более вариантов. Вы можете передать аргумент required, чтобы требовать один или более вариантов вместо этого:

```php
$categories = multiselect(
    label: 'Какие категории следует присвоить?',
    options: Category::pluck('name', 'id'),
    required: true
);
```

Если вы хотите настроить сообщение об ошибке валидации, вы можете передать строку в аргумент `required`:

```php
$categories = multiselect(
    label: 'Какие категории следует присвоить?',
    options: Category::pluck('name', 'id'),
    required: 'Вы должны выбрать хотя бы одну категорию'
);
```

<a name="multiselect-validation"></a>
#### Дополнительная валидация

Вы можете передать замыкание в аргумент `validate`, если вам нужно представить вариант, но предотвратить его выбор:

```php
$permissions = multiselect(
    label: 'Какие разрешения должен иметь пользователь?',
    options: [
        'read' => 'Читать',
        'create' => 'Создавать',
        'update' => 'Обновлять',
        'delete' => 'Удалять',
    ],
    validate: fn (array $values) => ! in_array('read', $values)
        ? 'Всем пользователям требуется разрешение на чтение.'
        : null
);
```

Если аргумент `options` является ассоциативным массивом, то замыкание получит выбранные ключи, в противном случае оно получит выбранные значения. Замыкание может вернуть сообщение об ошибке или `null`, если валидация прошла успешно.

<a name="suggest"></a>
### Подсказка

Функция `suggest` может использоваться для предоставления автозаполнения возможных вариантов. Пользователь все равно может ввести любой ответ, независимо от подсказок автозаполнения:

```php
use function Laravel\Prompts\suggest;

$name = suggest('Как вас зовут?', ['Taylor', 'Dayle']);
```

В качестве альтернативы, вы можете передать замыкание вторым аргументом в функцию `suggest`. Замыкание будет вызываться каждый раз, когда пользователь вводит символ. Замыкание должно принимать строку, содержащую ввод пользователя до этого момента, и возвращать массив вариантов для автозаполнения:

```php
$name = suggest(
    label: 'Как вас зовут?',
    options: fn ($value) => collect(['Taylor', 'Dayle'])
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
)
```

Вы также можете включить плейсхолдер текста, значение по умолчанию и информационную подсказку:

```php
$name = suggest(
    label: 'Как вас зовут?',
    options: ['Тейлор', 'Дэйл'],
    placeholder: 'Например: Тейлор',
    default: $user?->name,
    hint: 'Это будет отображаться в вашем профиле.'
);
```

<a name="suggest-info"></a>
#### Дополнительная информация

Аргумент `info` можно использовать для отображения дополнительной информации о текущем выделенном варианте. Если передано замыкание, оно получит значение текущего выделенного варианта и должно вернуть строку или `null`:

```php
$name = suggest(
    label: 'Как вас зовут?',
    options: ['Taylor', 'Dayle'],
    info: fn (string $value) => match ($value) {
        'Taylor' => 'Администратор',
        'Dayle' => 'Автор',
        default => null,
    }
);
```

<a name="suggest-required"></a>
#### Обязательные значения

Если вам необходимо, чтобы было введено значение, вы можете передать аргумент `required`:

```php
$name = suggest(
    label: 'Как вас зовут?',
    options: ['Тейлор', 'Дэйл'],
    required: true
);
```

Если вы хотите настроить сообщение об ошибке валидации, вы также можете передать строку:

```php
$name = suggest(
    label: 'Как вас зовут?',
    options: ['Тейлор', 'Дэйл'],
    required: 'Требуется ваше имя.'
);
```

<a name="suggest-validation"></a>
#### Дополнительная валидация

Наконец, если вам нужно выполнить дополнительную логику валидации, вы можете передать замыкание в аргумент `validate`:

```php
$name = suggest(
    label: 'Как вас зовут?',
    options: ['Тейлор', 'Дэйл'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'Имя должно состоять минимум из 3 символов.',
        strlen($value) > 255 => 'Имя не должно превышать 255 символов.',
        default => null
    }
);
```

Замыкание получит введенное значение и может вернуть сообщение об ошибке или `null`, если валидация проходит успешно.

Альтернативно вы можете использовать возможности [валидатора](/docs/{{version}}/validation) Laravel. Для этого предоставьте в аргумент `validate` массив, содержащий имя атрибута и желаемые правила проверки:

```php
$name = suggest(
    label: 'Как вас зовут?',
    options: ['Тейлор', 'Дэйл'],
    validate: ['name' => 'required|min:3|max:255']
);
```

<a name="search"></a>
### Поиск

Если пользователю доступно много вариантов, функция `search` позволяет отфильтровать их поисковым запросом, а затем выбрать нужный вариант клавишами со стрелками:

```php
use function Laravel\Prompts\search;

$id = search(
    label: 'Найдите пользователя, который должен получать почту',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

Замыкание получит текст, введенный пользователем, и должно вернуть массив вариантов. Если вы возвращаете ассоциативный массив, то будет возвращен выбранный ключ, в противном случае будет возвращено его значение.

При фильтрации массива, в который вы собираетесь вернуть значение, вам следует использовать функцию `array_values` ​​или метод Collection `values`, чтобы гарантировать, что массив не станет ассоциативным:

```php
$names = collect(['Taylor', 'Abigail']);

$selected = search(
    label: 'Найдите пользователя, который должен получать почту',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

Вы также можете включить плейсхолдер и информационную подсказку:

```php
$id = search(
    label: 'Найдите пользователя, который должен получать почту',
    placeholder: 'Например: Тейлор Отвелл',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'Пользователь немедленно получит электронное письмо.'
);
```

При наличии более пяти аргументов будет использоваться прокрутка списка. Вы можете настроить это, передав аргумент `scroll`:

```php
$id = search(
    label: 'Найдите пользователя, который должен получать почту',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="search-info"></a>
#### Дополнительная информация

Аргумент `info` можно использовать для отображения дополнительной информации о текущем выделенном варианте. Если передано замыкание, оно получит значение текущего выделенного варианта и должно вернуть строку или `null`:

```php
$id = search(
    label: 'Найдите пользователя, который должен получать почту',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    info: fn (int $userId) => User::find($userId)?->email
);
```

<a name="search-validation"></a>
#### Дополнительная валидация

Если вы хотите выполнить дополнительную логику валидации, вы можете передать замыкание в аргумент `validate`:

```php
$id = search(
    label: 'Найдите пользователя, который должен получать почту',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (int|string $value) {
        $user = User::findOrFail($value);

        if ($user->opted_out) {
            return 'Этот пользователь отказался от получения почты.';
        }
    }
);
```

Если замыкание `options` возвращает ассоциативный массив, то замыкание получит выбранный ключ, в противном случае оно получит выбранное значение. Замыкание может вернуть сообщение об ошибке или `null`, если валидация прошла успешно.

<a name="multisearch"></a>
### Множественный поиск

Если у вас много вариантов для поиска и вам нужно, чтобы пользователь мог выбирать несколько элементов, функция `multisearch` позволяет пользователю вводить запрос поиска для фильтрации результатов перед выбором вариантов с помощью стрелок и пробела:

```php
use function Laravel\Prompts\multisearch;

$ids = multisearch(
    'Поиск пользователей, которым нужно отправить письмо',
    fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

Замыкание получит текст, введенный пользователем до сих пор, и должно вернуть массив вариантов. Если вы возвращаете ассоциативный массив, то будут возвращены ключи выбранных вариантов; в противном случае будут возвращены их значения.

При фильтрации массива, в который вы собираетесь вернуть значение, вам следует использовать функцию `array_values` ​​или метод Collection `values`, чтобы гарантировать, что массив не станет ассоциативным:

```php
$names = collect(['Taylor', 'Abigail']);

$selected = multisearch(
    label: 'Поиск пользователей, которым нужно отправить письмо',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

Вы также можете включить плейсхолдер текста и информационную подсказку:

```php
$ids = multisearch(
    label: 'Поиск пользователей, которым нужно отправить письмо',
    placeholder: 'Например: Тейлор Отвелл',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'Пользователь немедленно получит электронное письмо.'
);
```

До пяти вариантов будет отображаться до начала прокрутки списка. Вы можете настроить это, указав аргумент `scroll`:

```php
$ids = multisearch(
    label: 'Поиск пользователей, которым нужно отправить письмо',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="multisearch-info"></a>
#### Дополнительная информация

Аргумент `info` можно использовать для отображения дополнительной информации о текущем выделенном варианте. Если передано замыкание, оно получит значение текущего выделенного варианта и должно вернуть строку или `null`:

```php
$ids = multisearch(
    label: 'Поиск пользователей, которые должны получать почту',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    info: fn (int $userId) => User::find($userId)?->email
);
```

<a name="multisearch-required"></a>
#### Обязательное значение

По умолчанию пользователь может выбрать ноль или более вариантов. Вы можете передать аргумент required, чтобы вместо этого требовать хотя бы один вариант:

```php
$ids = multisearch(
    label: 'Поиск пользователей, которые должны получать почту',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: true
);
```

Если вы хотите настроить сообщение об ошибке валидации, вы также можете передать строку в аргумент `required`:

```php
$ids = multisearch(
    label: 'Поиск пользователей, которые должны получать почту',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: 'Вы должны выбрать хотя бы одного пользователя.'
);
```

<a name="multisearch-validation"></a>
#### Дополнительная валидация

Если вам нужно выполнить дополнительную логику валидации, вы можете передать замыкание в аргумент `validate`:

```php
$ids = multisearch(
    label: 'Поиск пользователей, которые должны получать почту',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (array $values) {
        $optedOut = User::whereLike('name', '%a%')->findMany($values);

        if ($optedOut->isNotEmpty()) {
            return $optedOut->pluck('name')->join(', ', ', and ').' have opted out.';
        }
    }
);
```

Если замыкание `options` возвращает ассоциативный массив, то замыкание получит выбранные ключи; в противном случае оно получит выбранные значения. Замыкание может вернуть сообщение об ошибке или `null`, если валидация прошла успешно.

<a name="pause"></a>
### Пауза

Функция `pause` может использоваться для отображения информационного текста пользователю и ожидания его подтверждения продолжения нажатием клавиш Enter / Return:

```php
use function Laravel\Prompts\pause;

pause('Нажмите ENTER, чтобы продолжить.');
```

<a name="autocomplete"></a>
### Автодополнение

Функция `autocomplete` позволяет предложить inline-автодополнение для возможных вариантов. По мере ввода подходящие подсказки отображаются как ghost text, который можно принять клавишей `Tab` или стрелкой вправо:

```php
use function Laravel\Prompts\autocomplete;

$name = autocomplete(
    label: 'Как вас зовут?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim']
);
```

Вы также можете передать замыкание для динамического построения вариантов на основе ввода пользователя:

```php
$file = autocomplete(
    label: 'Какой файл?',
    options: fn (string $value) => collect($files)
        ->filter(fn ($file) => str_starts_with(strtolower($file), strtolower($value)))
        ->values()
        ->all(),
);
```

<a name="autocomplete-closure"></a>
#### Динамические варианты

Вы также можете передать замыкание для динамического построения вариантов на основе ввода пользователя. Замыкание будет вызываться каждый раз, когда пользователь вводит символ, и должно вернуть массив вариантов для автодополнения:

```php
$file = autocomplete(
    label: 'Какой файл?',
    options: fn (string $value) => collect($files)
        ->filter(fn ($file) => str_starts_with(strtolower($file), strtolower($value)))
        ->values()
        ->all(),
);
```

<a name="autocomplete-required"></a>
#### Обязательные значения

Если значение обязательно, передайте аргумент `required`:

```php
$name = autocomplete(
    label: 'Как вас зовут?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    required: true
);
```

Если вы хотите настроить сообщение об ошибке валидации, вы также можете передать строку:

```php
$name = autocomplete(
    label: 'Как вас зовут?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    required: 'Ваше имя обязательно.'
);
```

<a name="autocomplete-validation"></a>
#### Дополнительная валидация

Наконец, если вам нужно выполнить дополнительную логику валидации, вы можете передать замыкание в аргумент `validate`:

```php
$name = autocomplete(
    label: 'Как вас зовут?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'Имя должно состоять минимум из 3 символов.',
        strlen($value) > 255 => 'Имя не должно превышать 255 символов.',
        default => null
    }
);
```

Замыкание получит введенное значение и может вернуть сообщение об ошибке или `null`, если валидация пройдет успешно.

<a name="transforming-input-before-validation"></a>
## Преобразование входных данных перед проверкой

Иногда вам может потребоваться преобразовать вводимые данные до того, как произойдет проверка. Например, вы можете удалить пробелы из любых предоставленных строк. Для этого многие функции приглашения предоставляют аргумент `transform`, который принимает замыкание:

```php
$name = text(
    label: 'Как вас зовут?',
    transform: fn (string $value) => trim($value),
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'Имя должно состоять минимум из 3 символов.',
        strlen($value) > 255 => 'Имя не должно превышать 255 символов.',
        default => null
    }
);
```

<a name="forms"></a>
## Формы

Часто у вас будет несколько подсказок, которые будут отображаться последовательно для сбора информации перед выполнением дополнительных действий. Вы можете использовать функцию `form` для создания сгруппированного набора подсказок, которые пользователь должен выполнить:

```php
use function Laravel\Prompts\form;

$responses = form()
    ->text('Как вас зовут?', required: true)
    ->password('Какой у вас пароль?', validate: ['password' => 'min:8'])
    ->confirm('Вы принимаете условия?')
    ->submit();
```

Метод `submit` вернет числовой индексированный массив, содержащий все ответы на запросы формы. Однако вы можете указать имя для каждого приглашения с помощью аргумента `name`. Если указано имя, доступ к ответу на указанное приглашение можно получить по этому имени:

```php
use App\Models\User;
use function Laravel\Prompts\form;

$responses = form()
    ->text('Как вас зовут?', required: true, name: 'name')
    ->password(
        label: 'Какой у вас пароль?',
        validate: ['password' => 'min:8'],
        name: 'password'
    )
    ->confirm('Вы принимаете условия?')
    ->submit();

User::create([
    'name' => $responses['name'],
    'password' => $responses['password'],
]);
```

Основным преимуществом использования функции `form` является возможность для пользователя вернуться к предыдущим запросам в форме с помощью `CTRL + U`. Это позволяет пользователю исправлять ошибки или изменять выбор без необходимости отмены и перезапуска всей формы.

Если вам нужен более детальный контроль над подсказкой в ​​форме, вы можете вызвать метод `add` вместо прямого вызова одной из функций подсказки. Методу `add` передаются все предыдущие ответы, предоставленные пользователем:

```php
use function Laravel\Prompts\form;
use function Laravel\Prompts\outro;
use function Laravel\Prompts\text;

$responses = form()
    ->text('Как вас зовут?', required: true, name: 'name')
    ->add(function ($responses) {
        return text("Сколько тебе лет, {$responses['name']}?");
    }, name: 'age')
    ->submit();

outro("Ваше имя {$responses['name']} и вам {$responses['age']} лет.");
```

<a name="informational-messages"></a>
## Информационные Сообщения

Функции `note`, `info`, `warning`, `error` и `alert` могут быть использованы для отображения информационных сообщений:

```php
use function Laravel\Prompts\info;

info('Пакет успешно установлен.');
```

<a name="callouts"></a>
## Callouts

Функция `callout` отображает сообщение в рамке с label и content. Callouts полезны для важной информации, которая должна выделяться: deployment summaries, error details или status updates:

```php
use function Laravel\Prompts\callout;

callout(
    label: 'Environment Configured',
    content: 'Your application is running in production mode with 4 workers.',
);
```

Можно передать `warning` или `error` в аргумент `type`, чтобы изменить visual style callout:

```php
callout(
    label: 'Deprecation Notice',
    content: 'The `--prefer-stable` flag will be removed in v4.0. Use `--stability=stable` instead.',
    type: 'warning',
);

callout(
    label: 'Database Connection Failed',
    content: 'Could not connect to MySQL on 127.0.0.1:3306.',
    type: 'error',
);
```

Аргумент `info` добавляет footer line к callout, что удобно для metadata вроде IDs или timestamps:

```php
callout(
    label: 'Deployment Summary',
    content: 'Your application was deployed to production.',
    info: 'deploy-id: d4f8a2c',
);
```

<a name="callout-rich-content"></a>
#### Rich content

Вместо строки можно передать массив строк и elements, чтобы построить rich, structured callouts. Класс `Element` предоставляет factory methods для headings, bulleted lists, numbered lists, key-value lists и links:

```php
use Laravel\Prompts\Elements\Element;

use function Laravel\Prompts\callout;

callout('Deployment Summary', [
    'Your application was deployed to production at 2024-03-15 14:32 UTC.',
    Element::heading('What Changed'),
    Element::bulletedList([
        'Migrated 3 pending database migrations',
        'Cleared and rebuilt route cache',
        'Restarted 4 queue workers',
    ]),
    Element::heading('Next Steps'),
    Element::numberedList([
        'Verify the health check endpoint at /up',
        'Monitor error rates for the next 15 minutes',
        'Confirm background jobs are processing',
    ]),
]);
```

Также можно использовать `Element::keyValueList` для отображения labeled data:

```php
callout('Database Connection Failed', [
    'Could not connect to the database server.',
    Element::keyValueList([
        'Host' => '127.0.0.1',
        'Port' => '3306',
        'Database' => 'forge',
        'Status' => 'Connection refused',
    ]),
], type: 'error');
```

Метод `Element::link` создает кликабельную гиперссылку в терминалах, которые поддерживают [OSC 8](https://gist.github.com/egmontkob/eb114294efbcd5adb1944c9f3cb5feda). Можно передать только URL или URL с пользовательской меткой:

```php
callout('Server Health Check', [
    'Multiple services are reporting degraded performance.',
    Element::heading('Affected Services'),
    'Look here: '.Element::link('https://example.com/health', 'Health Dashboard'),
    Element::link('https://example.com/health'),
]);
```

Если label не передан, сам URL будет отображен как link text.

<a name="tables"></a>
## Таблицы

Функция `table` упрощает отображение нескольких строк и столбцов данных. Все, что вам нужно сделать, это указать имена столбцов и данные для таблицы:

```php
use function Laravel\Prompts\table;

table(
    headers: ['Name', 'Email'],
    rows: User::all(['name', 'email'])->toArray()
);
```

<a name="spin"></a>
## Spin

Функция `spin` отображает спиннер вместе с необязательным сообщением во время выполнения указанного обратного вызова. Она служит для обозначения выполнения процессов и возвращает результаты обратного вызова по его завершении:

```php
use function Laravel\Prompts\spin;

$response = spin(
    callback: fn () => Http::get('http://example.com'),
    message: 'Получение ответа...'
);
```

> [!WARNING]
> Функция `spin` требует наличие расширения PHP [PCNTL](https://www.php.net/manual/en/book.pcntl.php) для анимации спиннера. Когда это расширение недоступно, вместо этого будет отображаться статическая версия спиннера.

<a name="progress"></a>
## Прогресс-бар

Для длительных задач может быть полезно показать полосу прогресса, которая информирует пользователей о том, насколько завершена задача. Используя функцию `progress`, Laravel будет отображать полосу прогресса и продвигать ее для каждой итерации по заданному итерируемому значению:

```php
use function Laravel\Prompts\progress;

$users = progress(
    label: 'Обновление пользователей',
    steps: User::all(),
    callback: fn ($user) => $this->performTask($user)
);
```

Функция `progress` действует как функция map и вернет массив, содержащий возвращаемое значение каждой итерации вашего обратного вызова.

Обратный вызов также может принимать экземпляр `Laravel\Prompts\Progress`, что позволяет вам изменять метку и подсказку на каждой итерации:

```php
$users = progress(
    label: 'Обновление пользователей',
    steps: User::all(),
    callback: function ($user, $progress) {
        $progress
            ->label("Обновление {$user->name}")
            ->hint("Создано {$user->created_at}");

        return $this->performTask($user);
    },
    hint: 'Это может занять некоторое время.'
);
```

Иногда вам может потребоваться больше ручного контроля над тем, как продвигается полоса прогресса. Сначала определите общее количество шагов, через которые будет проходить процесс. Затем продвигайте полосу прогресса с помощью метода `advance` после обработки каждого элемента:

```php
$progress = progress(label: 'Обновление пользователей', steps: 10);

$users = User::all();

$progress->start();

foreach ($users as $user) {
    $this->performTask($user);

    $progress->advance();
}

$progress->finish();
```

<a name="task"></a>
## Задача

Функция `task` отображает задачу с подписью, spinner и прокручиваемой областью живого вывода во время выполнения callback. Она хорошо подходит для долгих процессов, например установки зависимостей или deployment-скриптов:

```php
use function Laravel\Prompts\task;

task(
    label: 'Установка зависимостей',
    callback: function ($logger) {
        // Долгий процесс...
    }
);
```

Callback получает экземпляр `Logger`, который можно использовать для вывода строк, статусных сообщений и потокового текста в области вывода задачи.

> [!WARNING]
> Функция `task` требует расширение PHP [PCNTL](https://www.php.net/manual/ru/book.pcntl.php) для анимации spinner. Если расширение недоступно, будет показана статическая версия задачи.

<a name="task-logging"></a>
#### Логирование строк

Метод `line` записывает одну строку лога в прокручиваемую область вывода задачи:

```php
task(
    label: 'Установка зависимостей',
    callback: function ($logger) {
        $logger->line('Разрешение пакетов...');
        // ...
        $logger->line('Загрузка laravel/framework');
    }
);
```

<a name="task-status-messages"></a>
#### Статусные сообщения

Методы `success`, `warning` и `error` отображают выделенные статусные сообщения над областью лога:

```php
task(
    label: 'Развертывание приложения',
    callback: function ($logger) {
        $logger->line('Запуск миграций...');
        $logger->warning('Нет новых миграций.');
        $logger->success('Кеш очищен!');
    }
);
```

<a name="task-label"></a>
#### Обновление подписи

Метод `label` позволяет обновить подпись задачи во время ее выполнения:

```php
task(
    label: 'Запуск развертывания...',
    callback: function ($logger) {
        $logger->label('Получение последних изменений...');
        // ...
        $logger->label('Запуск миграций...');
        // ...
        $logger->label('Очистка кеша...');
        // ...
    }
);
```

<a name="task-sub-label"></a>
#### Отображение sub-label

Метод `subLabel` отображает приглушенную строку под основной label задачи. Это удобно для ephemeral status, например текущего выполняемого шага. Передайте пустую строку, чтобы очистить sub-label:

```php
task(
    label: 'Deploying',
    callback: function ($logger) {
        $logger->subLabel('Building assets...');
        // ...
        $logger->subLabel('Running migrations...');
        // ...
        $logger->subLabel('');
    }
);
```

Также можно передать начальный sub-label через аргумент `subLabel`:

```php
task(
    label: 'Deploying',
    callback: function ($logger) {
        // ...
    },
    subLabel: 'Preparing...'
);
```

<a name="task-streaming"></a>
#### Потоковый текст

Для процессов, которые постепенно производят вывод, например AI-ответов, метод `partial` позволяет выводить текст частями. После завершения потока вызовите `commitPartial`:

```php
task(
    label: 'Генерация ответа...',
    callback: function ($logger) {
        foreach ($words as $word) {
            $logger->partial($word . ' ');
        }

        $logger->commitPartial();
    }
);
```

<a name="task-limit"></a>
#### Настройка ограничения вывода

По умолчанию задача отображает до 10 строк прокручиваемого вывода. Вы можете настроить это через аргумент `limit`:

```php
task(
    label: 'Установка зависимостей',
    callback: function ($logger) {
        // ...
    },
    limit: 20
);
```

<a name="task-keep-summary"></a>
#### Сохранение summary

По умолчанию output задачи стирается после завершения callback. Если вы хотите оставить status messages на экране после завершения задачи, передайте аргумент `keepSummary`:

```php
task(
    label: 'Deploying',
    callback: function ($logger) {
        $logger->success('Assets built');
        // ...
        $logger->success('Migrations complete');
    },
    keepSummary: true,
);
```

<a name="stream"></a>
## Поток

Функция `stream` отображает текст, который постепенно появляется в терминале. Это удобно для AI-контента или любого вывода, поступающего частями:

```php
use function Laravel\Prompts\stream;

$stream = stream();

foreach ($words as $word) {
    $stream->append($word . ' ');
    usleep(25_000);
}

$stream->close();
```

<a name="terminal-title"></a>
## Заголовок терминала

Функция `title` обновляет заголовок окна или вкладки терминала пользователя:

```php
use function Laravel\Prompts\title;

title('Установка зависимостей');
```

Чтобы сбросить заголовок терминала, передайте пустую строку:

```php
title('');
```

<a name="clear"></a>
## Очистка терминала

Функция `clear` может использоваться для очистки пользовательского терминала:

```php
use function Laravel\Prompts\clear;

clear();
```

<a name="terminal-considerations"></a>
## Учет Особенностей Терминала

<a name="terminal-width"></a>
#### Ширина Терминала

Если длина какой-либо метки, варианта или сообщения о валидации превышает количество "столбцов" в терминале пользователя, она будет автоматически усечена до соответствия. Рассмотрите возможность минимизации длины этих строк, если ваши пользователи могут использовать более узкие терминалы. Обычно безопасная максимальная длина составляет 74 символа для поддержки терминала шириной 80 символов.

<a name="terminal-height"></a>
#### Высота Терминала

Для всех запросов, которые принимают аргумент `scroll`, настроенное значение будет автоматически уменьшено для соответствия высоте терминала пользователя, включая место для сообщения о валидации.

<a name="fallbacks"></a>
## Неподдерживаемые Окружения и Резервные Варианты

Laravel Prompts поддерживает macOS, Linux и Windows с использованием WSL. Из-за ограничений в версии PHP для Windows в настоящее время невозможно использовать Laravel Prompts на Windows вне WSL.

По этой причине Laravel Prompts поддерживает откат к альтернативной реализации, такой как [Symfony Console Question Helper](https://symfony.com/doc/current/components/console/helpers/questionhelper.html).

> [!NOTE]
> При использовании Laravel Prompts с фреймворком Laravel резервные варианты для каждого запроса настроены для вас и будут автоматически включены в неподдерживаемых окружениях.

<a name="fallback-conditions"></a>
#### Условия Резервных Вариантов

Если вы не используете Laravel или нуждаетесь в настройке условий использования резервного поведения, вы можете передать булево значение методу `fallbackWhen` статического класса `Prompt`:

```php
use Laravel\Prompts\Prompt;

Prompt::fallbackWhen(
    ! $input->isInteractive() || windows_os() || app()->runningUnitTests()
);
```

<a name="fallback-behavior"></a>
#### Поведение Резервного Варианта

Если вы не используете Laravel или вам нужно настроить поведение резервного варианта, вы можете передать замыкание методу `fallbackUsing` статического класса каждого prompt:

```php
use Laravel\Prompts\TextPrompt;
use Symfony\Component\Console\Question\Question;
use Symfony\Component\Console\Style\SymfonyStyle;

TextPrompt::fallbackUsing(function (TextPrompt $prompt) use ($input, $output) {
    $question = (new Question($prompt->label, $prompt->default ?: null))
        ->setValidator(function ($answer) use ($prompt) {
            if ($prompt->required && $answer === null) {
                throw new \RuntimeException(
                    is_string($prompt->required) ? $prompt->required : 'Required.'
                );
            }

            if ($prompt->validate) {
                $error = ($prompt->validate)($answer ?? '');

                if ($error) {
                    throw new \RuntimeException($error);
                }
            }

            return $answer;
        });

    return (new SymfonyStyle($input, $output))
        ->askQuestion($question);
});
```

Резервные варианты должны быть настроены индивидуально для каждого класса prompt. Замыкание будет получать экземпляр класса prompt и должно возвращать соответствующий тип для prompt.

<a name="testing"></a>
## Тестирование

Laravel предоставляет ряд методов для проверки того, что ваша команда отображает ожидаемые сообщения Prompt:

```php tab=Pest
test('report generation', function () {
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
public function test_report_generation(): void
{
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
}
```
