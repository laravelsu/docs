---
git: 16384f6f8f408ec84934623a73e4a469f232e8b0
---

# Тестирование · База данных

<a name="introduction"></a>
## Введение

Laravel предлагает множество полезных инструментов, чтобы упростить тестирование приложений, использующих базу данных. Фабрики моделей (factory) и наполнители (seeders) позволяют безболезненно создавать записи тестовой базы данных с использованием моделей и отношений Eloquent вашего приложения. Мы обсудим все эти мощные функции в текущей документации.

<a name="resetting-the-database-after-each-test"></a>
### Сброс базы данных после каждого теста

Прежде чем продолжить, давайте обсудим, как сбрасывать вашу базу данных после каждого из ваших тестов, чтобы данные из предыдущего теста не мешали последующим тестам. Включенный в Laravel трейт `Illuminate\Foundation\Testing\RefreshDatabase` позаботится об этом за вас. Просто используйте трейт в своем тестовом классе:

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('basic example', function () {
    $response = $this->get('/');

    // ...
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A basic functional test example.
     */
    public function test_basic_example(): void
    {
        $response = $this->get('/');

        // ...
    }
}
```

Трейт `Illuminate\Foundation\Testing\RefreshDatabase` не мигрирует вашу базу данных, если ваша схема актуальна. Вместо этого он выполняет тест в пределах транзакции базы данных. Следовательно, любые записи, добавленные в базу данных в тестах, не использующих этот трейт, могут по-прежнему существовать в базе данных.

Если вы хотите полностью сбросить базу данных, вместо этого вы можете использовать трейты `Illuminate\Foundation\Testing\DatabaseMigrations` или `Illuminate\Foundation\Testing\DatabaseTruncation`. Однако обе эти опции значительно медленнее, чем трейт `RefreshDatabase`.

<a name="model-factories"></a>
## Фабрики моделей

При тестировании может возникнуть необходимость вставить несколько записей в вашу базу данных перед выполнением теста. Вместо ручного указания значения каждого столбца при создании тестовых данных Laravel позволяет вам определить набор атрибутов по умолчанию для каждой [модели Eloquent](/docs/{{version}}/eloquent), используя [фабрики моделей](/docs/{{version}}/eloquent-factories).

Для более подробной информации о создании и использовании фабрик моделей для создания моделей обратитесь к полной [документации по фабрикам моделей](/docs/{{version}}/eloquent-factories). После того, как вы определили фабрику модели, вы можете использовать ее внутри вашего теста для создания моделей:

```php tab=Pest
use App\Models\User;

test('models can be instantiated', function () {
    $user = User::factory()->create();

    // ...
});
```

```php tab=PHPUnit
use App\Models\User;

public function test_models_can_be_instantiated(): void
{
    $user = User::factory()->create();

    // ...
}
```

<a name="running-seeders"></a>
## Запуск наполнителей (seed, seeders)

Если вы хотите использовать [наполнители базы данных](/docs/{{version}}/seeding) для наполнения вашей базы данных во время функционального тестирования, то вы можете вызвать метод `seed`. По умолчанию метод `seed` будет запускать `DatabaseSeeder`, который должен запускать все другие ваши наполнители. Как вариант, вы можете передать конкретное имя класса-наполнителя методу `seed`:

```php tab=Pest
<?php

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('orders can be created', function () {
    // Run the DatabaseSeeder...
    $this->seed();

    // Run a specific seeder...
    $this->seed(OrderStatusSeeder::class);

    // ...

    // Run an array of specific seeders...
    $this->seed([
        OrderStatusSeeder::class,
        TransactionStatusSeeder::class,
        // ...
    ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Test creating a new order.
     */
    public function test_orders_can_be_created(): void
    {
        // Run the DatabaseSeeder...
        $this->seed();

        // Run a specific seeder...
        $this->seed(OrderStatusSeeder::class);

        // ...

        // Run an array of specific seeders...
        $this->seed([
            OrderStatusSeeder::class,
            TransactionStatusSeeder::class,
            // ...
        ]);
    }
}
```

В качестве альтернативы, вы можете указать Laravel автоматически заполнять базу данных перед каждым тестом, который использует трейт `RefreshDatabase`. Вы можете добиться этого, добавив атрибут `Seed` к вашему базовому тестовому классу:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\Attributes\Seed;
use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

#[Seed]
abstract class TestCase extends BaseTestCase
{
}
```

Когда присутствует атрибут `Seed`, класс `Database\Seeders\DatabaseSeeder` будет запускаться перед каждым тестом, который использует трейт `RefreshDatabase`. Однако, вы можете указать конкретный наполнитель, который должен выполняться, используя атрибут `Seeder` в вашем тестовом классе:

```php
<?php

namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Illuminate\Foundation\Testing\Attributes\Seeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

#[Seeder(OrderStatusSeeder::class)]
class OrderTest extends TestCase
{
    use RefreshDatabase;

    // ...
}
```

<a name="available-assertions"></a>
## Доступные утверждения

Laravel содержит несколько утверждений базы данных для ваших функциональных тестов [Pest](https://pestphp.com) или [PHPUnit](https://phpunit.de). Мы обсудим каждое из этих утверждений ниже.

<a name="assert-database-count"></a>
#### assertDatabaseCount

Утверждает, что таблица в базе данных содержит указанное количество записей:

```php
$this->assertDatabaseCount('users', 5);
```

<a name="assert-database-empty"></a>
#### assertDatabaseEmpty

Утверждает, что таблица в базе данных не содержит записей:

```php
$this->assertDatabaseEmpty('users');
```

<a name="assert-database-has"></a>
#### assertDatabaseHas

Утверждает, что таблица в базе данных содержит записи, соответствующие переданным ключ / значение ограничениям запроса:

```php
$this->assertDatabaseHas('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-database-missing"></a>
#### assertDatabaseMissing

Утверждает, что таблица в базе данных не содержит записей, соответствующих переданным ключ / значение ограничениям запроса:

```php
$this->assertDatabaseMissing('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-deleted"></a>
#### assertSoftDeleted

Метод `assertSoftDeleted` используется для утверждения того, что переданная модель Eloquent была «программно удалена»:

```php
$this->assertSoftDeleted($user);
```

<a name="assert-not-deleted"></a>
#### assertNotSoftDeleted

Метод `assertNotSoftDeleted` используется для утверждения того, что переданная модель Eloquent была «программно удалена»

```php
$this->assertNotSoftDeleted($user);
```

<a name="assert-model-exists"></a>
#### assertModelExists

Утверждает, что данная модель или коллекция моделей существует в базе данных:

```php
use App\Models\User;

$user = User::factory()->create();

$this->assertModelExists($user);
```

<a name="assert-model-missing"></a>
#### assertModelMissing

Утверждает, что данной модели или коллекции моделей не существует в базе данных:

```php
use App\Models\User;

$user = User::factory()->create();

$user->delete();

$this->assertModelMissing($user);
```

<a name="expects-database-query-count"></a>
#### expectsDatabaseQueryCount

Метод `expectsDatabaseQueryCount` может быть вызван в начале вашего теста для указания общего числа запросов к базе данных, которые вы ожидаете во время выполнения теста. Если фактическое количество выполненных запросов не соответствует ожиданиям, тест завершится неудачей:

```php
$this->expectsDatabaseQueryCount(5);

// Test...
```
