# 51.79. pg\_replication\_origin\_status

The `pg_replication_origin_status` view contains information about how far replay for a certain origin has progressed. For more on replication origins see [Chapter 49](https://www.postgresql.org/docs/10/static/replication-origins.html).

**Table 51.80. `pg_replication_origin_status` Columns**

| Name | Type | References | Description |
| --- | --- | --- | --- | --- |
| `local_id` | `Oid` | [`pg_replication_origin`](https://www.postgresql.org/docs/10/static/catalog-pg-replication-origin.html).roident | internal node identifier |
| `external_id` | `text` | [`pg_replication_origin`](https://www.postgresql.org/docs/10/static/catalog-pg-replication-origin.html).roname | external node identifier |
| `remote_lsn` | `pg_lsn` |   | The origin node's LSN up to which data has been replicated. |
| `local_lsn` | `pg_lsn` |   | This node's LSN at which `remote_lsn` has been replicated. Used to flush commit records before persisting data to disk when using asynchronous commits. |

