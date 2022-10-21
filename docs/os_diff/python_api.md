---
sidebar_position: 7
title: Python API Reference
---

----

### `connect()`
`data_diff.connect(_db_conf: Union[str, dict], thread_count: Optional[int] = 1_)→ Database`


Connect to a database using the given database configuration.

Configuration can be given either as a URI string, or as a dict of {option: value}.

thread_count determines the max number of worker threads per database, if relevant. None means no limit.

#### Parameters

* **db_conf** (_str | dict_) – The configuration for the database to connect. URI or dict.
* **thread_count** (_int, optional_) – Size of the threadpool. Ignored by cloud databases. (default: 1)

Note: For non-cloud databases, a low thread-pool size may be a performance bottleneck.

Supported drivers: - postgresql - mysql - oracle - snowflake - bigquery - redshift - presto - databricks - trino - clickhouse - vertica

----

### `connect_to_table()`
`data_diff.connect_to_table(_db_info: Union[str, dict], table_name: Union[Tuple[str, ...], str], key_columns: str = ('id',), thread_count: Optional[int] = 1, **kwargs_)→ TableSegment`

Connects to the given database, and creates a TableSegment instance

#### Parameters

* **db_info** – Either a URI string, or a dict of connection options.
* **table_name** – Name of the table as a string, or a tuple that signifies the path.
* **key_columns** – Names of the key columns
* **thread_count** – Number of threads for this connection (only if using a threadpooled db implementation)

See also: `connect()`

----

### `diff_tables()`
`data_diff.diff_tables(table1: TableSegment, table2: TableSegment, *, key_columns: Optional[Sequence[str]] = None, update_column: Optional[str] = None, extra_columns: Optional[Tuple[str, ...]] = None, min_key: Optional[Union[int, str, bytes, ArithUUID, ArithAlphanumeric]] = None, max_key: Optional[Union[int, str, bytes, ArithUUID, ArithAlphanumeric]] = None, min_update: Optional[datetime] = None, max_update: Optional[datetime] = None, algorithm: Algorithm = Algorithm.HASHDIFF, bisection_factor: int = 32, bisection_threshold: int = 16384, threaded: bool = True, max_threadpool_size: Optional[int] = 1)→ Iterator`

Finds the diff between table1 and table2.

#### Parameters
* **key_columns** (_Tuple[str, ...]_) – Name of the key column, which uniquely identifies each row (usually id)
* **update_column** (_str, optional_) – Name of updated column, which signals that rows changed. Usually updated_at or last_update. Used by min_update and max_update.
* **extra_columns** (_Tuple[str, ...], optional_) – Extra columns to compare
* **min_key** (**DbKey**, optional) – Lowest key value, used to restrict the segment
* **max_key** (**DbKey**, optional) – Highest key value, used to restrict the segment
* **min_update** (**DbTime**, optional) – Lowest update_column value, used to restrict the segment
* **max_update** (**DbTime**, optional) – Highest update_column value, used to restrict the segment
* **algorithm** (**Algorithm**) – Which diffing algorithm to use (HASHDIFF or JOINDIFF)
* **bisection_factor** (_int_) – Into how many segments to bisect per iteration. (Used when algorithm is HASHDIFF)
* **bisection_threshold** (_Number_) – Minimal row count of segment to bisect, otherwise download and compare locally. (Used when algorithm is HASHDIFF).
* **threaded** (_bool_) – Enable/disable threaded diffing. Needed to take advantage of database threads.
* **max_threadpool_size** (_int_) – Maximum size of each threadpool. None means auto. Only relevant when _threaded_ is `True`. There may be many pools, so number of actual threads can be a lot higher.

Note: The following parameters are used to override the corresponding attributes of the given `TableSegment` instances: _key_columns_, _update_column_, _extra_columns_, _min_key_, _max_key_. If different values are needed per table, it’s possible to omit them here, and instead set them directly when creating each `TableSegment`.

#### Example

`table1 = connect_to_table('postgresql:///', 'Rating', 'id')
list(diff_tables(table1, table1))
[]`

See also: TableSegment, HashDiffer, JoinDiffer

----

### `HashDiffer()`
`class data_diff.HashDiffer(_threaded: bool = True, max_threadpool_size: (int+NoneType) = 1, bisection_factor: int = 32, bisection_threshold: Number = 16384, stats: dict[(Any*Any)] = <factory>_)`

Finds the diff between two SQL tables

The algorithm uses hashing to quickly check if the tables are different, and then applies a bisection search recursively to find the differences efficiently.

Works best for comparing tables that are mostly the same, with minor discrepencies.

#### Parameters

* **bisection_factor** (_int_) – Into how many segments to bisect per iteration.
* **bisection_threshold** (_Number_) – When should we stop bisecting and compare locally (in row count).
* **threaded** (_bool_) – Enable/disable threaded diffing. Needed to take advantage of database threads.
* **max_threadpool_size** (_int_) – Maximum size of each threadpool. None means auto. Only relevant when threaded is True. There may be many pools, so number of actual threads can be a lot higher.

#### `HashDiffer.__init()`

```
__init__(threaded: bool = True, max_threadpool_size: (int+NoneType) = 1, bisection_factor: int = 32, bisection_threshold: Number = 16384, stats: dict[(Any*Any)] = <factory>)→ None
```

#### `HashDiffer.diff_tables()`

```
diff_tables(table1: TableSegment, table2: TableSegment)→ Iterator[Tuple[str, tuple]]
```

Diff the given tables.

#### Parameters

* **table1** (TableSegment) – The “before” table to compare. Or: source table
* **table2** (TableSegment) – The “after” table to compare. Or: target table

#### Returns

An iterator that yield pair-tuples, representing the diff. Items can be either - (‘-’, row) for items in table1 but not in table2. (‘+’, row) for items in table2 but not in table1. Where row is a tuple of values, corresponding to the diffed columns.

----

### `JoinDiffer()`
`classdata_diff.JoinDiffer(threaded: bool = True, max_threadpool_size: (int+NoneType) = 1, validate_unique_key: bool = True, sample_exclusive_rows: bool = True, materialize_to_table: (NoneType+tuple[str]) = None, materialize_all_rows: bool = False, table_write_limit: int = 1000, stats: dict[(Any*Any)] = <factory>)`

Finds the diff between two SQL tables in the same database, using JOINs.

The algorithm uses an OUTER JOIN (or equivalent) with extra checks and statistics. The two tables must reside in the same database, and their primary keys must be unique and not null.

All parameters are optional.

#### Parameters

* **threaded** (_bool_) – Enable/disable threaded diffing. Needed to take advantage of database threads.
* **max_threadpool_size** (_int_) – Maximum size of each threadpool. None means auto. Only relevant when threaded is True. There may be many pools, so number of actual threads can be a lot higher.
* **validate_unique_key** (_bool_) – Enable/disable validating that the key columns are unique. Single query, and can’t be threaded, so it’s very slow on non-cloud dbs. Future versions will detect UNIQUE constraints in the schema.
* **sample_exclusive_rows** (_bool_) – Enable/disable sampling of exclusive rows. Creates a temporary table.
* **materialize_to_table** (_DbPath_) – Path of new table to write diff results to. Disabled if not provided.
* **table_write_limit** (_int_) – Maximum number of rows to write when materializing, per thread.

#### JoinDiffer.__init__()

```
__init__(threaded: bool = True, max_threadpool_size: (int+NoneType) = 1, validate_unique_key: bool = True, sample_exclusive_rows: bool = True, materialize_to_table: (NoneType+tuple[str]) = None, materialize_all_rows: bool = False, table_write_limit: int = 1000, stats: dict[(Any*Any)] = <factory>)→ None
```

#### JoinDiffer.diff_tables()

```
diff_tables(table1: TableSegment, table2: TableSegment)→ Iterator[Tuple[str, tuple]]
Diff the given tables.
```

#### Parameters

* **table1** (TableSegment) – The “before” table to compare. Or: source table
* **table2** (TableSegment) – The “after” table to compare. Or: target table

#### Returns
An iterator that yield pair-tuples, representing the diff. Items can be either - (‘-’, row) for items in table1 but not in table2. (‘+’, row) for items in table2 but not in table1. Where row is a tuple of values, corresponding to the diffed columns.

----

### `TableSegment()

`classdata_diff.TableSegment(database: Database = <object object>, table_path: tuple[str] = <object object>, key_columns: tuple[str] = <object object>, update_column: (NoneType+str) = None, extra_columns: tuple[str] = (), min_key: (NoneType+(str+bytes+int+ArithUUID+ArithAlphanumeric)) = None, max_key: (NoneType+(str+bytes+int+ArithUUID+ArithAlphanumeric)) = None, min_update: (NoneType+datetime) = None, max_update: (NoneType+datetime) = None, where: (NoneType+str) = None, case_sensitive: bool = True, _schema: (NoneType+CaseAwareMapping) = None)**`

Signifies a segment of rows (and selected columns) within a table

#### Parameters

* **database** (_Database_) – Database instance. See connect()
* **table_path** (**DbPath**) – Path to table in form of a tuple. e.g. (‘my_dataset’, ‘table_name’)
* **key_columns** (_Tuple[str]_) – Name of the key column, which uniquely identifies each row (usually id)
* **update_column** (_str_, optional) – Name of updated column, which signals that rows changed. Usually updated_at or last_update. Used by min_update and max_update.
* **extra_columns** (_Tuple_[str, ...], optional) – Extra columns to compare
* **min_key** (_DbKey_, optional) – Lowest key value, used to restrict the segment
* **max_key** (_DbKey_, optional) – Highest key value, used to restrict the segment
* **min_update** (_DbTime_, optional) – Lowest update_column value, used to restrict the segment
* **max_update** (_DbTime_, optional) – Highest update_column value, used to restrict the segment
* **where** (_str_, optional) – An additional ‘where’ expression to restrict the search space.
* **case_sensitive** (_bool_) – If false, the case of column names will adjust according to the schema. Default is true.

#### `TableSegment.with_schema()`
`with_schema()→ TableSegment`
Queries the table schema from the database, and returns a new instance of TableSegment, with a schema.

#### `TableSegment.get_values()`
`get_values()→ list`
Download all the relevant values of the segment from the database

#### `TableSegment.choose_checkpoints()`
`choose_checkpoints(count: int)→ List[Union[int, str, bytes, ArithUUID, ArithAlphanumeric]]`
Suggests a bunch of evenly-spaced checkpoints to split by (not including start, end)

#### `TableSegment.segment_by_checkpoints()`
`segment_by_checkpoints(checkpoints: List[Union[int, str, bytes, ArithUUID, ArithAlphanumeric]])→ List[TableSegment]`
Split the current TableSegment to a bunch of smaller ones, separated by the given checkpoints

#### `TableSegment.new()`
`new(**kwargs)→ TableSegment`
Using new() creates a copy of the instance using ‘replace()’

#### `TableSegment.count()`
 `count()→ Tuple[int, int]`
Count how many rows are in the segment, in one pass.

#### `TableSegment.count_and_checksum()`
`count_and_checksum()→ Tuple[int, int]`
Count and checksum the rows in the segment, in one pass.

#### `TableSegment.__init__()`
```
__init__(database: Database = <object object>, table_path: tuple[str] = <object object>, key_columns: tuple[str] = <object object>, update_column: (NoneType+str) = None, extra_columns: tuple[str] = (), min_key: (NoneType+(str+bytes+int+ArithUUID+ArithAlphanumeric)) = None, max_key: (NoneType+(str+bytes+int+ArithUUID+ArithAlphanumeric)) = None, min_update: (NoneType+datetime) = None, max_update: (NoneType+datetime) = None, where: (NoneType+str) = None, case_sensitive: bool = True, _schema: (NoneType+CaseAwareMapping) = None)→ None
```

----

### `AbstractDatabase`
`classdata_diff.databases.database_types.AbstractDatabase`

#### `AbstractDatabase.select_table_schema()`
_abstract_ `select_table_schema(path: Tuple[str, ...])→ str`
Provide SQL for selecting the table schema as (name, type, date_prec, num_prec)

#### `AbstractDatabase.query_table_schema()`
_abstract_ `query_table_schema(path: Tuple[str, ...])→ Dict[str, tuple]`
Query the table for its schema for table in ‘path’, and return {column: tuple} where the tuple is (table_name, col_name, type_repr, datetime_precision?, numeric_precision?, numeric_scale?)

#### `AbstractDatabase.parse_table_name()`
_abstract_ `parse_table_name(name: str)→ Tuple[str, ...]`
Parse the given table name into a DbPath

#### `AbstractDatabase.close()`
_abstract_ `close()`
Close connection(s) to the database instance. Querying will stop functioning.

#### `AbstractDatabase.normalize_value_by_type()`
`normalize_value_by_type(value: str, coltype: ColType)→ str`
Creates an SQL expression, that converts ‘value’ to a normalized representation.

The returned expression must accept any SQL value, and return a string.

The default implementation dispatches to a method according to coltype:

````
TemporalType    -> normalize_timestamp()
FractionalType  -> normalize_number()
*else*          -> to_string()

(`Integer` falls in the *else* category)
````

----

### `AbstractDialect`
_class_ `data_diff.databases.database_types.AbstractDialect`
Dialect-dependent query expressions

#### `AbstractDialect.quote()`
_abstract_ `quote(s: str)`
Quote SQL name

#### `AbstractDialect.concat()`
_abstract_ `concat(l: List[str])→ str`
Provide SQL for concatenating a bunch of columns into a string

#### `AbstractDialect.is_distinct_from()`
_abstract_ `is_distinct_from(a: str, b: str)→ str`
Provide SQL for a comparison where NULL = NULL is true

#### `AbstractDialect.to_string()`
_abstract_ `to_string(s: str)→ str`
Provide SQL for casting a column to string

#### `AbstractDialect.random()`
_abstract_ `random()→ str`
Provide SQL for generating a random number betweein 0..1

#### `AbstractDialect.offset_limit()`
_abstract_ `offset_limit(offset: Optional[int] = None, limit: Optional[int] = None)`
Provide SQL fragment for limit and offset inside a select

#### ``AbstractDialect.explain_as_text()`
_abstract_ `explain_as_text(query: str)→ str`
Provide SQL for explaining a query, returned as table(varchar)

#### `AbstractDialect.timestamp_value()`
_abstract_ `timestamp_value(t: datetime)→ str`
Provide SQL for the given timestamp value

----

### `DbKey`
`data_diff.DbKey`

The central part of internal API.

This represents a generic version of type ‘origin’ with type arguments ‘params’. There are two kind of these aliases: user defined and special. The special ones are wrappers around builtin collections and ABCs in collections.abc. These must have ‘name’ always set. If ‘inst’ is False, then the alias can’t be instantiated, this is used by e.g. typing.List and typing.Dict.

alias of `Union`[`int`, `str`, `bytes`, `ArithUUID`, `ArithAlphanumeric`]

----

### `DbTime`
`data_diff.DbTime= <class 'datetime.datetime'>)`

datetime(year, month, day[, hour[, minute[, second[, microsecond[,tzinfo]]]]])

The year, month and day arguments are required. tzinfo may be None, or an instance of a tzinfo subclass. The remaining arguments may be ints.

----

### `DbPath`
`data_diff.DbPath`
The central part of internal API.

This represents a generic version of type ‘origin’ with type arguments ‘params’. There are two kind of these aliases: user defined and special. The special ones are wrappers around builtin collections and ABCs in collections.abc. These must have ‘name’ always set. If ‘inst’ is False, then the alias can’t be instantiated, this is used by e.g. typing.List and typing.Dict.

alias of `Tuple`[`str`, …]

----

### `Algorithm`
`enum data_diff.Algorithm(value)`

An enumeration.

Valid values are as follows:

```
AUTO= <Algorithm.AUTO: 'auto'>
```

```
JOINDIFF= <Algorithm.JOINDIFF: 'joindiff'>
```

```
HASHDIFF= <Algorithm.HASHDIFF: 'hashdiff'>
```