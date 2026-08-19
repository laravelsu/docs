---
git: b2300431921ec68c8ba31bbef248bff1511e8eff
---

# Laravel Scout

<a name="introduction"></a>
## Введение

[Laravel Scout](https://github.com/laravel/scout) предоставляет простое решение на основе драйверов для добавления полнотекстового поиска в ваши [модели Eloquent](/docs/{{version}}/eloquent). Используя наблюдателей (observers) моделей, Scout будет автоматически синхронизировать поисковые индексы с данными моделей Eloquent.

Scout поставляется со встроенным движком `database`, который использует полнотекстовые индексы MySQL / PostgreSQL и условия `LIKE` для поиска по существующей базе данных без внешнего сервиса. Для большинства приложений этого достаточно. Обзор всех возможностей поиска в Laravel доступен в [документации по поиску](/docs/{{version}}/search).

Scout также включает драйверы для [Algolia](https://www.algolia.com/), [Meilisearch](https://www.meilisearch.com) и [Typesense](https://typesense.org), когда вам нужны исправление опечаток, фасетная фильтрация или гео-поиск в большом масштабе. Для локальной разработки доступен драйвер «коллекций», а также можно писать [собственные движки](#custom-engines).

<a name="installation"></a>
## Установка

Сначала установите Scout через менеджер пакетов Composer:

```shell
composer require laravel/scout
```

После установки Scout вы должны опубликовать файл конфигурации Scout с помощью Artisan-команды `vendor:publish`. Эта команда добавит файл конфигурации `scout.php` в каталог `config` вашего приложения:

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

Наконец, добавьте трейт (trait) `Laravel\Scout\Searchable` к модели, которую вы хотите сделать доступной для поиска. Этот трейт зарегистрирует наблюдателя модели, который будет автоматически синхронизировать модель с вашим драйвером поиска:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;
}
```

<a name="queueing"></a>
### Использование очереди

При использовании движка, отличного от отличного от `database` или `collection`, рекомендуется настроить [драйвер очереди](/docs/{{version}}/queues) перед использованием библиотеки. Запуск рабочего процесса очереди позволит Scout помещать все операции, синхронизирующие информацию модели с поисковыми индексами, в очередь, что обеспечит более быстрое время ответа для веб-интерфейса вашего приложения.

После настройки драйвера очереди установите значение параметра `queue` в вашем конфигурационном файле `config/scout.php` в `true`:

```php
'queue' => true,
```

Даже когда параметр `queue` установлен в `false`, важно помнить, что некоторые драйверы Scout, такие как Algolia и Meilisearch, всегда индексируют записи асинхронно. Это означает, что даже если операция индексации завершена в вашем приложении Laravel, сама поисковая система может не сразу отразить новые и обновленные записи.

Чтобы указать соединение и очередь, которые используют ваши задания Scout, вы можете определить параметр конфигурации `queue` как массив:

```php
'queue' => [
    'connection' => 'redis',
    'queue' => 'scout'
],
```

Конечно, если вы настроите соединение и очередь, которые используют задания Scout, вам следует запустить обработчик очереди для обработки заданий в этом соединении и очереди:

```shell
php artisan queue:work redis --queue=scout
```

<a name="unique-jobs"></a>
#### Уникальные задания

В приложениях с большим количеством записей вам может понадобиться запретить Scout ставить в очередь повторяющиеся задания для одних и тех же записей моделей. Вы можете включить уникальные задания индексации, зарегистрировав классы заданий `MakeSearchableUniquely` и `RemoveFromSearchUniquely`, обычно в методе `boot` сервис-провайдера:

```php
use Laravel\Scout\Jobs\MakeSearchableUniquely;
use Laravel\Scout\Jobs\RemoveFromSearchUniquely;
use Laravel\Scout\Scout;

Scout::makeSearchableUsing(MakeSearchableUniquely::class);
Scout::removeFromSearchUsing(RemoveFromSearchUniquely::class);
```

Эти задания используют [блокировки уникальных заданий](/docs/{{version}}/queues#unique-jobs) Laravel, чтобы избежать отправки повторяющихся операций индексации в очередь для одних и тех же searchable-записей моделей, пока соответствующее задание уже находится в очереди.

<a name="driver-prerequisites"></a>
## Требования к драйверам

<a name="algolia"></a>
### Algolia

При использовании драйвера Algolia вы должны настроить учетные данные Algolia `id` и `secret` в файле конфигурации `config/scout.php`. После того как ваши учетные данные будут настроены, вам также необходимо будет установить Algolia PHP SDK через диспетчер пакетов Composer:

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
### Meilisearch

[Meilisearch](https://www.meilisearch.com) это быстрая поисковая система с открытым исходным кодом. Если вы не знаете, как установить Meilisearch на свой локальный компьютер, вы можете использовать [Laravel Sail](/docs/{{version}}/sail#meilisearch), официально поддерживаемую Laravel среду разработки Docker.

При использовании драйвера Meilisearch вам необходимо установить Meilisearch PHP SDK через менеджер пакетов Composer:

```shell
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

Затем установите переменную среды `SCOUT_DRIVER`, а также учетные данные вашего Meilisearch `host` и `key` в файле` .env` вашего приложения:

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

Для получения дополнительной информации обратитесь к [документации MeiliSearch](https://docs.meilisearch.com/learn/getting_started/quick_start.html).

Кроме того, по [документации о совместимости Meilisearch](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch) убедитесь, что установленная версия `meilisearch/meilisearch-php` совместима с версией исполняемого файла Meilisearch.

> [!WARNING]
> При обновлении Scout в приложении, которое использует MeiliSearch, вы всегда должны [просматривать любые дополнительные критические изменения](https://github.com/meilisearch/MeiliSearch/releases) в самой службе Meilisearch.

<a name="typesense"></a>
### Typesense

[Typesense](https://typesense.org) - это быстрый и открытый поисковый движок, поддерживающий поиск по ключевым словам, семантический поиск, гео-поиск и векторный поиск.

Вы можете [разместить Typesense у себя](https://typesense.org/docs/guide/install-typesense.html#option-2-local-machine-self-hosting) или использовать [Typesense Cloud](https://cloud.typesense.org).

Чтобы начать использовать Typesense с Scout, установите Typesense PHP SDK через менеджер пакетов Composer:

```shell
composer require typesense/typesense-php
```

Затем установите переменную среды `SCOUT_DRIVER`, а также укажите адрес вашего Typesense и ключ API в файле `.env` вашего приложения:

```ini
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```

Если вы используете [Laravel Sail](/docs/{{version}}/sail), вам может потребоваться настроить переменную среды `TYPESENSE_HOST`, чтобы она соответствовала имени контейнера Docker. Вы также можете указать порт, путь и протокол вашей установки:

```ini
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

Дополнительные настройки и определения схемы для коллекций Typesense можно найти в конфигурационном файле вашего приложения `config/scout.php`. Для получения дополнительной информации о Typesense, пожалуйста, обратитесь к [документации Typesense](https://typesense.org/docs/guide/#quick-start).

<a name="configuration"></a>
## Настройка

<a name="configuring-searchable-data"></a>
### Настройка поисковых данных

По умолчанию вся форма `toArray` данной модели будет сохранена в ее поисковом индексе. Если вы хотите настроить данные, которые синхронизируются с поисковым индексом, вы можете переопределить метод `toSearchableArray` в модели:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * Переопределение массива индекса модели по умолчанию
     *
     * @return array<string, mixed>
     */
    public function toSearchableArray(): array
    {
        $array = $this->toArray();

        // Настраиваем массив данных...

        return $array;
    }
}
```

<a name="configuring-search-engines-per-model"></a>
#### Настройка поисковых драйверов для каждой модели

При выполнении поиска Scout обычно использует поисковый драйвер, указанный по умолчанию в файле конфигурации `scout` вашего приложения. Однако поисковый движок для определенной модели можно изменить, переопределив метод `searchableUsing` в модели:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Engines\Engine;
use Laravel\Scout\Scout;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Используйте движок для индексации модели.
     */
    public function searchableUsing(): Engine
    {
        return Scout::engine('meilisearch');
    }
}
```

<a name="database-and-collection-engines"></a>
## Движки базы данных и коллекций

<a name="database-engine"></a>
### Движок базы данных

> [!WARNING]
> В настоящее время движок базы данных поддерживает MySQL и PostgreSQL, которые обеспечивают быструю полнотекстовую индексацию столбцов.

Движок `database` использует полнотекстовые индексы MySQL / PostgreSQL и условия `LIKE` для прямого поиска по существующей базе данных. Для многих приложений это самый простой и практичный способ добавить поиск без внешнего сервиса и дополнительной инфраструктуры.

Чтобы использовать движок базы данных, установите значение переменной среды `SCOUT_DRIVER` равным `database`:

```ini
SCOUT_DRIVER=database
```

После настройки вы можете [определить данные для поиска](#configuring-searchable-data) и начать [выполнять поисковые запросы](#searching) к моделям. В отличие от сторонних движков, движок базы данных не требует отдельного этапа индексации - он ищет напрямую по таблицам базы данных.

#### Настройка стратегий поиска в базе данных

По умолчанию движок базы данных выполняет запрос `LIKE` к каждому атрибуту модели, который вы [настроили для поиска](#configuring-searchable-data). Однако для конкретных столбцов можно назначить более эффективные стратегии. Атрибут `SearchUsingFullText` использует полнотекстовый индекс базы данных, а `SearchUsingPrefix` ищет только начало строк (`example%`) вместо поиска внутри всей строки (`%example%`).

Чтобы определить это поведение, назначьте PHP-атрибуты методу `toSearchableArray` вашей модели. Любые столбцы без атрибута продолжат использовать стратегию `LIKE` по умолчанию:

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * Получить индексируемый массив данных для модели.
 *
 * @return array<string, mixed>
 */
#[SearchUsingPrefix(['id', 'email'])]
#[SearchUsingFullText(['bio'])]
public function toSearchableArray(): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'bio' => $this->bio,
    ];
}
```

> [!WARNING]
> Перед тем как указывать, что столбец должен использовать ограничения полнотекстового запроса, убедитесь, что столбцу был назначен [полнотекстовый индекс](/docs/{{version}}/migrations#available-index-types).

<a name="collection-engine"></a>
### Движок коллекций

Движок `collection` предназначен для быстрых прототипов, очень маленьких наборов данных (несколько сотен записей) или запуска тестов. Он извлекает все возможные записи из базы данных и фильтрует их в PHP с помощью помощника `Str::is`, поэтому ему не требуется индексация или специфичные возможности базы данных. Для всего, что выходит за рамки тривиальных сценариев, следует использовать [движок базы данных](#database-engine).

Чтобы использовать движок коллекций, установите значение переменной среды `SCOUT_DRIVER` равным `collection` или укажите драйвер `collection` непосредственно в конфигурационном файле `scout` вашего приложения:

```ini
SCOUT_DRIVER=collection
```

После того как вы указали драйвер коллекций в качестве предпочтительного, вы можете начать [выполнять поисковые запросы](#searching) к моделям. Индексирование поискового движка, например индексация для заполнения индексов Algolia, Meilisearch или Typesense, при использовании движка коллекций не требуется.

#### Отличия от движка базы данных

В то время как движок базы данных использует полнотекстовые индексы и условия `LIKE` для эффективного поиска совпадающих записей, движок коллекций извлекает все записи и фильтрует их в PHP. Движок коллекций наиболее переносим, поскольку работает со всеми реляционными базами данных, поддерживаемыми Laravel (включая SQLite и SQL Server); однако он значительно менее эффективен, чем движок базы данных, и не должен использоваться с большими наборами данных.

<a name="third-party-engine-configuration"></a>
## Настройка сторонних движков

Следующие параметры конфигурации относятся только к сторонним поисковым движкам, таким как Algolia, Meilisearch или Typesense. Если вы используете [движок базы данных](#database-engine), этот раздел можно пропустить.

<a name="configuring-model-indexes"></a>
### Настройка индексов моделей

Каждая модель Eloquent синхронизируется с заданным поисковым «индексом», который содержит все доступные для поиска записи для этой модели. Другими словами, вы можете думать о каждом индексе как о таблице MySQL. По умолчанию каждая модель будет сохранена в индексе, соответствующем типичному «табличному» имени модели. Обычно это форма множественного числа от названия модели; однако вы можете настроить индекс, переопределив метод `searchableAs` в модели:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * Переопределение имени индекса модели по умолчанию
     */
    public function searchableAs(): string
    {
        return 'posts_index';
    }
}
```

> [!NOTE]
> Метод `searchableAs` не влияет на движок базы данных, который всегда выполняет поиск непосредственно в таблице модели.

<a name="configuring-the-model-id"></a>
#### Настройка идентификатора модели

По умолчанию Scout будет использовать первичный ключ модели в качестве уникального идентификатора / ключа модели, который хранится в поисковом индексе. Если вам нужно настроить это поведение, вы можете переопределить методы `getScoutKey` и `getScoutKeyName` в модели:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Переопределение значения ключа индекса модели по умолчанию
     */
    public function getScoutKey(): mixed
    {
        return $this->email;
    }

    /**
     * Переопределение имени ключа индекса модели по умолчанию
     */
    public function getScoutKeyName(): mixed
    {
        return 'email';
    }
}
```

> [!NOTE]
> Методы `getScoutKey` и `getScoutKeyName` не влияют на движок базы данных, который всегда использует первичный ключ модели.

<a name="algolia-configuration"></a>
### Algolia

<a name="algolia-index-settings"></a>
#### Настройка параметров индекса (Algolia)

Иногда вам может понадобиться настроить дополнительные параметры индексов Algolia. Хотя вы можете управлять этими параметрами через пользовательский интерфейс Algolia, иногда более эффективно управлять желаемым состоянием конфигурации индекса непосредственно из файла конфигурации `config/scout.php` вашего приложения.

Этот подход позволяет вам развертывать эти настройки через автоматизированный конвейер развертывания вашего приложения, избегая ручной настройки и обеспечивая согласованность в нескольких средах. Вы можете настроить фильтруемые атрибуты, ранжирование, фасетирование или [любые другие поддерживаемые параметры](https://www.algolia.com/doc/rest-api/search/#tag/Indices/operation/setSettings).

Для начала добавьте настройки для каждого индекса в файл конфигурации `config/scout.php` вашего приложения:

```php
use App\Models\User;
use App\Models\Flight;

'algolia' => [
    'id' => env('ALGOLIA_APP_ID', ''),
    'secret' => env('ALGOLIA_SECRET', ''),
    'index-settings' => [
        User::class => [
            'searchableAttributes' => ['id', 'name', 'email'],
            'attributesForFaceting'=> ['filterOnly(email)'],
            // Other settings fields...
        ],
        Flight::class => [
            'searchableAttributes'=> ['id', 'destination'],
        ],
    ],
],
```

Если модель, лежащая в основе данного индекса, является мягко удаляемой и включена в массив `index-settings`, Scout автоматически включит поддержку фасетирования для мягко удаляемых моделей в этом индексе. Если у вас нет других атрибутов фасетирования для определения мягко удаляемого модельного индекса, вы можете просто добавить пустую запись в массив `index-settings` для этой модели:

```php
'index-settings' => [
    Flight::class => []
],
```

После настройки параметров индекса вашего приложения вы должны вызвать команду Artisan `scout:sync-index-settings`. Эта команда сообщит Algolia о ваших текущих настроенных параметрах индекса. Для удобства вы можете сделать эту команду частью процесса развертывания:

```shell
php artisan scout:sync-index-settings
```

<a name="algolia-identifying-users"></a>
#### Идентификация пользователей

Scout также позволяет автоматически идентифицировать пользователей при использовании [Algolia](https://algolia.com). Связывание аутентифицированного пользователя с операциями поиска может быть полезно при просмотре аналитики поиска на панели инструментов Algolia. Вы можете включить идентификацию пользователя, определив для переменной среды `SCOUT_IDENTIFY` значение `true` в файле `.env` вашего приложения:

```ini
SCOUT_IDENTIFY=true
```

Включение этой функции также передаст IP-адрес запроса и основной идентификатор вашего аутентифицированного пользователя в Algolia, поэтому эти данные будут связаны с любым поисковым запросом, сделанным пользователем.

<a name="meilisearch-configuration"></a>
### Meilisearch

<a name="meilisearch-index-settings"></a>
#### Настройка фильтруемых данных и параметров индекса (Meilisearch)

В отличие от других драйверов Scout, Meilisearch требует предварительного определения настроек поиска индекса, таких как фильтруемые атрибуты, сортируемые атрибуты и [другие поддерживаемые поля настроек](https://docs.meilisearch.com/reference/api/settings.html).

Фильтруемые атрибуты - это любые атрибуты, по которым вы планируете фильтровать при вызове метода `where` Scout, в то время как сортируемые атрибуты - это любые атрибуты, по которым вы планируете сортировать при вызове метода `orderBy` Scout. Чтобы определить настройки вашего индекса, отредактируйте раздел `index-settings` в записи `meilisearch` в файле конфигурации `scout` вашего приложения:

```php
use App\Models\User;
use App\Models\Flight;

'meilisearch' => [
    'host' => env('MEILISEARCH_HOST', 'http://localhost:7700'),
    'key' => env('MEILISEARCH_KEY', null),
    'index-settings' => [
        User::class => [
            'filterableAttributes'=> ['id', 'name', 'email'],
            'sortableAttributes' => ['created_at'],
            // Другие поля настроек...
        ],
        Flight::class => [
            'filterableAttributes'=> ['id', 'destination'],
            'sortableAttributes' => ['updated_at'],
        ],
    ],
],
```

Если модель, лежащая в основе определенного индекса, поддерживает мягкое удаление и включена в массив `index-settings`, Scout автоматически включит поддержку фильтрации для мягко удаленных моделей в этом индексе. Если у вас нет других атрибутов, по которым можно фильтровать или сортировать для определения индекса модели с мягким удалением, вы можете просто добавить пустую запись в массив `index-settings` для этой модели:

```php
'index-settings' => [
    Flight::class => []
],
```

После настройки параметров индекса вашего приложения необходимо вызвать команду Artisan `scout:sync-index-settings`. Эта команда сообщит Meilisearch о ваших текущих настройках индекса. Для удобства вы можете включить эту команду в ваш процесс развертывания:

```shell
php artisan scout:sync-index-settings
```

<a name="meilisearch-data-types"></a>
#### Типы поисковых данных

Некоторые поисковые движки, такие как Meilisearch, выполнят операции фильтрации (`>`, `<` и т. д.) только для данных правильного типа. Поэтому, при использовании таких поисковых движков и настройке вашего поискового контента, убедитесь, что числовые значения преобразованы в правильный тип:

```php
public function toSearchableArray()
{
    return [
        'id' => (int) $this->id,
        'name' => $this->name,
        'price' => (float) $this->price,
    ];
}
```

<a name="typesense-configuration"></a>
### Typesense

<a name="typesense-searchable-data"></a>
#### Подготовка поисковых данных

При использовании Typesense ваши модели для поиска должны определить метод `toSearchableArray`, который преобразует основной ключ вашей модели в строку и дату создания в метку времени UNIX:

```php
/**
 * Получить индексируемый массив данных для модели.
 *
 * @return array<string, mixed>
 */
public function toSearchableArray(): array
{
    return array_merge($this->toArray(),[
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
```

Вы также должны определить схемы коллекций Typesense в файле конфигурации вашего приложения `config/scout.php`. Схема коллекции описывает типы данных каждого поля, которые можно искать с помощью Typesense. Для получения дополнительной информации о всех доступных параметрах схемы обратитесь к [документации Typesense](https://typesense.org/docs/latest/api/collections.html#schema-parameters).

Если вам необходимо изменить схему коллекции Typesense после её определения, вы можете либо выполнить команды `scout:flush` и `scout:import`, которые удалят все существующие индексированные данные и создадут схему заново. Либо вы можете использовать API Typesense для изменения схемы коллекции без удаления каких-либо индексированных данных.

Если ваша модель для поиска поддерживает мягкое удаление, вы должны определить поле `__soft_deleted` в схеме соответствующей модели Typesense в файле конфигурации вашего приложения `config/scout.php`.

```php
User::class => [
    'collection-schema' => [
        'fields' => [
            // ...
            [
                'name' => '__soft_deleted',
                'type' => 'int32',
                'optional' => true,
            ],
        ],
    ],
],
```

<a name="typesense-dynamic-search-parameters"></a>
#### Динамические параметры поиска

Typesense позволяет вам динамически изменять [параметры поиска](https://typesense.org/docs/latest/api/search.html#search-parameters) при выполнении операции поиска с помощью метода `options`:

```php
use App\Models\Todo;

Todo::search('Groceries')->options([
    'query_by' => 'title, description'
])->get();
```

<a name="indexing"></a>
## Индексирование сторонними движками

> [!NOTE]
> Описанные в этом разделе возможности индексирования относятся прежде всего к сторонним движкам Algolia, Meilisearch и Typesense. Движок базы данных выполняет поиск непосредственно по таблицам, поэтому не требует ручного управления индексами.

<a name="batch-import"></a>
### Пакетный импорт

Если вы устанавливаете Scout в существующий проект, возможно, у вас уже есть записи базы данных, которые необходимо импортировать в индексы. Scout предоставляет Artisan-команду `scout:import`, которую вы можете использовать для импорта всех существующих записей в поисковые индексы:

```shell
php artisan scout:import "App\Models\Post"
```

Команду `scout:queue-import` можно использовать для импорта всех существующих записей с помощью [очередей заданий](/docs/{{version}}/queues):

```shell
php artisan scout:queue-import "App\Models\Post" --chunk=500
```

Команду `flush` можно использовать для удаления всех записей из поисковых индексов:

```shell
php artisan scout:flush "App\Models\Post"
```

<a name="modifying-the-import-query"></a>
#### Изменение запроса на импорт

Если вы хотите изменить запрос, который используется для получения моделей для пакетного импорта, вы можете определить метод `makeAllSearchableUsing` в модели. Это отличное место для добавления любых отношений, которые могут потребоваться перед импортом:

```php
use Illuminate\Database\Eloquent\Builder;

/**
 * Измените запрос, сделав поиск по всем моделям.
 */
protected function makeAllSearchableUsing(Builder $query): Builder
{
    return $query->with('author');
}
```

> [!WARNING]
> Метод `makeAllSearchableUsing` может оказаться не применимым при использовании очереди для пакетного импорта моделей. Связи [не восстанавливаются](/docs/{{version}}/queues#handling-relationships) при обработке коллекций моделей в заданиях.

<a name="adding-records"></a>
### Добавление записей

После того как вы добавили в модель трейт `Laravel\Scout\Searchable`, всё, что вам нужно сделать, это вызвать метод `save` или `create` на экземпляре модели, и она будет автоматически добавлена в поисковый индекс. Если вы настроили Scout для [использования очередей](#queueing), эта операция будет выполняться в фоновом режиме обработчиком очереди:

```php
use App\Models\Order;

$order = new Order;

// ...

$order->save();
```

<a name="adding-records-via-query"></a>
#### Добавление записей через запрос

Если вы хотите добавить коллекцию моделей в поисковый индекс с помощью запроса Eloquent, вы можете связать метод `searchable` с запросом Eloquent. Метод `searchable` [разделит результаты](/docs/{{version}}/eloquent#chunking-results) запроса и добавит блоки в поисковый индекс. Опять же, если вы настроили Scout для использования очередей, все блоки будут импортированы в фоновом режиме обработчиком очереди:

```php
use App\Models\Order;

Order::where('price', '>', 100)->searchable();
```

Вы также можете вызвать метод `searchable` для экземпляра коллекции Eloquent:

```php
$user->orders()->searchable();
```

Или, если у вас уже есть коллекция Eloquent, вы можете вызвать метод `searchable` для коллекции, чтобы добавить экземпляры моделей в их соответствующий индекс:

```php
$orders->searchable();
```

> [!NOTE]
> Метод `searchable` можно считать операцией "upsert". Другими словами, если запись модели уже есть в поисковом индексе, то она будет обновлена. Если записи нет, она будет добавлена в индекс.

<a name="updating-records"></a>
### Обновление записей

Чтобы обновить поисковый индекс модели, вам нужно только обновить свойства экземпляра модели и вызвать метод `save` для сохранения в базе данных. Scout автоматически сохранит изменения в поисковом индексе:

```php
use App\Models\Order;

$order = Order::find(1);

// Обновляем заказ...

$order->save();
```

Вы также можете вызвать метод searchable` в экземпляре запроса Eloquent, чтобы обновить коллекцию моделей. Если моделей нет в поисковом индексе, они будут созданы:

```php
Order::where('price', '>', 100)->searchable();
```

Если вы хотите обновить записи поискового индекса для всех моделей в коллекции, вы можете вызвать метод `searchable` в цепочке вызова:

```php
$user->orders()->searchable();
```

Или, если у вас уже есть коллекция Eloquent, вы можете вызвать метод `searchable` для коллекции, чтобы добавить экземпляры моделей в их соответствующий индекс:

```php
$orders->searchable();
```

<a name="modifying-records-before-importing"></a>
#### Изменение Записей Перед Импортом

Иногда вам может потребоваться подготовить коллекцию моделей перед тем, как они станут доступны для поиска. Например, вы можете захотеть предварительно загрузить отношение, чтобы данные они могли быть эффективно добавлены в ваш индекс поиска. Для достижения этой цели определите метод `makeSearchableUsing` в соответствующей модели:

```php
use Illuminate\Database\Eloquent\Collection;

/**
 * Измените коллекцию моделей, которые будут доступны для поиска.
 */
public function makeSearchableUsing(Collection $models): Collection
{
    return $models->load('author');
}
```

<a name="conditionally-updating-the-search-index"></a>
#### Условное обновление поискового индекса

По умолчанию Scout переиндексирует обновленную модель независимо от того, какие атрибуты были изменены. Если вы хотите настроить это поведение, определите метод `searchIndexShouldBeUpdated` в модели:

```php
/**
 * Определить, должен ли быть обновлен поисковый индекс.
 */
public function searchIndexShouldBeUpdated(): bool
{
    return $this->wasRecentlyCreated || $this->wasChanged(['title', 'body']);
}
```

<a name="removing-records"></a>
### Удаление записей

Чтобы удалить запись из поискового индекса, вы можете вызвать метод `delete` на экземпляре модели для удаления модели из базы данных. Это можно сделать, даже если вы используете [псевдоудаление](/docs/{{version}}/eloquent#soft-deleting) модели:

```php
use App\Models\Order;

$order = Order::find(1);

$order->delete();
```

Если вы не хотите извлекать модель перед удалением записи, вы можете использовать метод `unsearchable` для экземпляра запроса Eloquent:

```php
Order::where('price', '>', 100)->unsearchable();
```

Если вы хотите удалить записи поискового индекса для всех моделей в коллекции, вы можете вызвать метод `unsearchable` на цепочке вызова:

```php
$user->orders()->unsearchable();
```

Или, если у вас уже есть коллекция Eloquent, вы можете вызвать метод `unsearchable` для коллекции, чтобы удалить экземпляры моделей из индекса:

```php
$orders->unsearchable();
```

Чтобы удалить все записи модели из соответствующего индекса, вы можете вызвать метод `removeAllFromSearch`:

```php
Order::removeAllFromSearch();
```

<a name="pausing-indexing"></a>
### Приостановка индексации

Иногда вам может потребоваться выполнить некоторые операции Eloquent с моделью без синхронизации данных модели с поисковым индексом. Вы можете сделать это, используя метод `withoutSyncingToSearch`. Этот метод принимает замыкание, которое будет немедленно выполнено. Любые операции модели, которые происходят внутри замыкания, не будут синхронизироваться с индексом модели:

```php
use App\Models\Order;

Order::withoutSyncingToSearch(function () {
    // Выполняем действия модели...
});
```

<a name="conditionally-searchable-model-instances"></a>
### Экземпляры моделей с условным поиском

Иногда может потребоваться сделать модель доступной для поиска только при определенных условиях. Например, представьте, что у вас есть модель `App\Models\Post`, которая может находиться в одном из двух состояний: «черновик» и «опубликована». Вы можете добавлять в поисковый индекс только «опубликованные» сообщения. Для этого необходимо определить в модели метод `shouldBeSearchable`:

```php
/**
 * Определите, когда модель должна быть доступной для поиска.
 */
public function shouldBeSearchable(): bool
{
    return $this->isPublished();
}
```

Метод `shouldBeSearchable` применяется только при манипуляциях с моделями с помощью методов `save`, `create`, запросов или в цепочке вызовов. Непосредственное добавление моделей или коллекций в поисковый индекс с помощью метода `searchable` переопределит результат метода `shouldBeSearchable`.

> [!WARNING]
> Метод `shouldBeSearchable` не применим при использовании драйвера "database" в Scout, так как все данные для поиска всегда хранятся в базе данных. Для достижения аналогичного поведения при использовании драйвера базы данных следует использовать [Условия Where](#where-clauses) вместо этого.

<a name="searching"></a>
## Поиск

Вы можете выполнить поиск по модели, используя метод `search`. Метод `search` принимает строку в качестве поискового запроса, которая будет использоваться для поиска по модели. Затем вы должны вызвать метод `get`, чтобы получить модель Eloquent в качестве результата заданного поискового запроса:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->get();
```

Поскольку поисковые запросы Scout возвращают коллекцию моделей Eloquent, вы можете возвращать результаты непосредственно из маршрута или контроллера, и они будут автоматически преобразованы в JSON:

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/search', function (Request $request) {
    return Order::search($request->search)->get();
});
```

Если вы хотите получить необработанные результаты поиска до того, как они будут преобразованы в модели Eloquent, вы можете использовать метод `raw`:

```php
$orders = Order::search('Star Trek')->raw();
```

<a name="custom-indexes"></a>
#### Пользовательский индекс

Поисковые запросы обычно выполняются по индексу, указанному в методе [searchchableAs](#configuring-model-indexes) модели. Однако вы можете использовать метод `within`, чтобы указать индекс, который следует использовать вместо этого:

```php
$orders = Order::search('Star Trek')
    ->within('tv_shows_popularity_desc')
    ->get();
```

<a name="where-clauses"></a>
### Условия Where

Scout позволяет добавлять условия `where` к поисковым запросам. Например, базовые проверки равенства полезны для ограничения поисковых запросов по идентификатору владельца:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->where('user_id', 1)->get();
```

Вы также можете использовать операторы сравнения `=`, `!=`, `<`, `>`, `>=`, `<=` для построения более сложных запросов:

```php
Order::search('Star Trek')
  ->where('status', '=', 'completed')
  ->where('is_refunded', '!=', true)
  ->where('total_price', '>', 100)
  ->where('shipping_cost', '<', 20)
  ->where('discount_percent', '>=', 10)
  ->where('item_count', '<=', 5)
  ->get();
```

Кроме того, метод `whereIn` может быть использован для проверки того, содержится ли значение заданного столбца в указанном массиве:

```php
$orders = Order::search('Star Trek')->whereIn(
    'status', ['open', 'paid']
)->get();
```

Метод `whereNotIn` проверяет, что значение заданного столбца не содержится в указанном массиве:

```php
$orders = Order::search('Star Trek')->whereNotIn(
    'status', ['closed']
)->get();
```

> [!WARNING]
> Если ваше приложение использует Meilisearch, вам необходимо настроить [фильтруемые атрибуты](#meilisearch-index-settings) перед использованием условий `where` Scout.

<a name="customizing-the-eloquent-results-query"></a>
#### Настройка Запроса Результатов Eloquent

После того, как Scout получает список соответствующих моделей Eloquent из поискового движка вашего приложения, для извлечения всех соответствующих моделей по их первичным ключам используется Eloquent. Вы можете настроить этот запрос, вызвав метод `query`. Метод `query` принимает замыкание, которое получит экземпляр построителя запросов Eloquent в качестве аргумента:

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

При использовании стороннего движка эта callback-функция вызывается после того, как соответствующие модели уже были получены из поискового движка, поэтому ее не следует использовать для «фильтрации» результатов - используйте [условия where Scout](#where-clauses). Однако при использовании движка базы данных ограничения метода `query` применяются непосредственно к запросу базы данных, поэтому его можно использовать и для фильтрации.

<a name="pagination"></a>
### Постраничная разбивка данных (Pagination) (Пагинация)

Помимо получения коллекции моделей, вы можете разбить результаты поиска на страницы, используя метод `paginate`. Этот метод вернет экземпляр `Illuminate\Pagination\LengthAwarePaginator`, как если бы вы [разбили на страницы](/docs/{{version}}/pagination) обычный запрос Eloquent:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->paginate();
```

Вы можете указать, сколько моделей извлекать на странице, передав количество в качестве аргумента методу `paginate`:

```php
$orders = Order::search('Star Trek')->paginate(15);
```

При использовании движка базы данных вы также можете использовать метод `simplePaginate`. В отличие от `paginate`, который получает общее количество совпадающих записей для отображения номеров страниц, `simplePaginate` только определяет, есть ли результаты на следующей странице, что эффективнее для больших наборов данных:

```php
$orders = Order::search('Star Trek')->simplePaginate(15);
```

Получив результаты, вы можете отобразить результаты и отобразить ссылки на страницы с помощью [Blade](/docs/{{version}}/blade), как если бы вы разбили на страницы обычный запрос Eloquent:

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

Конечно, если вы хотите получить результаты разбиения на страницы в виде JSON, вы можете вернуть экземпляр пагинатора прямо из маршрута или контроллера:

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    return Order::search($request->input('query'))->paginate(15);
});
```

> [!WARNING]
> Поскольку поисковые движки не осведомлены о глобальных определениях области видимости вашей Eloquent-модели, вы не должны использовать глобальные области видимости в приложениях, которые используют пагинацию Scout. Или же вы должны воссоздать ограничения глобальной области видимости при поиске через Scout.

<a name="soft-deleting"></a>
### Псевдоудаление

Если ваши проиндексированные модели [псевдоудалены](/docs/{{version}}/eloquent#soft-deleting) и вам нужно выполнить поиск по своим псевдоудаленным моделям, установите параметр `soft_delete` в файле `config/scout.php` на `true`:

```php
'soft_delete' => true,
```

Когда этот параметр имеет значение `true`, Scout не будет удалять псевдоудаленные модели из поискового индекса. Вместо этого он установит скрытый атрибут `__soft_deleted` для проиндексированной записи. Затем вы можете использовать методы `withTrashed` или `onlyTrashed` для получения псевдоудаленных записей при поиске:

```php
use App\Models\Order;

// Использовать удаленные записи при получении результатов...
$orders = Order::search('Star Trek')->withTrashed()->get();

// Использовать только удаленные записи при получении результатов...
$orders = Order::search('Star Trek')->onlyTrashed()->get();
```

> [!NOTE]
> Когда псевдоудаленная модель будет окончательно удалена с помощью `forceDelete`, Scout автоматически удалит ее из поискового индекса.

<a name="customizing-engine-searches"></a>
### Настройка поискового движка

Если вам нужно выполнить расширенную настройку поведения поискового движка, вы можете передать замыкание в качестве второго аргумента методу `search`. Например, вы можете использовать замыкание, чтобы добавить данные о геолокации в параметры поиска до того, как поисковый запрос будет передан в Algolia:

```php
use Algolia\AlgoliaSearch\SearchIndex;
use App\Models\Order;

Order::search(
    'Star Trek',
    function (SearchIndex $algolia, string $query, array $options) {
        $options['body']['query']['bool']['filter']['geo_distance'] = [
            'distance' => '1000km',
            'location' => ['lat' => 36, 'lon' => 111],
        ];

        return $algolia->search($query, $options);
    }
)->get();
```

<a name="custom-engines"></a>
## Разработка поискового движка

<a name="writing-the-engine"></a>
#### Реализация своего поискового механизма

Если одна из встроенных поисковых систем Scout не соответствует вашим потребностям, вы можете написать свой собственный поисковый механизм (поисковый движок) и зарегистрировать его в Scout. Ваш движок должен расширять абстрактный класс `Laravel\Scout\Engines\Engine`. Этот класс содержит восемь методов, которые должен реализовать ваш движок:

```php
use Laravel\Scout\Builder;

abstract public function update($models);
abstract public function delete($models);
abstract public function search(Builder $builder);
abstract public function paginate(Builder $builder, $perPage, $page);
abstract public function mapIds($results);
abstract public function map(Builder $builder, $results, $model);
abstract public function getTotalCount($results);
abstract public function flush($model);
```

Возможно, вам будет полезно просмотреть реализации этих методов в классе `Laravel\Scout\Engines\AlgoliaEngine`. Этот класс предоставит вам хорошую отправную точку для изучения того, как реализовать каждый из этих методов в вашем собственном движке.

<a name="registering-the-engine"></a>
#### Регистрация поискового движка

После того как вы написали свой собственный движок, вы можете зарегистрировать его в Scout, используя метод `extend` менеджера Scout. Диспетчер Scout может быть определён из служебного контейнера Laravel. Вы должны вызвать метод `extend` из метода `boot` вашего класса `App\Providers\AppServiceProvider` или любого другого провайдера, используемого вашим приложением:

```php
use App\ScoutExtensions\MySqlSearchEngine;
use Laravel\Scout\EngineManager;

/**
 * Загрузка сервисов приложения.
 */
public function boot(): void
{
    resolve(EngineManager::class)->extend('mysql', function () {
        return new MySqlSearchEngine;
    });
}
```

После регистрации движка вы можете указать его в качестве `driver` по умолчанию в файле конфигурации `config/scout.php`:

```php
'driver' => 'mysql',
```
