---
sourcePath: ru/ydb/ydb-docs-core/ru/core/reference/ydb-sdk/_includes/auth.md
---
# Аутентификация при соединении SDK с БД

Как описано в статье о [подключении клиента {{ ydb-short-name }} к серверу и аутентификации](../../../concepts/connect.md), для успешной аутентификации в запрос клиентом должен быть добавлен аутентификационный токен, проверяемый сервером.

{{ ydb-short-name }} SDK использует объект, отвечающий за генерацию таких токенов. SDK предоставляет встроенные методы получения такого объекта:

1. Специализированные методы под разные [режимы аутентификации](../../../concepts/connect.md#auth-modes) с явной передачей параметров.
2. Метод определения режима аутентификации и необходимых параметров из окружения, в котором запускается приложение.

Обычно объект генерации токенов создается перед инициализацией драйвера {{ ydb-short-name }} и передается параметром в его конструктор. C++ и Go SDK дополнительно позволяют через один драйвер работать с несколькими БД и объектами генерации токенов.

Если объект генерации токенов не определен, драйвер не будет добавлять в запросы какой-либо аутентификационной информации, что может позволить успешно соединиться с локально развернутыми кластерами {{ ydb-short-name }}, не требующими аутентификации. Для всех доступных по сети баз данных такие запросы будут отклонены с выдачей ошибки аутентификации.

## Методы создания объекта генерации токенов {#auth-provider}

На приведенные ниже методы можно кликнуть, чтобы перейти к исходному коду релевантного примера на github.com. Также можно ознакомиться с [рецептами кода Аутентификации](../recipes/auth/index.md).

{% list tabs %}

- Python

  [Режим](../../../concepts/connect.md#auth-modes) | Метод 
  ----- | ----- 
  Anonymous | [`ydb.AnonymousCredentials()`](https://github.com/yandex-cloud/ydb-python-sdk/tree/master/examples/anonymous-credentials)
  Access Token | [`ydb.AccessTokenCredentials( token )`](https://github.com/yandex-cloud/ydb-python-sdk/tree/master/examples/access-token-credentials)
  Metadata | [`ydb.iam.MetadataUrlCredentials()`](https://github.com/yandex-cloud/ydb-python-sdk/tree/master/examples/metadata-credentials)
  Service Account Key | [`ydb.iam.ServiceAccountCredentials.from_file(`</br>&nbsp;&nbsp;&nbsp;&nbsp;`key_file, iam_endpoint=None, iam_channel_credentials=None )`](https://github.com/yandex-cloud/ydb-python-sdk/tree/master/examples/service-account-credentials)
  Определяется по переменным окружения | `ydb.construct_credentials_from_environ()`

- Go

  [Режим](../../../concepts/connect.md#auth-modes) | Пакет | Метод
  ----- | ----- | ----
  Anonymous | [ydb-go-sdk/v3](https://github.com/ydb-platform/ydb-go-sdk/blob/master/go.mod) | [`ydb.WithAnonymousCredentials()`](https://github.com/ydb-platform/ydb-go-examples/tree/master/cmd/auth/anonymous_credentials)
  Access Token | [ydb-go-sdk/v3](https://github.com/ydb-platform/ydb-go-sdk/blob/master/go.mod) | [`ydb.WithAccessTokenCredentials( token )`](https://github.com/ydb-platform/ydb-go-examples/tree/master/cmd/auth/access_token_credentials)
  Metadata | [ydb-go-yc](https://github.com/ydb-platform/ydb-go-yc/blob/master/go.mod) | [`yc.WithMetadataCredentials( ctx )`](https://github.com/ydb-platform/ydb-go-examples/tree/master/cmd/auth/metadata_credentials)
  Service Account Key | [ydb-go-yc](https://github.com/ydb-platform/ydb-go-yc/blob/master/go.mod) | [`yc.WithServiceAccountKeyFileCredentials( key_file )`](https://github.com/ydb-platform/ydb-go-examples/tree/master/cmd/auth/service_account_credentials)
  Определяется по переменным окружения | [ydb-go-sdk-auth-environ](https://github.com/ydb-platform/ydb-go-sdk-auth-environ/blob/master/go.mod) | [`environ.WithEnvironCredentials(ctx)`](https://github.com/ydb-platform/ydb-go-examples/tree/master/cmd/auth/environ)

- Java

  [Режим](../../../concepts/connect.md#auth-modes) | Метод
  ----- | -----
  Anonymous | [`com.yandex.ydb.core.auth.NopAuthProvider.INSTANCE`](https://github.com/yandex-cloud/ydb-java-sdk/tree/master/examples/auth/anonymous_credentials)
  Access Token | [`com.yandex.ydb.auth.iam.CloudAuthProvider.newAuthProvider(`</br>&nbsp;&nbsp;&nbsp;&nbsp;`yandex.cloud.sdk.auth.provider.IamTokenCredentialProvider`</br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.builder()`</br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.token(accessToken)`</br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.build()`</br>`);`](https://github.com/yandex-cloud/ydb-java-sdk/tree/master/examples/auth/access_token_credentials)
  Metadata | [`com.yandex.ydb.auth.iam.CloudAuthProvider.newAuthProvider(`</br>&nbsp;&nbsp;&nbsp;&nbsp;`yandex.cloud.sdk.auth.provider.ComputeEngineCredentialProvider`</br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.builder()`</br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.build()`</br>`);`](https://github.com/yandex-cloud/ydb-java-sdk/tree/master/examples/auth/metadata_credentials)
  Service Account Key | [`com.yandex.ydb.auth.iam.CloudAuthProvider.newAuthProvider(`</br>&nbsp;&nbsp;&nbsp;&nbsp;`yandex.cloud.sdk.auth.provider.ApiKeyCredentialProvider`</br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.builder()`</br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.fromFile(Paths.get(saKeyFile))`</br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.build()`</br>`);`](https://github.com/yandex-cloud/ydb-java-sdk/tree/master/examples/auth/service_account_credentials)
  Определяется по переменным окружения | [`com.yandex.ydb.auth.iam.CloudAuthHelper.getAuthProviderFromEnviron();`](https://github.com/yandex-cloud/ydb-java-sdk/tree/master/examples/auth/environ/src/main/java/com/yandex/ydb/example)

- Node.js

  [Режим](../../../concepts/connect.md#auth-modes) | Метод
  ----- | -----
  Anonymous | [`new 'ydb-sdk'.AnonymousAuthService()`](https://github.com/ydb-platform/ydb-nodejs-sdk/tree/main/examples/auth/anonymous-credentials)
  Access Token | [`new 'ydb-sdk'.TokenAuthService( accessToken, database )`](https://github.com/ydb-platform/ydb-nodejs-sdk/tree/main/examples/auth/access-token-credentials)
  Metadata | [`new 'ydb-sdk'.MetadataAuthService( database )`](https://github.com/ydb-platform/ydb-nodejs-sdk/tree/main/examples/auth/metadata-credentials)
  Service Account Key | [`new 'ydb-sdk'.getSACredentialsFromJson( saKeyFile )`](https://github.com/ydb-platform/ydb-nodejs-sdk/tree/main/examples/auth/service-account-credentials)
  Определяется по переменным окружения | [`new 'ydb-sdk'.getCredentialsFromEnv( entryPoint, database, logger )`](https://github.com/ydb-platform/ydb-nodejs-sdk/tree/main/examples/auth/environ)

{% endlist %}

## Порядок определения режима и параметров аутентификации из окружения {#env}

Выполняется следующий алгоритм, одинаковый для всех SDK:

1. Если задано значение переменной окружения `YDB_SERVICE_ACCOUNT_KEY_FILE_CREDENTIALS`, то используется режим аутентификации **System Account Key**, а ключ загружается из файла, имя которого указано в данной переменной
2. Иначе, если задано значение переменной окружения `YDB_ANONYMOUS_CREDENTIALS`, равное 1, то используется анонимный режим аутентификации
3. Иначе, если задано значение переменной окружения `YDB_METADATA_CREDENTIALS`, равное 1, то используется режим аутентификации **Metadata**
4. Иначе, если задано значение переменной окружения `YDB_ACCESS_TOKEN_CREDENTIALS`, то используется режим аутентификации **Access token**, в который передается значение данной переменной
5. Иначе используется режим аутентификации **Metadata**

Наличие последним пунктом алгоритма выбора режима **Metadata** позволяет развернуть рабочее приложение на виртуальных машинах и в Cloud Functions {{ yandex-cloud }} без задания каких-либо переменных окружения.

## Особенности Python SDK

{% note warning %}

Поведение Python SDK отличается от описанного выше.

{% endnote %}

1. Алгоритм определения режима аутентификации и необходимых параметров из переменных окружения в методе `construct_credentials_from_environ()` отличается от применяемого в других SDK:
   - Если задано значение переменной окружения `USE_METADATA_CREDENTIALS`, равное 1, то используется режим аутентификации **Metadata**
   - Иначе, если задано значение переменной окружения `YDB_TOKEN`, то используется режим аутентификации **Access Token**, в который передается значение данной переменной
   - Иначе, если задано значение переменной окружения `SA_KEY_FILE`, то используется режим аутентификации **System Account Key**, а ключ загружается из файла, имя которого указано в данной переменной
   - Иначе в запросах не будет добавлена информация об аутентификации.
2. В случае, если при инициализации драйвера не передан никакой объект, отвечающий за генерацию токенов, то применяется [общий порядок](#env) чтения значений переменных окружения.