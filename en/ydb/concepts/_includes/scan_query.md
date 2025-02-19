---
sourcePath: en/ydb/ydb-docs-core/en/core/concepts/_includes/scan_query.md
---
# Scan Queries in Yandex Database

*Scan Queries* is a separate data access interface designed primarily for performing analytical ad hoc queries on a DB.

This method of executing queries has the following unique features:

* Only *Read-Only* queries.
* In *SERIALIZABLE_RW* mode, a data snapshot is taken and then used for all subsequent operations. As a result, the impact on OLTP transactions is minimal (only taking a snapshot).
* The result of the query is a data stream ([grpc stream](https://grpc.io/docs/what-is-grpc/core-concepts/)). This means scan queries have no limit on the number of rows in the result.
* Due to the high overhead, it is only suitable for ad hoc queries.

{% node info %}

From the *Scan Queries* interface, you can query [system tables](../../troubleshooting/system_views.md).

{% endnote %}

Scan queries cannot currently be considered an effective solution for running OLAP queries due to their technical limitations (which will be removed in time):

* The query duration is limited to 5 minutes.
* Many operations (including sorting) are performed entirely in memory, which may lead to resource shortage errors when running complex queries.
* Only one strategy is currently used for Joins: *MapJoin* (a.k.a. *Broadcast Join*), where the "right" table is converted to a map, so it should be no more than units of gigabytes.
* Prepared form isn't supported, so for each call, a query is compiled.
* There is no optimization for point reads or reading small ranges of data.
* The SDK doesn't support automatic retry.

{% note info %}

Despite the fact that *Scan Queries* obviously don't interfere with the execution of OLTP transactions, they still use common DB resources: CPU, memory, disk, and network. Therefore, running complex queries **may lead to resource hunger**, which will affect the performance of the entire DB.

{% endnote %}

## How do I use it?

Like other types of queries, *Scan Queries* are available via the [management console]({{ link-console-main }}) (the query must specify `PRAGMA Kikimr.ScanQuery = "true";`), [CLI](../../reference/ydb-cli/commands/scan-query.md), and [SDK](../../reference/ydb-sdk/index.md).

