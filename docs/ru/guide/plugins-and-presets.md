# Плагины и пресеты настроек

## Плагины

Архитектура Vue CLI основана на плагинах. Если изучить `package.json` в свежесозданном проекте, то вы найдёте зависимости имена которых начинаются с `@vue/cli-plugin-`. Такие плагины могут изменять внутреннюю конфигурацию webpack и внедрять команды в `vue-cli-service`. Большинство возможностей, упоминаемых при создании проекта, реализованы ими.

Основанная на плагинах архитектура позволяет Vue CLI оставаться гибкой и расширяемой. Если вы хотите создать собственный плагин — изучите [руководство по созданию плагинов](../dev-guide/plugin-dev.md).

::: tip Совет
Устанавливать и управлять плагинами можно в GUI с помощью команды `vue ui`.
:::

### Установка плагинов в существующий проект

Каждый плагин для CLI поставляется с генератором (который создаёт файлы) и плагином для runtime (который меняет конфигурацию webpack и внедряет команды). Когда вы используете `vue create` для создания нового проекта, некоторые плагины будут уже предустановлены, в зависимости от вашего выбора необходимых возможностей. В случае, когда необходимо установить плагин в уже существующий проект, вы должны сделать это командой `vue add`:

``` bash
vue add @vue/eslint
```

::: tip Совет
Команда `vue add` специально разработана для установки и запуска плагинов Vue CLI. Это не означает, что она заменяет обычные npm-пакеты. Для установки обычных npm-пакетов по-прежнему используйте менеджер пакетов который использовали.
:::

::: warning Предупреждение
Рекомендуется закоммитить все текущие изменения в вашем проекте перед запуском `vue add`, поскольку команда запустит генератор файлов плагина и, возможно, внесёт изменения в уже существующие файлы.
:::

Команда `@vue/eslint` трансформируется в полное название пакета `@vue/cli-plugin-eslint`, устанавливает его из npm, и запускает его генератор.

``` bash
# это аналогично предыдущей команде
vue add @vue/cli-plugin-eslint
```

Без префикса `@vue` команда будет трансформировать название к публичному пакету. Например, чтобы установить сторонний плагин `vue-cli-plugin-apollo`:

``` bash
# устанавливает и запускает vue-cli-plugin-apollo
vue add apollo
```

Вы можете также использовать сторонние плагины со специфичным scope. Например, если плагин назван `@foo/vue-cli-plugin-bar`, то его можно добавить командой:

``` bash
vue add @foo/bar
```

Можно передавать опции генерации в установленный плагин (для пропуска интерактивного выбора):

``` bash
vue add @vue/eslint --config airbnb --lintOn save
```

Добавление `vue-router` и `vuex` — особый случай, у них нет собственных плагинов, но вы тем не менее можете их добавить:

``` bash
vue add router
vue add vuex
```

Если плагин уже установлен, вы можете пропустить установку и только вызвать его генератор с помощью команды `vue invoke`. Команда принимает такие же аргументы, как и `vue add`.

::: tip Совет
Если, по какой-либо причине, ваши плагины перечисляются в файле `package.json`, отличном от расположенного в проекте, то укажите путь к каталогу с другим файлом `package.json` в опции `vuePlugins.resolveFrom` в файле `package.json` проекта.

Например, если у вас есть файл `.config/package.json`:

```json
{
  "vuePlugins": {
    "resolveFrom": ".config"
  }
}
```
:::

### Локальный плагин проекта

Если требуется доступ к API плагина в вашем проекте и вы не хотите создавать полноценный плагин для этого, можно использовать опцию `vuePlugins.service` в файле `package.json`:

```json
{
  "vuePlugins": {
    "service": ["my-commands.js"]
  }
}
```

Каждому файлу необходимо экспортировать функцию, принимающую API плагина первым аргументом. Для получения дополнительной информации об API плагина, ознакомьтесь с [руководством по разработке плагинов](../dev-guide/plugin-dev.md).

Вы можете добавить файлы, которые будут вести себя как плагины UI опцией `vuePlugins.ui`:

```json
{
  "vuePlugins": {
    "ui": ["my-ui.js"]
  }
}
```

Для получения дополнительной информации, ознакомьтесь с [API плагина для UI](../dev-guide/ui-api.md).

## Пресеты настроек

Пресет настроек для Vue CLI — JSON-объект, который содержит предустановленные опции и плагины для создания нового проекта, чтобы пользователю не приходилось проходить через цепочку вопросов для их выбора.

Пресеты сохраняемые во время `vue create` создаются в конфигурационном файле в домашнем каталоге пользователя (`~/.vuerc`). Вы можете напрямую изменять этот файл для внесения правок / добавления / удаление сохранённых пресетов.

Вот пример пресета настроек:

``` json
{
  "useConfigFiles": true,
  "router": true,
  "vuex": true,
  "cssPreprocessor": "sass",
  "plugins": {
    "@vue/cli-plugin-babel": {},
    "@vue/cli-plugin-eslint": {
      "config": "airbnb",
      "lintOn": ["save", "commit"]
    }
  }
}
```

Информация из пресета настроек используется генераторами плагинов для создания в проекте соответствующих файлов. Кроме того, в дополнении к полям выше, возможно добавление дополнительных настроек для встроенных инструментов:

``` json
{
  "useConfigFiles": true,
  "plugins": {...},
  "configs": {
    "vue": {...},
    "postcss": {...},
    "eslintConfig": {...},
    "jest": {...}
  }
}
```

Эти дополнительные конфигурации будут объединены в `package.json` или соответствующие файлы конфигураций, основываясь на значении `useConfigFiles`. Например, при указании `"useConfigFiles": true`, значение `configs.vue` будет объединено в файл `vue.config.js`.

### Версии плагинов в пресете

Вы можете явно указать версии используемых плагинов:

``` json
{
  "plugins": {
    "@vue/cli-plugin-eslint": {
      "version": "^3.0.0",
      // ... другие настройки для этого плагина
    }
  }
}
```

Обратите внимание, это не требуется для официальных плагинов — если не указано, то CLI автоматически будет использовать последнюю версию, доступную в регистре. Тем не менее, **рекомендуется указывать явный диапазон версий для любых сторонних плагинов, перечисленных в пресете**.

### Предоставление интерактивного выбора пользователю

Каждый плагин может предоставлять интерактивный выбор пользователю на этапе создания проекта, но при использовании пресета, все эти запросы пользователю пропускаются, потому что Vue CLI подразумевает что все настройки плагинов уже определены в пресете.

В некоторых случаях может возникнуть необходимость, чтобы пресет только объявлял плагины, оставляя при этом некоторую гибкость настройки, запрашивая выбор у пользователя из вариантов, предоставляемых плагином.

Для таких случаев можно указать `"prompts": true` в настройках плагина, чтобы позволить пользователю сделать свой выбор:

``` json
{
  "plugins": {
    "@vue/cli-plugin-eslint": {
      // разрешаем пользователю выбрать свою конфигурацию ESLint
      "prompts": true
    }
  }
}
```

### Пресет из удалённого репозитория

Вы можете поделиться пресетом с другими разработчиками, опубликовав его в git репозитории. Репозиторий должен содержать следующие файлы:

  - `preset.json`: основной файл, содержащий настройки пресета (обязателен).
  - `generator.js`: [генератор](../dev-guide/plugin-dev.md#generator), который внедряет или модифицирует файлы в проекте.
  - `prompts.js`: [файл подсказок](../dev-guide/plugin-dev.md#prompts-for-3rd-party-plugins), который может собирать настройки для генератора.

После публикации репозитория, при создании проекта теперь можно указать опцию `--preset` для использования пресета из удалённого репозитория:

``` bash
# использование пресета из репозитория GitHub
vue create --preset username/repo my-project
```

GitLab и BitBucket также поддерживаются. Убедитесь, что используете опцию `--clone` при загрузке из стороннего репозитория:

``` bash
vue create --preset gitlab:username/repo --clone my-project
vue create --preset bitbucket:username/repo --clone my-project
```

### Пресет из локального файла

При разработке удалённого пресета настроек, может часто требоваться отправлять пресет в удалённый репозиторий для его проверки. Для упрощения разработки можно использовать локальные пресеты напрямую. Vue CLI будет загружать локальные пресеты, если путь в значении `--preset` будет относительным или абсолютным, или заканчиваться на `.json`:

``` bash
# ./my-preset должен быть каталогом, содержащим preset.json
vue create --preset ./my-preset my-project

# или напрямую использовать json-файл в cwd:
vue create --preset my-preset.json
```