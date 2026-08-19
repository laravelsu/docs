---
git: 4030a84b979f7420788dda14df439dd4d66d765d
---

# Работа с изображениями

- [Введение](#introduction)
- [Установка](#installation)
    - [Конфигурирование](#configuration)
- [Чтение изображений](#reading-images)
    - [Загруженные файлы](#uploaded-files)
    - [Файлы из хранилища](#storage-files)
    - [Другие источники](#other-sources)
- [Изменение изображений](#manipulating-images)
    - [Изменение размера изображений](#resizing-images)
    - [Другие преобразования](#other-transformations)
- [Кодирование изображений](#encoding-images)
- [Сохранение изображений](#storing-images)
- [Получение сведений об изображениях](#inspecting-images)
- [Драйверы изображений](#image-drivers)
    - [Пользовательские драйверы изображений](#custom-image-drivers)
    - [Пользовательские преобразования](#custom-transformations)

<a name="introduction"></a>
## Введение

Laravel предоставляет выразительный API для работы с изображениями, который позволяет изменять размер, обрезать, кодировать и сохранять изображения, используя те же удобные соглашения, что и во всем фреймворке. Возможности Laravel для работы с изображениями основаны на [Intervention Image](https://image.intervention.io/) и поддерживают PHP-расширения GD и Imagick.

API изображений полезен при работе с загруженными файлами, файлами, сохраненными на [дисках файловой системы](/docs/{{version}}/filesystem) Laravel, локальными файлами, удаленными URL или необработанными байтами изображения:

```php
use Illuminate\Support\Facades\Image;

$path = Image::fromStorage('avatars/photo.jpg', 'public')
    ->cover(400, 400)
    ->toWebp()
    ->quality(80)
    ->storePublicly('avatars', 'public');
```

> [!WARNING]
> Работа с изображениями может активно использовать CPU и память. Для обработки больших изображений рассмотрите выполнение работы в [задании очереди](/docs/{{version}}/queues), а не во время HTTP-запроса, который принимает загрузку.

<a name="installation"></a>
## Установка

Перед использованием возможностей Laravel для работы с изображениями установите пакет Intervention Image через Composer:

```shell
composer require intervention/image:^4.0
```

Также убедитесь, что в вашей установке PHP установлено расширение GD или Imagick, в зависимости от того, какой драйвер будет использовать приложение.

<a name="configuration"></a>
### Конфигурирование

Файл конфигурации изображений Laravel находится по адресу `config/images.php`. Если в вашем приложении нет файла конфигурации `images`, вы можете опубликовать его с помощью Artisan-команды `config:publish`:

```shell
php artisan config:publish images
```

Файл конфигурации изображений позволяет указать драйвер изображений по умолчанию для приложения. Также драйвер по умолчанию можно указать с помощью переменной окружения `IMAGE_DRIVER`. Поддерживаемые драйверы: `gd` и `imagick`:

```ini
IMAGE_DRIVER=imagick
```

<a name="reading-images"></a>
## Чтение изображений

Фасад `Image` предоставляет несколько методов для чтения изображений из распространенных источников. Содержимое изображения загружается лениво, поэтому источник обычно не читается до обработки изображения или запроса его байтов.

<a name="uploaded-files"></a>
### Загруженные файлы

Вы можете получить загруженное изображение из входящего запроса с помощью метода `image`. Этот метод возвращает экземпляр `Illuminate\Image\Image` для загруженного файла или `null`, если файл отсутствует:

```php
use Illuminate\Http\Request;

Route::post('/avatar', function (Request $request) {
    $request->validate(['avatar' => ['required', 'image']]);

    $path = $request->image('avatar')
        ->cover(400, 400)
        ->toWebp()
        ->storePublicly('avatars', 'public');

    // ...
});
```

Альтернативно, вы можете создать экземпляр изображения из экземпляра `Illuminate\Http\UploadedFile` с помощью метода `fromUpload`:

```php
use Illuminate\Support\Facades\Image;

$image = Image::fromUpload($request->file('avatar'));
```

Когда изображение создано из загруженного файла, базовый загруженный файл можно получить с помощью метода `file`:

```php
$file = $image->file();
```

<a name="storage-files"></a>
### Файлы из хранилища

Вы можете создать экземпляр изображения из файла, сохраненного на одном из [дисков файловой системы](/docs/{{version}}/filesystem) приложения, с помощью метода `fromStorage`. Первый аргумент - путь к файлу, второй аргумент - имя диска:

```php
use Illuminate\Support\Facades\Image;

$image = Image::fromStorage('avatars/photo.jpg', disk: 'public');
```

Также можно создавать экземпляры изображений напрямую из экземпляра диска файловой системы с помощью метода `image`:

```php
use Illuminate\Support\Facades\Storage;

$image = Storage::disk('public')->image('avatars/photo.jpg');
```

<a name="other-sources"></a>
### Другие источники

Фасад `Image` также содержит методы для создания экземпляров изображений из необработанных байтов, локальных путей к файлам, удаленных URL и строк, закодированных в Base64:

```php
use Illuminate\Support\Facades\Image;

$image = Image::fromBytes($contents);
$image = Image::fromBase64($base64);
$image = Image::fromPath(storage_path('app/avatars/photo.jpg'));
$image = Image::fromUrl('https://example.com/photo.jpg');
```

<a name="manipulating-images"></a>
## Изменение изображений

Экземпляры изображений неизменяемы. Каждый метод изменения возвращает новый экземпляр изображения с добавленным преобразованием в конвейер обработки, что позволяет свободно выстраивать цепочки методов:

```php
$image = $request->image('avatar')
    ->orient()
    ->cover(400, 400)
    ->sharpen(10);
```

Преобразования выполняются в порядке их добавления в конвейер изображения, а кодирование изображения выполняется только один раз в конце.

<a name="resizing-images"></a>
### Изменение размера изображений

Метод `resize` изменяет размер изображения до указанных размеров. Вы можете передать ширину и высоту или только одно измерение с помощью именованных аргументов:

```php
$image = $image->resize(800, 600);
$image = $image->resize(width: 800);
$image = $image->resize(height: 600);
```

Метод `scale` пропорционально уменьшает изображение так, чтобы оно поместилось в указанные размеры. Этот метод никогда не увеличивает размер изображения:

```php
$image = $image->scale(800, 600);
$image = $image->scale(width: 800);
$image = $image->scale(height: 600);
```

Метод `cover` изменяет размер и обрезает изображение так, чтобы оно полностью покрывало указанные размеры:

```php
$image = $image->cover(400, 400);
```

Метод `contain` изменяет размер изображения так, чтобы оно поместилось в указанные размеры с сохранением всего изображения. При необходимости пустое пространство будет заполнено необязательным фоновым цветом:

```php
$image = $image->contain(400, 400);
$image = $image->contain(400, 400, '#ffffff');
$image = $image->contain(400, 400, 'dominant');
```

Вы можете указать `dominant` в качестве фонового цвета, чтобы заполнить пустое пространство доминирующим цветом изображения.

Вы можете обрезать изображение с помощью метода `crop`. Первые два аргумента - желаемые ширина и высота, а необязательные третий и четвертый аргументы задают координаты `x` и `y` для обрезки:

```php
$image = $image->crop(300, 200);
$image = $image->crop(300, 200, x: 50, y: 25);
```

<a name="other-transformations"></a>
### Другие преобразования

Laravel также предоставляет множество дополнительных методов преобразования изображений:

```php
$image = $image->orient();
$image = $image->rotate(90);
$image = $image->rotate(90, '#ffffff');
$image = $image->rotate(90, 'dominant');
$image = $image->blur(5);
$image = $image->grayscale();
$image = $image->sharpen(10);
$image = $image->flipVertically();
$image = $image->flipHorizontally();
```

Метод `orient` поворачивает изображение согласно данным ориентации EXIF. Метод `rotate` поворачивает изображение по часовой стрелке на указанный угол и принимает необязательный фоновый цвет. Методы `blur` и `sharpen` принимают значения от `0` до `100`.

<a name="conditional-transformations"></a>
#### Условные преобразования

Экземпляры изображений поддерживают трейт Laravel `Conditionable`, что позволяет условно применять преобразования с помощью методов `when` и `unless`:

```php
$image = $request->image('avatar')
    ->when($request->boolean('crop'), fn ($image) => $image->cover(400, 400))
    ->unless($request->boolean('preserve_format'), fn ($image) => $image->toWebp());
```

<a name="encoding-images"></a>
## Кодирование изображений

По умолчанию обработанные изображения кодируются в исходном формате. Однако перед получением или сохранением изображения вы можете преобразовать его в другой поддерживаемый формат:

```php
$image = $image->toWebp();
$image = $image->toJpg();
$image = $image->toJpeg();
$image = $image->toPng();
$image = $image->toGif();
$image = $image->toAvif();
$image = $image->toBmp();
```

Метод `quality` позволяет задать качество вывода. Значение качества будет ограничено диапазоном от `1` до `100`:

```php
$image = $image->toWebp()->quality(80);
```

Метод `optimize` - удобное сокращение для преобразования изображения в указанный формат и установки его качества. По умолчанию изображения оптимизируются как WebP с качеством `70`:

```php
$image = $image->optimize();

$image = $image->optimize(format: 'jpg', quality: 85);
```

Вы можете получить содержимое обработанного изображения как строку байтов, строку в Base64 или data URI:

```php
$bytes = $image->toBytes();
$base64 = $image->toBase64();
$dataUri = $image->toDataUri();
```

Экземпляр изображения также можно привести к строке, чтобы получить data URI:

```php
$dataUri = (string) $image;
```

<a name="storing-images"></a>
## Сохранение изображений

Метод `store` сохраняет обработанное изображение на одном из дисков файловой системы вашего приложения. Как и для загруженных файлов, Laravel сгенерирует уникальное имя файла и вернет сохраненный путь. Второй аргумент можно использовать для указания диска:

```php
$path = $request->image('avatar')
    ->cover(400, 400)
    ->store(path: 'avatars');

$path = $request->image('avatar')
    ->cover(400, 400)
    ->store(path: 'avatars', disk: 's3');
```

Вы можете использовать метод `storeAs`, чтобы указать имя сохраняемого файла:

```php
$path = $request->image('avatar')
    ->cover(400, 400)
    ->storeAs(path: 'avatars', name: 'avatar.jpg', disk: 'public');
```

Методы `storePublicly` и `storePubliclyAs` сохраняют изображение с видимостью `public`:

```php
$path = $request->image('avatar')
    ->cover(400, 400)
    ->storePublicly(path: 'avatars', disk: 'public');

$path = $request->image('avatar')
    ->cover(400, 400)
    ->storePubliclyAs(path: 'avatars', name: 'avatar.webp', disk: 'public');
```

Если изображение не удалось сохранить, методы сохранения вернут `false`.

<a name="inspecting-images"></a>
## Получение сведений об изображениях

Вы можете получить MIME-тип, расширение, размеры, ширину, высоту и доминирующий цвет изображения с помощью следующих методов:

```php
$mimeType = $image->mimeType();
$extension = $image->extension();

[$width, $height] = $image->dimensions();
$width = $image->width();
$height = $image->height();

$dominantColor = $image->dominantColor();
```

Эти методы работают с обработанным изображением. Например, вызов `width` после `cover(400, 400)` вернет `400`.

<a name="image-drivers"></a>
## Драйверы изображений

<a name="custom-image-drivers"></a>
### Пользовательские драйверы изображений

Менеджер изображений Laravel расширяет базовый класс `Illuminate\Support\Manager` Laravel. Это означает, что вы можете регистрировать пользовательские драйверы изображений с помощью метода `extend`, доступного в менеджере изображений и фасаде `Image`.

Пользовательские драйверы изображений должны реализовывать интерфейс `Illuminate\Contracts\Image\Driver`. Метод `process` получает исходное содержимое изображения и упорядоченный `Illuminate\Image\ImagePipeline`, который должен быть применен к изображению, и должен вернуть байты обработанного изображения:

```php
<?php

namespace App\Images;

use Illuminate\Contracts\Image\Driver;
use Illuminate\Image\ImagePipeline;

class VipsDriver implements Driver
{
    /**
     * Process the given image contents with the specified pipeline.
     */
    public function process(string $contents, ImagePipeline $pipeline): string
    {
        // Apply the pipeline's transformations and output options...

        return $contents;
    }

    /**
     * Register a transformation handler.
     */
    public function transformUsing(string $transformation, callable $callback): static
    {
        // Store the handler so it may be applied while processing the pipeline...

        return $this;
    }
}
```

> [!NOTE]
> Чтобы лучше понять, как реализовать пользовательский драйвер изображений, вы можете изучить встроенный класс фреймворка `Illuminate\Image\Drivers\InterventionDriver`.

После реализации пользовательского драйвера его можно зарегистрировать с помощью метода `extend` фасада `Image`. Обычно это следует делать в методе `boot` сервис-провайдера:

```php
use App\Images\VipsDriver;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Image;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Image::extend('vips', function (Application $app) {
        return new VipsDriver;
    });
}
```

После регистрации драйвера вы можете использовать его для конкретного изображения с помощью метода `using`:

```php
$image = $request->image('avatar')
    ->using('vips')
    ->cover(400, 400);
```

Также можно настроить пользовательский драйвер как драйвер изображений по умолчанию для приложения с помощью опции `default` в файле конфигурации `config/images.php` или переменной окружения `IMAGE_DRIVER`:

```ini
IMAGE_DRIVER=vips
```

<a name="custom-transformations"></a>
### Пользовательские преобразования

Приложения и пакеты могут определять пользовательские преобразования, создавая класс, который реализует контракт `Illuminate\Contracts\Image\Transformation`. Затем пользовательские преобразования можно добавить в конвейер изображения с помощью метода `transform`:

```php
<?php

namespace App\Images\Transformations;

use Illuminate\Contracts\Image\Transformation;

class Pixelate implements Transformation
{
    public function __construct(
        public readonly int $size,
    ) {
        //
    }
}
```

Затем зарегистрируйте обработчик для преобразования и драйвера с помощью метода `transformUsing` фасада `Image`. Обычно это следует делать в методе `boot` сервис-провайдера:

```php
use App\Images\Transformations\Pixelate;
use Illuminate\Support\Facades\Image;
use Intervention\Image\Interfaces\ImageInterface;

Image::transformUsing('gd', Pixelate::class, function (ImageInterface $image, Pixelate $transformation) {
    return $image->pixelate($transformation->size);
});
```

После регистрации обработчика преобразования вы можете применить преобразование к изображению:

```php
use App\Images\Transformations\Pixelate;

$image = $request->image('avatar')
    ->transform(new Pixelate(12))
    ->store('avatars');
```
