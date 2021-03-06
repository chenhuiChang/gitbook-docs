# 64.2. Built-in Operator Classes

The core PostgreSQL distribution includes the GIN operator classes shown in [Table 64.1](https://www.postgresql.org/docs/10/static/gin-builtin-opclasses.html#GIN-BUILTIN-OPCLASSES-TABLE). \(Some of the optional modules described in [Appendix F](https://www.postgresql.org/docs/10/static/contrib.html) provide additional GIN operator classes.\)

**Table 64.1. Built-in GIN Operator Classes**

| Name | Indexed Data Type | Indexable Operators |
| --- | --- | --- | --- | --- |
| `array_ops` | `anyarray` | `&&` `<@` `=` `@>` |
| `jsonb_ops` | `jsonb` | `?` `?&` `?|` `@>` |
| `jsonb_path_ops` | `jsonb` | `@>` |
| `tsvector_ops` | `tsvector` | `@@` `@@@` |

Of the two operator classes for type `jsonb`, `jsonb_ops` is the default. `jsonb_path_ops` supports fewer operators but offers better performance for those operators. See [Section 8.14.4](../../sql/datatype/8.14.-json-xing-bie.md#8-14-4-jsonbindexing) for details.

