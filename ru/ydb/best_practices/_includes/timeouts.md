---
title: Использование таймаутов в YDB
description: 'Значение operation_timeout определяет время, в течение которого результат запроса интересен пользователю. Если за данное время операция не выполнилась, сервер возвращает ошибку c кодом Timeout и попытается прекратить выполнение запроса, однако отмена запроса не гарантируется. Всегда рекомендуется устанавливать и таймаут на операцию, и транспортный таймаут. '
sourcePath: ru/ydb/ydb-docs-core/ru/core/best_practices/_includes/timeouts.md
---

# Использование таймаутов

В разделе приведено описание доступных таймаутов и представлены примеры использования на различных языках программирования.

## Предпосылки к использованию таймаутов {#intro}

Механизм таймаутов в {{ ydb-short-name }} призван решить следующие проблемы:

* Не дать запросу выполнятся так долго, что результат запроса становится не интересен для дальнейшего использования.
* Обнаружить проблемы сетевой связности.

Оба этих случая важны для обеспечения отказоустойчивости системы в целом. Рассмотрим таймауты подробнее.

## Таймаут на операцию {#operational}

Значение ``operation_timeout`` определяет время, в течение которого результат запроса интересен пользователю. Если за данное время операция не выполнилась, сервер возвращает ошибку c кодом ``Timeout`` и попытается прекратить выполнение запроса, однако отмена запроса не гарантируется. Таким образом, запрос, на который пользователю была возвращена ошибка ``Timeout``, может быть как успешно выполнен на сервере, так и отменен.

## Таймаут отмены операции {#cancel}

Значение ``cancel_after`` определяет время, через которое сервер начнет отмену запроса, если отменить запрос возможно. В случае успешной отмены запроса сервер вернет ошибку с кодом ``Cancelled``.

## Транспортный таймаут {#transport}

На каждый запрос клиент должен выставить транспортный таймаут. Данное значение позволяет определить количество времени, которое клиент готов ждать ответа от сервера. Если за данное время сервер не ответил, то клиенту будет возвращена транспортная ошибка c кодом ``DeadlineExceeded``. Важно выставить такое значение клиентского таймаута чтоб при нормальной работе приложения и сети транспортные таймауты не срабатывали.

## Применение таймаутов {#usage}

Всегда рекомендуется устанавливать и таймаут на операцию и транспортный таймаут. Значение транспортного таймаута следует делать на 50-100 миллисекунд больше чем значение таймаута на операцию, чтобы оставался некоторый запас времени, за который клиент сможет получить серверную ошибку c кодом ``Timeout``.

Пример использования таймаутов:

{% list tabs %}

- Python

  ```python
  from kikimr.public.sdk.python import client as ydb

  def execute_in_tx(session, query):
    settings = ydb.BaseRequestSettings()
    settings = settings.with_timeout(0.5)  # transport timeout
    settings = settings.with_operation_timeout(0.4)  # operation timeout
    settings = settings.with_cancel_after(0.4)  # cancel after timeout
    session.transaction().execute(
        query,
        commit_tx=True,
        settings=settings,
    )
  ```


- Go

  ```go
  import (
      "context"
      "a.yandex-team.ru/kikimr/public/sdk/go/ydb"
      "a.yandex-team.ru/kikimr/public/sdk/go/ydb/table"
  )

  func executeInTx(ctx context.Context, s *table.Session, query string) {
      newCtx, close := context.WithTimeout(ctx, time.Millisecond*300)         // client and by default operation timeout
      newCtx2 := ydb.WithOperationTimeout(newCtx, time.Millisecond*400)       // operation timeout override
      newCtx3 := ydb.WithOperationCancelAfter(newCtx2, time.Millisecond*300)  // cancel after timeout
      defer close()
      tx := table.TxControl(table.BeginTx(table.WithSerializableReadWrite()), table.CommitTx())
      _, res, err := session.Execute(newCtx3, tx, query)
  }
  ```

{% endlist %}
