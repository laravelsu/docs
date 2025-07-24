---
git: 6bc6cd05d1b1754de15eb73e6160384c7aaa094f
---

# Руководство по обновлению

- [Обновление с 11.0 версии до 12.x](#upgrade-12.0)

<a name="high-impact-changes"></a>
## Изменения с высоким уровнем влияния

<!-- <div class="content-list" markdown="1"> -->

- [Обновление зависимостей](#updating-dependencies)
- [Обновление Laravel Installer](#updating-the-laravel-installer)

<!-- </div> -->

<a name="medium-impact-changes"></a>
## Изменения со средним уровнем влияния

<!-- <div class="content-list" markdown="1"> -->

- [Модели и UUIDv7](#models-and-uuidv7)

<!-- </div> -->

<a name="low-impact-changes"></a>
###### Изменения с низким уровнем влияния

<!-- <div class="content-list" markdown="1"> -->

- [Carbon 3](#carbon-3)  
- [Сопоставление индексов результатов в Concurrency](#concurrency-result-index-mapping)
- [Разрешение зависимостей контейнера](#container-class-dependency-resolution)
- [Проверка изображений больше не включает SVG](#image-validation)
- [Корневой путь по умолчанию для локального диска файловой системы](#local-filesystem-disk-default-root-path)
- [Просмотр баз данных с несколькими схемами](#multi-schema-database-inspecting)
- [Слияние вложенных массивов в запросах](#nested-array-request-merging)

<!-- </div> -->

<a name="upgrade-11.0"></a>
## Обновление с 11.0 версии до 12.x

<a name="estimated-upgrade-time-??-minutes"></a>
#### Приблизительное время обновления: 5 минут

> [!NOTE]
> Мы стараемся задокументировать каждое возможное изменение, которое может привести к нарушению совместимости. Поскольку некоторые из этих критических изменений находятся в малоизвестных частях фреймворка, только часть этих изменений может повлиять на ваше приложение. Хотите сэкономить время? Вы можете использовать [Laravel Shift](https://laravelshift.com/) , чтобы автоматизировать процесс обновления вашего приложения.

<a name="updating-dependencies"></a>
### Обновление зависимостей

**Вероятность воздействия: высокая**

#### Зависимости Composer

Обновите следующие зависимости в вашем файле `composer.json`:

<!-- <div class="content-list" markdown="1"> -->

- `laravel/framework` to `^12.0`
- `phpunit/phpunit` to `^11.0`
- `pestphp/pest` to `^3.0` (если установлено)

<!-- </div> -->

<a name="carbon-3"></a>
#### Carbon 3


**Вероятность влияния: низкая**

Поддержка [Carbon 2.x](https://carbon.nesbot.com/docs/) удалена. Laravel 12 требует использования [Carbon 3.x](https://carbon.nesbot.com/docs/#api-carbon-3).

<a name="updating-the-laravel-installer"></a>
### Обновление Laravel Installer

Если вы используете CLI-установщик Laravel, обновите его до версии, совместимой с Laravel 12.x и новыми [стартер-китами](https://laravel.com/starter-kits).  
Если установили через `composer global require`, выполните:

```bash
composer global update laravel/installer
```

Если устанавливали Laravel через `php.new`, просто повторите установку для своей ОС:

**macOS:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

**Windows (PowerShell, от имени администратора):**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

**Linux:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

Если вы используете [Laravel Herd](https://herd.laravel.com), обновите его до последней версии.


<a name="authentication"></a>
### Аутентификация

<a name="updated-databasetokenrepository-constructor-signature"></a>
#### Обновлённая сигнатура конструктора `DatabaseTokenRepository`

**Вероятность влияния: очень низкая**

Теперь конструктор класса `Illuminate\Auth\Passwords\DatabaseTokenRepository` ожидает параметр `$expires` в **секундах**, а не в минутах.

<a name="concurrency"></a>
### Concurrency

<a name="concurrency-result-index-mapping"></a>
#### Сопоставление индексов результатов

**Вероятность влияния: низкая**

Теперь при использовании `Concurrency::run` с ассоциативным массивом, ключи сохраняются в результатах:

```php
$result = Concurrency::run([
    'task-1' => fn () => 1 + 1,
    'task-2' => fn () => 2 + 2,
]);

// ['task-1' => 2, 'task-2' => 4]
```


<a name="container"></a>
### Контейнер

<a name="container-class-dependency-resolution"></a>
#### Разрешение зависимостей класса

**Вероятность влияния: низкая**

Контейнер теперь учитывает значения по умолчанию свойств при разрешении зависимостей:

```php
class Example
{
    public function __construct(public ?Carbon $date = null) {}
}

$example = resolve(Example::class);

// До 12.x
$example->date instanceof Carbon;

// С 12.x
$example->date === null;
```

<a name="database"></a>
### База данных

<a name="multi-schema-database-inspecting"></a>
#### Просмотр баз с несколькими схемами

**Вероятность влияния: низкая**

Методы `Schema::getTables()`, `getViews()`, `getTypes()` теперь возвращают результаты по **всем схемам** по умолчанию:

```php
$tables = Schema::getTables(); // Все таблицы во всех схемах

$tables = Schema::getTables(schema: 'main'); // Только схема 'main'

$tables = Schema::getTables(schema: ['main', 'blog']); // Несколько схем
```

Метод `Schema::getTableListing()` теперь возвращает имена таблиц с префиксом схемы:

```php
$tables = Schema::getTableListing();
// ['main.migrations', 'main.users', 'blog.posts']

$tables = Schema::getTableListing(schema: 'main');
// ['main.migrations', 'main.users']

$tables = Schema::getTableListing(schema: 'main', schemaQualified: false);
// ['migrations', 'users']
```

Команды `db:table` и `db:show` теперь показывают все схемы и для MySQL, MariaDB, SQLite (как это уже делалось в PostgreSQL и SQL Server).

#### Обновлённая сигнатура конструктора `Blueprint`

**Вероятность влияния: очень низкая**

Теперь конструктор класса `Illuminate\Database\Schema\Blueprint` ожидает первым аргументом объект `Illuminate\Database\Connection`.


<a name="eloquent"></a>
### Eloquent

<a name="models-and-uuidv7"></a>
#### Модели и UUIDv7

**Вероятность влияния: средняя**

Трейт `HasUuids` теперь генерирует UUID версии 7.  
Если вы хотите продолжить использовать UUID v4 с упорядоченными значениями, используйте трейт `HasVersion4Uuids`:

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids; // Удалить
use Illuminate\Database\Eloquent\Concerns\HasVersion4Uuids as HasUuids; // Добавить
```

Трейт `HasVersion7Uuids` удалён — теперь его заменяет `HasUuids`.


<a name="requests"></a>
### Requests

<a name="nested-array-request-merging"></a>
#### Слияние вложенных массивов

**Вероятность влияния: низкая**

Метод `$request->mergeIfMissing()` теперь поддерживает слияние с помощью "dot" нотации:

```php
$request->mergeIfMissing([
    'user.last_name' => 'Otwell',
]);
```

Если раньше вы рассчитывали, что ключ `'user.last_name'` создаст одноуровневый массив — пересмотрите логику.

<a name="storage"></a>
### Хранилище

<a name="local-filesystem-disk-default-root-path"></a>
#### Корневой путь по умолчанию для локального диска файловой системы

**Вероятность влияния: низкая**

Если ваше приложение явно не определяет `local` диск в конфигурации файловой системы, Laravel теперь будет по умолчанию использовать `storage/app/private` в качестве корня локального диска. В предыдущих версиях это был `storage/app`. В результате вызовы `Storage::disk('local')` будут читать и записывать данные в `storage/app/private`, если не указано иное. Чтобы восстановить прежнее поведение, вы можете вручную определить `local` диск и указать нужный путь к корню.

<a name="validation"></a>
### Валидация

<a name="image-validation"></a>
#### Правило `image` больше не пропускает SVG

**Вероятность влияния: низкая**

Теперь по умолчанию SVG-файлы не считаются изображениями при использовании правила `image`. Чтобы разрешить SVG, укажите это явно:

```php
'photo' => 'required|image:allow_svg'

// Или с использованием объекта правил:
'photo' => ['required', File::image(allowSvg: true)],
```


<a name="miscellaneous"></a>
### Разное

Мы также рекомендуем вам просмотреть изменения в `laravel/laravel` [репозиторий GitHub](https://github.com/laravel/laravel). Хотя многие из этих изменений не обязательны, вы можете захотеть синхронизировать эти файлы с вашим приложением. Некоторые из этих изменений будут описаны в этом руководстве по обновлению, а другие, например изменения в файлах конфигурации или комментариях, не будут рассмотрены. Вы можете легко просмотреть изменения с помощью [инструмента сравнения GitHub](https://github.com/laravel/laravel/compare/10.x...11.x) и выбрать, какие обновления важны для вас.
