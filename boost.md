---
git: 9a047b04377b9e9fca43800eeb78c386f032c4cd
---

# Laravel Boost

- [Введение](#introduction)
- [Установка](#installation)
    - [Настройка агентов](#set-up-your-agents)
    - [Поддержание ресурсов Boost в актуальном состоянии](#keeping-boost-resources-updated)
- [MCP-сервер](#mcp-server)
    - [Доступные MCP-инструменты](#available-mcp-tools)
    - [Ручная регистрация MCP-сервера](#manually-registering-the-mcp-server)
- [AI-рекомендации](#ai-guidelines)
    - [Доступные AI-рекомендации](#available-ai-guidelines)
    - [Добавление собственных AI-рекомендаций](#adding-custom-ai-guidelines)
    - [Переопределение AI-рекомендаций Boost](#overriding-boost-ai-guidelines)
    - [AI-рекомендации сторонних пакетов](#third-party-package-ai-guidelines)
- [Agent Skills](#agent-skills)
    - [Доступные skills](#available-skills)
    - [Собственные skills](#custom-skills)
    - [Переопределение skills](#overriding-skills)
    - [Skills сторонних пакетов](#third-party-package-skills)
- [Guidelines и Skills](#guidelines-vs-skills)
- [Правила проекта](#project-rules)
    - [Запись правил](#recording-rules)
    - [Определение соглашений приложения](#inferring-your-applications-conventions)
    - [Отключение правил проекта](#disabling-project-rules)
- [API документации](#documentation-api)
- [Расширение Boost](#extending-boost)
    - [Добавление поддержки других IDE / AI-агентов](#adding-support-for-other-ides-ai-agents)

<a name="introduction"></a>
## Введение

Laravel Boost ускоряет разработку с помощью AI, предоставляя необходимые рекомендации и agent skills, которые помогают AI-агентам писать качественные Laravel-приложения, соответствующие лучшим практикам Laravel.

Boost также предоставляет мощный API документации экосистемы Laravel, объединяющий встроенный MCP-инструмент и обширную базу знаний с более чем 17 000 фрагментов информации о Laravel, улучшенную семантическим поиском через embeddings для точных результатов с учетом контекста. Boost инструктирует AI-агентов вроде Claude Code и Cursor использовать этот API, чтобы узнавать об актуальных возможностях Laravel и лучших практиках.

<a name="installation"></a>
## Установка

Laravel Boost можно установить через Composer:

```shell
composer require laravel/boost --dev
```

Затем установите MCP-сервер и рекомендации по написанию кода:

```shell
php artisan boost:install
```

Команда `boost:install` сгенерирует соответствующие файлы рекомендаций и skills для coding agents, выбранных вами во время установки.

После установки Laravel Boost можно начинать работу с Cursor, Claude Code или любым другим AI-агентом.

> [!NOTE]
> Сгенерированный файл MCP-конфигурации (`.mcp.json`), файлы рекомендаций (`CLAUDE.md`, `AGENTS.md`, `junie/` и т. д.) и файл конфигурации `boost.json` можно добавить в `.gitignore`, так как они автоматически пересоздаются при запуске `boost:install` и `boost:update`.

<a name="set-up-your-agents"></a>
### Настройка агентов

```text tab=Cursor
1. Откройте палитру команд (`Cmd+Shift+P` или `Ctrl+Shift+P`)
2. Нажмите `enter` на "/open MCP Settings"
3. Включите переключатель для `laravel-boost`
```

```text tab=Claude Code
Поддержка Claude Code обычно включается автоматически. Если этого не произошло, откройте shell в каталоге проекта и выполните:

claude mcp add -s local -t stdio laravel-boost php artisan boost:mcp
```

```text tab=Codex
Поддержка Codex обычно включается автоматически. Если этого не произошло, откройте shell в каталоге проекта и выполните:

codex mcp add laravel-boost -- php "artisan" "boost:mcp"
```

```text tab=Gemini CLI
Поддержка Gemini CLI обычно включается автоматически. Если этого не произошло, откройте shell в каталоге проекта и выполните:

gemini mcp add -s project -t stdio laravel-boost php artisan boost:mcp
```

```text tab=GitHub Copilot (VS Code)
1. Откройте палитру команд (`Cmd+Shift+P` или `Ctrl+Shift+P`)
2. Нажмите `enter` на "MCP: List Servers"
3. Выберите `laravel-boost` и нажмите `enter`
4. Выберите "Start server"
```

```text tab=Junie
1. Дважды нажмите `shift`, чтобы открыть палитру команд
2. Найдите "MCP Settings" и нажмите `enter`
3. Отметьте флажок рядом с `laravel-boost`
4. Нажмите "Apply" в правом нижнем углу
```

<a name="keeping-boost-resources-updated"></a>
### Поддержание ресурсов Boost в актуальном состоянии

Вы можете периодически обновлять локальные ресурсы Boost (AI-рекомендации и skills), чтобы они соответствовали актуальным версиям пакетов экосистемы Laravel, установленных в проекте. Для этого используйте Artisan-команду `boost:update`.

```shell
php artisan boost:update
```

Также можно автоматизировать процесс, добавив его в Composer scripts `post-update-cmd`:

```json
{
  "scripts": {
    "post-update-cmd": [
      "@php artisan boost:update --ansi"
    ]
  }
}
```

По умолчанию команда `boost:update` обновляет только те ресурсы Boost, которые уже опубликованы в вашем приложении. Если вы хотите, чтобы Boost просканировал приложение на наличие недавно установленных пакетов и предложил опубликовать соответствующие guidelines и skills, используйте опцию `--discover`:

```shell
php artisan boost:update --discover
```

<a name="mcp-server"></a>
## MCP-сервер

Laravel Boost предоставляет MCP-сервер (Model Context Protocol), который открывает инструменты для взаимодействия AI-агентов с вашим Laravel-приложением. Эти инструменты позволяют агентам изучать структуру приложения, выполнять запросы к базе данных, запускать код и многое другое.

<a name="available-mcp-tools"></a>
### Доступные MCP-инструменты

<div class="overflow-auto">

| Название             | Примечания                                                                                                      |
| -------------------- | --------------------------------------------------------------------------------------------------------------- |
| Application Info     | Читает версии PHP и Laravel, движок базы данных, список пакетов экосистемы с версиями и Eloquent-модели        |
| Browser Logs         | Читает логи и ошибки браузера                                                                                   |
| Database Connections | Инспектирует доступные соединения базы данных, включая соединение по умолчанию                                  |
| Database Query       | Выполняет запрос к базе данных                                                                                  |
| Database Schema      | Читает схему базы данных                                                                                        |
| Get Absolute URL     | Преобразует относительные URI в абсолютные, чтобы агенты генерировали корректные URL                            |
| Last Error           | Читает последнюю ошибку из логов приложения                                                                     |
| Read Log Entries     | Читает последние N записей лога                                                                                 |
| Record Rule          | Записывает постоянное [правило проекта](#project-rules) в `.ai/rules`, чтобы оно применялось будущими агентами |
| Search Docs          | Выполняет запрос к размещенному Laravel API документации, учитывая установленные пакеты                         |

</div>

<a name="manually-registering-the-mcp-server"></a>
### Ручная регистрация MCP-сервера

Иногда MCP-сервер Laravel Boost нужно зарегистрировать в выбранном редакторе вручную. Используйте следующие параметры:

<table>
<tr><td><strong>Command</strong></td><td><code>php</code></td></tr>
<tr><td><strong>Args</strong></td><td><code>artisan boost:mcp</code></td></tr>
</table>

Пример JSON:

```json
{
    "mcpServers": {
        "laravel-boost": {
            "command": "php",
            "args": ["artisan", "boost:mcp"]
        }
    }
}
```

<a name="ai-guidelines"></a>
## AI-рекомендации

AI-рекомендации - это составные instruction-файлы, загружаемые заранее, чтобы дать AI-агентам важный контекст о пакетах экосистемы Laravel. Они содержат основные соглашения, лучшие практики и framework-specific паттерны, помогающие агентам генерировать согласованный и качественный код.

<a name="available-ai-guidelines"></a>
### Доступные AI-рекомендации

Laravel Boost включает AI-рекомендации для следующих пакетов и фреймворков. Рекомендации `core` дают общие советы по пакету, применимые ко всем версиям.

<div class="overflow-auto">

| Пакет             | Поддерживаемые версии  |
| ----------------- | ---------------------- |
| Core & Boost      | core                   |
| Laravel Framework | core, 10.x, 11.x, 12.x, 13.x |
| Livewire          | core, 2.x, 3.x, 4.x    |
| Flux UI           | core, free, pro        |
| Folio             | core                   |
| Herd              | core                   |
| Inertia Laravel   | core, 1.x, 2.x, 3.x    |
| Inertia React     | core, 1.x, 2.x, 3.x    |
| Inertia Vue       | core, 1.x, 2.x, 3.x    |
| Inertia Svelte    | core, 1.x, 2.x, 3.x    |
| MCP               | core                   |
| Pennant           | core                   |
| Pest              | core, 3.x, 4.x         |
| PHPUnit           | core                   |
| Pint              | core                   |
| Sail              | core                   |
| Tailwind CSS      | core, 3.x, 4.x         |
| Livewire Volt     | core                   |
| Wayfinder         | core                   |
| Enforce Tests     | conditional            |

</div>

> **Note:** Чтобы поддерживать AI-рекомендации в актуальном состоянии, смотрите раздел [Поддержание ресурсов Boost в актуальном состоянии](#keeping-boost-resources-updated).

<a name="adding-custom-ai-guidelines"></a>
### Добавление собственных AI-рекомендаций

Чтобы расширить Laravel Boost своими AI-рекомендациями, добавьте файлы `.blade.php` или `.md` в каталог `.ai/guidelines/*` вашего приложения. Эти файлы будут автоматически включены в рекомендации Laravel Boost при запуске `boost:install`.

<a name="overriding-boost-ai-guidelines"></a>
### Переопределение AI-рекомендаций Boost

Вы можете переопределить встроенные AI-рекомендации Boost, создав собственные рекомендации с совпадающими путями. Если пользовательская рекомендация совпадает с существующим путем рекомендации Boost, Boost использует вашу версию вместо встроенной.

Например, чтобы переопределить рекомендации Boost "Inertia React v2 Form Guidance", создайте файл `.ai/guidelines/inertia-react/2/forms.blade.php`. При запуске `boost:install` Boost включит вашу рекомендацию вместо стандартной.

<a name="third-party-package-ai-guidelines"></a>
### AI-рекомендации сторонних пакетов

Если вы сопровождаете сторонний пакет и хотите, чтобы Boost включал для него AI-рекомендации, добавьте файл `resources/boost/guidelines/core.blade.php` в пакет. Когда пользователи пакета выполнят `php artisan boost:install`, Boost автоматически загрузит ваши рекомендации.

AI-рекомендации должны кратко описывать назначение пакета, указывать нужную структуру файлов или соглашения и объяснять, как создавать или использовать основные возможности (с примерами команд или кода). Делайте их лаконичными, практичными и сфокусированными на лучших практиках, чтобы AI мог генерировать корректный код:

```php
## Package Name

This package provides [brief description of functionality].

### Features

- Feature 1: [clear & short description].
- Feature 2: [clear & short description]. Example usage:

@verbatim
<code-snippet name="How to use Feature 2" lang="php">
$result = PackageName::featureTwo($param1, $param2);
</code-snippet>
@endverbatim
```

<a name="agent-skills"></a>
## Agent Skills

[Agent Skills](https://agentskills.io/home) - это легкие, целевые модули знаний, которые агенты могут активировать по требованию при работе с конкретными областями. В отличие от guidelines, которые загружаются заранее, skills позволяют загружать подробные паттерны и лучшие практики только когда они релевантны, уменьшая раздувание контекста и улучшая качество AI-генерируемого кода.

Когда вы запускаете `boost:install` и выбираете skills как возможность, skills автоматически устанавливаются на основе пакетов, найденных в `composer.json`. Например, если проект включает `livewire/livewire`, skill `livewire-development` будет установлен автоматически. Skills, входящие в Boost, например `infer-conventions`, устанавливаются независимо от используемых пакетов.

<a name="available-skills"></a>
### Доступные skills

<div class="overflow-auto">

| Skill                      | Пакет          |
| -------------------------- | -------------- |
| fluxui-development         | Flux UI        |
| folio-routing              | Folio          |
| infer-conventions          | Boost          |
| inertia-react-development  | Inertia React  |
| inertia-svelte-development | Inertia Svelte |
| inertia-vue-development    | Inertia Vue    |
| livewire-development       | Livewire       |
| mcp-development            | MCP            |
| pennant-development        | Pennant        |
| pest-testing               | Pest           |
| tailwindcss-development    | Tailwind CSS   |
| volt-development           | Volt           |
| wayfinder-development      | Wayfinder      |

</div>

> **Note:** Чтобы поддерживать skills в актуальном состоянии, смотрите раздел [Поддержание ресурсов Boost в актуальном состоянии](#keeping-boost-resources-updated).

<a name="custom-skills"></a>
### Собственные skills

Чтобы создать собственный skill, добавьте файл `SKILL.md` в каталог `.ai/skills/{skill-name}/` вашего приложения. При запуске `boost:update` ваш skill будет установлен вместе со встроенными skills Boost.

Например, чтобы создать skill для доменной логики приложения:

```
.ai/skills/creating-invoices/SKILL.md
```

<a name="overriding-skills"></a>
### Переопределение skills

Вы можете переопределить встроенные skills Boost, создав собственные skills с совпадающими именами. Если пользовательский skill совпадает с именем встроенного skill Boost, Boost использует вашу версию вместо стандартной.

Например, чтобы переопределить skill `livewire-development`, создайте файл `.ai/skills/livewire-development/SKILL.md`. При запуске `boost:update` Boost включит ваш skill вместо стандартного.

<a name="third-party-package-skills"></a>
### Skills сторонних пакетов

Если вы сопровождаете сторонний пакет и хотите, чтобы Boost включал skills для него, добавьте файл `resources/boost/skills/{skill-name}/SKILL.md` в пакет. Когда пользователи пакета выполнят `php artisan boost:install`, Boost автоматически установит ваши skills в соответствии с пользовательскими настройками.

Boost Skills поддерживают [формат Agent Skills](https://agentskills.io/what-are-skills) и должны быть структурированы как папка с файлом `SKILL.md`, содержащим YAML frontmatter и Markdown-инструкции. Файл `SKILL.md` должен включать обязательные поля frontmatter (`name` и `description`) и может дополнительно содержать scripts, templates и справочные материалы.

Skills должны описывать нужную структуру файлов или соглашения и объяснять, как создавать или использовать основные возможности (с примерами команд или кода). Делайте их лаконичными, практичными и сфокусированными на лучших практиках, чтобы AI мог генерировать корректный код:

```markdown
---
name: package-name-development
description: Build and work with PackageName features, including components and workflows.
---

# Package Name Development

## When to use this skill
Use this skill when working with PackageName features...

## Features

- Feature 1: [clear & short description].
- Feature 2: [clear & short description]. Example usage:

$result = PackageName::featureTwo($param1, $param2);
```

<a name="guidelines-vs-skills"></a>
## Guidelines и Skills

Laravel Boost предоставляет два разных способа дать AI-агентам контекст о приложении: **guidelines** и **skills**.

**Guidelines** загружаются заранее при старте AI-агента и дают основной контекст о соглашениях Laravel и лучших практиках, широко применимых по всей кодовой базе.

**Skills** активируются по требованию при работе над конкретными задачами и содержат подробные паттерны для отдельных областей, например компонентов Livewire или тестов Pest. Загрузка skills только при необходимости уменьшает раздувание контекста и повышает качество кода.

<div class="overflow-auto">

| Аспект       | Guidelines                        | Skills                                  |
| ------------ | --------------------------------- | --------------------------------------- |
| **Загрузка** | Заранее, всегда присутствуют      | По требованию, когда релевантны         |
| **Область**  | Широкая, фундаментальная          | Узкая, task-specific                    |
| **Цель**     | Основные соглашения и практики    | Подробные паттерны реализации           |

</div>

И рекомендации, и навыки описывают экосистему Laravel. Для фиксации соглашений собственного приложения используйте [правила проекта](#project-rules).

<a name="project-rules"></a>
## Правила проекта

Рекомендации и навыки учат агентов писать код на Laravel, а правила проекта - писать именно ваше приложение. Правилом может стать все, что в противном случае пришлось бы заново объяснять в каждой новой сессии:

<div class="content-list" markdown="1">

- Решения, принятые вами, вашими агентами или коллегами в ходе работы.
- Рекомендации по стилю и предпочтения, которым агенту сложно следовать без явного указания.
- Неочевидные ограничения и особенности, которые нельзя вывести из окружающего кода.

</div>

Правила хранятся в Markdown-файлах каталога `.ai/rules` приложения и должны быть добавлены в систему контроля версий. В отличие от собственной памяти агента, привязанной к пользователю и сессии, правила доступны всей команде и каждому агенту, работающему над приложением.

В frontmatter каждого файла правил указываются шаблоны путей, к которым он применяется:

```markdown
---
paths:
  - app/Http/Controllers/**
---

# HTTP-контроллеры

## Наследование от BaseController для ограничения по арендатору

Все контроллеры должны наследовать `App\Http\Controllers\BaseController`, который
применяет область запроса текущего арендатора. Прямое наследование от базового
контроллера Laravel приведет к утечке данных между арендаторами.
```

Кроме того, Boost поддерживает файл `.ai/rules/index.md`, сопоставляющий шаблоны путей с файлами правил. Агентам предписывается обращаться к этому индексу перед планированием или редактированием файла, поэтому правило загружается только тогда, когда оно относится к задаче:

```markdown
# Индекс правил проекта

Перед планированием или редактированием найдите строку, шаблоны которой соответствуют
пути файла, и прочитайте указанный файл правил.

| Применяется к | Файл правил |
| --- | --- |
| app/Http/Controllers/** | .ai/rules/controllers.md |
| app/Models/** | .ai/rules/models.md |
```

> [!NOTE]
> В отличие от `.mcp.json` и сгенерированных файлов рекомендаций, каталог `.ai/rules` следует добавить в систему контроля версий, чтобы правила были доступны всей команде.

<a name="recording-rules"></a>
### Запись правил

Чтобы записать правило, можно просто попросить агента запомнить его:

```text
Запомни, что все денежные значения хранятся целым числом копеек, а не дробным числом.
```

Агент вызовет MCP-инструмент Boost `record-rule`, передав `glob`, краткий `title` и `note`. Boost добавит правило в подходящий раздел, при необходимости создаст файл правил и обновит индекс.

Всегда записывайте правила с помощью инструмента `record-rule`, а не создавайте файлы вручную. При записи правила Boost заново формирует `.ai/rules/index.md`, на который агенты полагаются при поиске правил для текущего файла. Добавленное вручную правило не будет обнаружено до следующего формирования индекса.

<a name="inferring-your-applications-conventions"></a>
### Определение соглашений приложения

Последовательная запись новых правил удобна для дальнейшей работы, однако в существующем приложении уже могут накопиться соглашения за несколько лет. Навык `infer-conventions` позволяет сформировать начальный набор правил на основе уже написанного кода. Чтобы начать, попросите агента использовать этот навык:

```text
Используй навык infer-conventions
```

Навык проверит приложение по перечню аспектов соглашений Laravel, включая валидацию, контроллеры, авторизацию, модели, архитектуру, тестирование, frontend, базу данных и консоль, а затем выполнит свободный поиск таких паттернов, как базовые классы, общие трейты и структура модулей.

Навык документирует фактическое устройство кода, а не желаемое. Он записывает только хорошо подтвержденные соглашения, отличающиеся от стандартных, пропускает правила фреймворка и все, что уже обеспечивает Pint или Rector, а при наличии действительно смешанных подходов сообщает о них вместо записи правила. Перед созданием правил навык покажет каждое найденное соглашение вместе с подтверждающими примерами для вашего одобрения. Чтобы записать все найденные соглашения без подтверждения, можно сказать ему «yolo».

<a name="disabling-project-rules"></a>
### Отключение правил проекта

Правила проекта включены по умолчанию. Чтобы полностью отключить их, задайте следующую переменную окружения. Это удалит MCP-инструмент `record-rule` и прекратит управление каталогом `.ai/rules` со стороны Boost:

```ini
BOOST_RULES_ENABLED=false
```

<a name="documentation-api"></a>
## API документации

Laravel Boost включает Documentation API, который дает AI-агентам доступ к обширной базе знаний, содержащей более 17 000 фрагментов информации о Laravel. API использует семантический поиск через embeddings, чтобы выдавать точные результаты с учетом контекста.

MCP-инструмент `Search Docs` позволяет агентам запрашивать размещенный Laravel сервис API документации и получать документацию на основе установленных пакетов. AI-рекомендации и skills Boost автоматически инструктируют coding agent использовать этот API.

<div class="overflow-auto">

| Пакет             | Поддерживаемые версии |
| ----------------- | --------------------- |
| Laravel Framework | 10.x, 11.x, 12.x, 13.x |
| Filament          | 2.x, 3.x, 4.x, 5.x    |
| Flux UI           | 2.x Free, 2.x Pro     |
| Inertia           | 1.x, 2.x              |
| Livewire          | 1.x, 2.x, 3.x, 4.x    |
| Nova              | 4.x, 5.x              |
| Pest              | 3.x, 4.x              |
| Tailwind CSS      | 3.x, 4.x              |

</div>

<a name="extending-boost"></a>
## Расширение Boost

Boost из коробки работает со многими популярными IDE и AI-агентами. Если ваш coding tool пока не поддерживается, можно создать собственного агента и интегрировать его с Boost.

<a name="adding-support-for-other-ides-ai-agents"></a>
### Добавление поддержки других IDE / AI-агентов

Чтобы добавить поддержку новой IDE или AI-агента, создайте класс, расширяющий `Laravel\Boost\Install\Agents\Agent`, и реализуйте один или несколько следующих контрактов в зависимости от потребностей:

- `Laravel\Boost\Contracts\SupportsGuidelines` - добавляет поддержку AI-рекомендаций.
- `Laravel\Boost\Contracts\SupportsMcp` - добавляет поддержку MCP.
- `Laravel\Boost\Contracts\SupportsSkills` - добавляет поддержку Agent Skills.

<a name="writing-the-agent"></a>
#### Написание агента

```php
<?php

declare(strict_types=1);

namespace App;

use Laravel\Boost\Contracts\SupportsGuidelines;
use Laravel\Boost\Contracts\SupportsMcp;
use Laravel\Boost\Contracts\SupportsSkills;
use Laravel\Boost\Install\Agents\Agent;

class CustomAgent extends Agent implements SupportsGuidelines, SupportsMcp, SupportsSkills
{
    // Ваша реализация...
}
```

Пример реализации смотрите в [ClaudeCode.php](https://github.com/laravel/boost/blob/main/src/Install/Agents/ClaudeCode.php).

<a name="registering-the-agent"></a>
#### Регистрация агента

Зарегистрируйте собственного агента в методе `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
use Laravel\Boost\Boost;

public function boot(): void
{
    Boost::registerAgent('customagent', CustomAgent::class);
}
```

После регистрации агент будет доступен для выбора при запуске `php artisan boost:install`.
