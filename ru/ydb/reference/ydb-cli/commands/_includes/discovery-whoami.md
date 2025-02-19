---
sourcePath: ru/ydb/ydb-docs-core/ru/core/reference/ydb-cli/commands/_includes/discovery-whoami.md
---
# Проверка аутентификации

Информационная команда `discovery whoami` позволяет проверить, от имени какой учетной записи воспринимает запросы сервер:

``` bash
{{ ydb-cli }} [connection options] discovery whoami [-g]
```

{% include [conn_options_ref.md](conn_options_ref.md) %}

В ответ выводится имя учетной записи (User SID) и, если указана опция `-g`, то информация о принадлежности учетной записи группам.

Если на сервере {{ ydb-short-name }} не включена аутентификация (что может применяться, например, при самостоятельном локальном развертывании), то выполнение команды завершится с ошибкой.

Поддержка опции `-g` зависит от конфигурации сервера. Если она не включена, то вы будете получать в ответ `User has no groups` вне зависимости от фактического включения вашей учетной записи в какие-либо группы.

## Пример

``` bash
$ ydb --profile db1 discovery whoami -g
User SID: aje5kkjdgs0puc18976co@as

User has no groups
```

