# Note of ClickHouse

|时间|内容|
|:---|:---|
|20210331|kick off: data type, table engine.|

## 术语

<!-- 记录阅读过程中出现的关键字及其简单的解释. -->

## 介绍

<!-- 描述软件的来源、特性、解决的关键性问题等. -->

## 动机

<!-- 描述阅读软件源码的动机, 要达到什么目的等. -->

## 系统结构

<!-- 描述软件的系统结构, 核心和辅助组件的结构; 系统较复杂时细分展示. -->

## 使用

<!-- 记录软件如何使用. -->

### Install

``` shell
# https://github.com/ClickHouse/ClickHouse/blob/master/docker/server/README.md

# start server
docker run -d --name some-clickhouse-server --ulimit nofile=262144:262144 -p 8123:8123 -p 9000:9000 -p 9009:9009 --volume=D:/workspace/clickhouse/data:/var/lib/clickhouse yandex/clickhouse-server

# start server from root
docker run -d -e CLICKHOUSE_UID=0 -e CLICKHOUSE_GID=0 --name some-clickhouse-server --ulimit nofile=262144:262144 -p 8123:8123 -p 9000:9000 -p 9009:9009 --volume=D:/workspace/clickhouse/data:/var/lib/clickhouse yandex/clickhouse-server


# use client to load tutorial data
docker run -it --rm --link some-clickhouse-server:clickhouse-server --volume=D:/workspace/clickhouse/dataset:/var/data yandex/clickhouse-client --host clickhouse-server
/usr/bin/clickhouse-client --host clickhouse-server --query "INSERT INTO tutorial.hits_v1 FORMAT TSV" --max_insert_block_size=100000 < /var/data/hits_v1.tsv
/usr/bin/clickhouse-client --host clickhouse-server --query "INSERT INTO tutorial.visits_v1 FORMAT TSV" --max_insert_block_size=100000 < /var/data/visits_v1.tsv
```

### SQL
#### Data Types

|ID|Description|
|:---|:---|
|UInt8<br/>UInt16<br/>UInt32<br/>UInt64<br/>UInt256||
|Int8<br/>Int16<br/>Int32<br/>Int64<br/>Int128<br/>Int256|TINYINT, BOOL, BOOLEAN, INIT1<br/>SMALLINT, INT2<br/>INT, INT4, INTEGER<br/>BIGINT<br/>|
|Float32<br/>Float64|FLOAT<br/>DOUBLE|
|Decimal(P, S)|P: precision, in `[1:76]`<br/>S: scale, in `[0:P]` <br/>Decimal32(S): P in `[1:9]`<br/>Decimal64(S): P in `[10:18]`<br/>Decimal128(S): P in `[19:38]`<br/>Decimal256(S): P in `[39:76]`|
|Boolean|Uint8: 0, 1.|
|String|任意长度的字符串.|
|FixedString(N)|N个字节的字符串.|
|UUID|16个字节的universally unique identifier.|
|Date|日期, 距`1970-01-01`的天数的两个字节.|
|DateTime([timezone])|瞬时时间, 精度: 1秒, 范围`[1970-01-01 00:00:00, 2105-12-31 23:59:59]`.|
|DateTime64(precision, [timezone])|瞬时时间, 精度: 10^{-precision} 秒.|
|Enum|枚举: 8-bit `Enum8`, 16-bit `Enum16`.|
|LowCardinality(data_type)|将其他数据类型的表示修改为字典编码的. data_type可以是String, FixedString, Date, DateTime, 除Decimal之外的数值类型.|
|Array(T)|类型T的数组, T可以是任何类型(包括数组).|
|Nested(name1 Type1, Name2 Type2, ...)|一个单元中类似于表的内嵌数据结构.|
|Tuple(T1, T2, ...)|元组, 用于临时性的列分组.|
|Map(key, value)|存储`key:value`对.<br/>key: String, Integer<br/> value: String, Integer, Array.|
|Nullable(typename)|存储类型的缺失值`NULL`.|
|Special Data Types:||
|Expression|用于表示高阶函数中的lambda.|
|Set|用于作为`IN`表达式的右侧部分.|
|Nothing|表示非预期值的情况. 例: 字面量`NULL`的类型为`Nullable(Nothing)`.|
|Interval|`INTERVAL`操作符的结果类型: SECOND, MINUTE, HOUR, DAY, WEEK, MONTH, QUARTER, YEAR.|
|Domain<br/>IPv4<br/>IPv6|在基本的类型上添加额外特征的特殊用途类型.<br/>UInt32<br/>FixedString(16).|
|Multiword Types|Float64: DOUBLE PRECISION<br>String: CHAR LARGE OBJECT, CHAR VARYING, CHARACTER LARGE OBJECT, CHARACTER VARYING, NCHAR LARGE OBJECT, NCHAR VARYING, NATIONAL CHARACTER LARGE OBJECT, NATIONAL CHARACTER VARYING, NATIONAL CHAR VARYING, NATIONAL CHARACTER, NATIONAL CHAR, BINARY LARGE OBJECT, BINARY VARYING.|
|Geo|Point: Tuple(Float64, Float64)<br/>Ring: Array(Point)<br/>Polygon: Array(Ring)<br/>MultiPolygon: Array(Polygon).|
|AggregateFunction(name, types_of_arguments...)|聚合函数有实现定义的中间状态, 可以序列化为聚合函数数据类型, 通常用物化事务存储在表中. 使用`State`后缀生成聚合函数状态, 使用`Merge`后缀调用聚合函数获取结果.|
|SimpleAggregateFunction(name, types_of_arguments...)|存储聚合函数的当前值, 而不是AggregateFunction的完整状态. 可用于满足f(S1 UNION ALL S2) = f(f(S1) UNION ALL f(S2))的函数. 支持: any, anyLast, min, max, sum, sumWithOverflow, groupBitAnd, groupBitOr, groupBitXor, groupArrayArray, groupUniqArrayArray, sumMap, minMap, maxMap, argMin, argMax.|

#### Functions

#### SQL Syntax

表达式(expression): 函数、标识符、字面量、操作符应用、括号中表达式、子查询、`*`、别名. 一组表达式用`,`分隔. 函数和操作符可以使用表达式作为参数.


## 数据结构和算法

<!-- 描述软件中重要的数据结构和算法, 支撑过程部分的记录. -->

### Table Engines

#### MergeTree

|Name|Description|
|:---|:---|
|MergeTree|设计用于在一张表中插入大量的数据, 按part快速写入数据, 在后台应用规则合并part.|
|ReplacingMergeTree|移除排序键值相同的重复项.|
|SummingMergeTree|将具有相同排序键的行替换为包含数值类型列汇总值的一行.|
|AggregatingMergeTree|将具有相同排序键的行替换为存储了聚合函数的状态组合的一行.|
|CollapsingMergeTree|异步的删除/折叠具有相同的排序键, `sign`字段有值1和-1的行对. 保留无行对的行.|
|VersionedCollapsingMergeTree|使用`version`字段帮助恰当的折叠行.|
|GraphiteMergeTree|用于减少和聚合/平均Graphite数据.|
|Replicated*|支持数据复制.|

##### MergeTree

特性:

- 按主键(primary key)排序存储数据;
- 按分区键(partitioning key)将数据分区存储;
- 支持数据复制(replication);
- 支持数据采样(sampling).

创建表:

``` sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()                                    -- 引擎
ORDER BY expr                                             -- 排序键
[PARTITION BY expr]                                       -- 分区键
[PRIMARY KEY expr]                                        -- 主键: 不要求唯一性
[SAMPLE BY expr]                                          -- 采样表达式
[TTL expr                                                 -- 描述行存储周期的规则
    [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx' [, ...] ]
    [WHERE conditions]
    [GROUP BY key_expr [SET v1 = aggr_func(v1) [, v2 = aggr_func(v2) ...]] ] ]
[SETTINGS name=value, ...]                                -- 控制MergeTree行为的参数
```

SETTINGS参数:

|Name|Description|Default Value|
|:---|:---|:---|
|index_granularity|Maximum number of data rows between the marks of an index.|8192|
|index_granularity_bytes|Maximum size of data granules in bytes. <br/>To restrict the granule size only by number of rows, set to 0 (not recommended).|10Mb|
|min_index_granularity_bytes|Min allowed size of data granules in bytes. <br/>To provide a safeguard against accidentally creating tables with very low `index_granularity_bytes`.|1024b|
|enable_mixed_granularity_parts|Enables or disables transitioning to control the granule size with the `index_granularity_bytes` setting.<br/> Before version 19.11, there was only the `index_granularity` setting for restricting granule size. <br/>The `index_granularity_bytes` setting improves ClickHouse performance when selecting data from tables with big rows (tens and hundreds of megabytes). <br/>If you have tables with big rows, you can enable this setting for the tables to improve the efficiency of SELECT queries.||
|use_minimalistic_part_header_in_zookeeper|Storage method of the data parts headers in ZooKeeper. <br/>If use_minimalistic_part_header_in_zookeeper=1, then ZooKeeper stores less data.||
|min_merge_bytes_to_use_direct_io|The minimum data volume for merge operation that is required for using direct I/O access to the storage disk. <br/>When merging data parts, ClickHouse calculates the total storage volume of all the data to be merged. If the volume exceeds min_merge_bytes_to_use_direct_io bytes, ClickHouse reads and writes the data to the storage disk using the direct I/O interface (O_DIRECT option). <br/>If min_merge_bytes_to_use_direct_io = 0, then direct I/O is disabled.|10 * 1024 * 1024 * 1024 bytes|
|merge_with_ttl_timeout|Minimum delay in seconds before repeating a merge with TTL.|86400 (1 day)|
|write_final_mark|Enables or disables writing the final index mark at the end of data part (after the last byte). <br/>Don’t turn it off.|1|
|merge_max_block_size|Maximum number of rows in block for merge operations.|8192|
|storage_policy|Storage policy.||
|min_bytes_for_wide_part<br/> min_rows_for_wide_part|Minimum number of bytes/rows in a data part that can be stored in `Wide` format. <br/>You can set one, both or none of these settings.||
|max_parts_in_total|Maximum number of parts in all partitions.||
|max_compress_block_size|Maximum size of blocks of uncompressed data before compressing for writing to a table. <br/>You can also specify this setting in the global settings (see max_compress_block_size setting). The value specified when table is created overrides the global value for this setting.||
|min_compress_block_size|Minimum size of blocks of uncompressed data required for compression when writing the next mark. <br/>You can also specify this setting in the global settings (see min_compress_block_size setting). The value specified when table is created overrides the global value for this setting.||
|max_partitions_to_read|Limits the maximum number of partitions that can be accessed in one query. <br/>You can also specify setting max_partitions_to_read in the global setting.||

> From create table syntax and these settings, need to find out
>
> ttl, disk, volume
>
> mark of index
>
> data granules
>
> data parts
>
> merge operation

数据存储:

- 表由 ==data parts== 构成, data part中按主键排序.
- 向表中插入数据时, 创建独立的data parts, 每个data part中按主键字典序排序.
- 属于不同分区的数据被放置在不同的part中. 在后台合并data parts以实现高效的存储. 属于不同分区的parts不会被合并. 合并机制不会确保有相同主键的行在同一个data part中.
- data part的存储格式有`Wide`和`Compact`. 在`Wide`格式中, 每个列存储在文件系统的独立的文件中; 在`Compact`格式中, 所有列存储在一个文件中.
- 每个data part逻辑上划分为 ==granule== . granule是ClickHouse读选择数据时的最小不可拆分数据集.
- ClickHouse不会拆分行或值, 每个granule总是包含整数数量的行.
- granule的第一行被用该行的主键标记(mark).
- 对每个data part, ClickHouse创建存储mark的索引文件.
- 对每列, 不管是否在主键中, ClickHouse存储相同的mark.

存储配置:

``` xml
<!-- config.xml -->

<storage_configuration>
    <disks>
        <disk_name_1> <!-- disk name -->
            <path>/mnt/fast_ssd/clickhouse/</path>
        </disk_name_1>
        <disk_name_2>
            <path>/mnt/hdd1/clickhouse/</path>
            <keep_free_space_bytes>10485760</keep_free_space_bytes>
        </disk_name_2>
        <disk_name_3>
            <path>/mnt/hdd2/clickhouse/</path>
            <keep_free_space_bytes>10485760</keep_free_space_bytes>
        </disk_name_3>
    </disks>

    <policies>
        <policy_name_1> <!-- policy name -->
            <volumes>
                <volume_name_1> <!-- volume name -->
                    <disk>disk_name_from_disks_configuration</disk>
                    <max_data_part_size_bytes>1073741824</max_data_part_size_bytes>
                </volume_name_1>
                <volume_name_2>
                </volume_name_2>
            </volumes>
            <move_factor>0.2</move_factor>
        </policy_name_1>
        <policy_name_2>
            <prefer_not_to_merge>true</prefer_not_to_merge>
        </policy_name_2>
    </policies>
</storage_configuration>
```

##### ReplacingMergeTree

``` sql
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = ReplacingMergeTree([ver])
[PARTITION BY expr]
[ORDER BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

`ver`: 有版本的列, `Uint*`、`Date`或`DateTime`.

合并时从具有相同排序键的所有行中只保留一个:

- 未指定`ver`: selection的最后一个. selection是参与合并的一组part中的行集合, 最近创建的part是slection的最后一个.
- 指定了`ver`: 最大版本的行.

##### SummingMergeTree

``` sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = SummingMergeTree([columns])
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

`columns`: 列名称的元组, 这些列必须输数值类型且不在主键中. 如果没有指定, 会汇总所有不再主键中的数值类型列.

##### AggregatingMergeTree

``` sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = AggregatingMergeTree()
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[TTL expr]
[SETTINGS name=value, ...]
```

插入数据时使用`INSERT SELECT`和`<aggregate-function-name>State`函数;
查询数据时, 使用`GROUP BY`子句和`<aggregate-function-name>Merge`函数.

##### CollapsingMergeTree

``` sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = CollapsingMergeTree(sign)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

`sign`: 列的名称, 数据类型`Int8`. 值1标识状态行(state row), 值-1标识取消行(cancel row).


##### VersionedCollapsingMergeTree

``` sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = VersionedCollapsingMergeTree(sign, version)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

`version`: 标识状态对象版本的列名称, 数据类型`UInt*`. 如果该列不在主键中, ClickHouse默认将其作为主键的最后一列.

##### GraphiteMergeTree

``` sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    Path String,                              -- metric name
    Time DateTime,                            -- time
    Value <Numeric_type>,                     -- metric value
    Version <Numeric_type>                    -- metric version
    ...
) ENGINE = GraphiteMergeTree(config_section)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

`config_section`: 配置文件中的片段名称, 例: `graphite_rollup`

``` xml
<graphite_rollup>
    <path_column_name />                                <!-- 默认: Path -->
    <time_column_name />                                <!-- 默认: Time -->
    <value_column_name />                               <!-- 默认: Value -->
    <version_column_name>Version</version_column_name>  <!-- 默认: Timestamp -->
    <!-- pattern必须严格排序: (1) 无function或rentention的 (2) 有function和rentention的 (3) default -->
    <pattern>                                           
        <regexp>click_cost</regexp>                     <!-- metric名称模式 -->
        <function>any</function>                        <!-- 聚合函数, 数据的age在[age, age+precision] -->
        <retention>
            <age>0</age>                                <!-- 数据的最小age -->
            <precision>5</precision>                    <!-- 精度 -->
        </retention>
        <retention>
            <age>86400</age>
            <precision>60</precision>
        </retention>
    </pattern>
    <default>
        <function>max</function>
        <retention>
            <age>0</age>
            <precision>60</precision>
        </retention>
        <retention>
            <age>3600</age>
            <precision>300</precision>
        </retention>
        <retention>
            <age>86400</age>
            <precision>3600</precision>
        </retention>
    </default>
</graphite_rollup>
```

##### Replicated*

> Replication works at the level of an individual table, not the entire server. A server can store both replicated and non-replicated tables at the same time.
>
> Replication does not depend on sharding. Each shard has its own independent replication.
>
> Compressed data for INSERT and ALTER queries is replicated (for more information, see the documentation for ALTER).
>
> CREATE, DROP, ATTACH, DETACH and RENAME queries are executed on a single server and are not replicated:
>
> The CREATE TABLE query creates a new replicatable table on the server where the query is run. If this table already exists on other servers, it adds a new replica.
> The DROP TABLE query deletes the replica located on the server where the query is run.
> The RENAME query renames the table on one of the replicas. In other words, replicated tables can have different names on different replicas.

> sharding?

ZooKeeper设置:

``` xml
<zookeeper>
    <node>
        <host>example1</host>
        <port>2181</port>
    </node>
    <node>
        <host>example2</host>
        <port>2181</port>
    </node>
    <node>
        <host>example3</host>
        <port>2181</port>
    </node>
</zookeeper>

<!-- CREATE TABLE table_name ( ... )
ENGINE = ReplicatedMergeTree(
  'zookeeper_name_configured_in_auxiliary_zookeepers:path',
  'replica_name') ... -->
<auxiliary_zookeepers>
    <zookeeper2>
        <node>
            <host>example_2_1</host>
            <port>2181</port>
        </node>
        <node>
            <host>example_2_2</host>
            <port>2181</port>
        </node>
        <node>
            <host>example_2_3</host>
            <port>2181</port>
        </node>
    </zookeeper2>
    <zookeeper3>
        <node>
            <host>example_3_1</host>
            <port>2181</port>
        </node>
    </zookeeper3>
</auxiliary_zookeepers>
```



``` sql
CREATE TABLE table_name
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32,
    ver UInt16
)
-- {layer}-{shar}: shard标识符
-- '/clickhouse/tables/{layer}-{shard}/db_name.table_name'
-- '/clickhouse/tables/{layer}-{shard}/{database}/{table}' 内置占位符database, table
ENGINE = ReplicatedReplacingMergeTree(
  '/clickhouse/tables/{layer}-{shard}/table_name',    -- zoo_path: ZooKeeper中表路径
  '{replica}',                                        -- replica_name: ZooKeeper中副本名称
  ver                                                 -- other_parameters: 用于创建复制版本
)
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)
```

占位符值设置:

``` xml
<macros>
    <layer>05</layer>
    <shard>02</shard>
    <replica>example05-02-1.yandex.ru</replica>
</macros>
```



#### Log

|Name|Description|
|:---|:---|
|TinyLog||
|StripeLog||
|Log||

#### Integration Engines

|Name|Description|
|:---|:---|
|Kafka||
|MySQL||
|ODBC||
|JDBC||
|HDFS||
|S3||

### Special Engines

|Name|Description|
|:---|:---|
|Distributed||
|MaterializedView||
|Dictionary||
|Merge||
|File||
|Null||
|Set||
|Join||
|URL||
|View||
|Memory||
|Buffer||


## 过程

<!-- 描述软件中重要的过程性内容, 例如服务器的启动、服务器响应客户端请求、服务器背景活动等. -->

## 文献引用

<!-- 记录软件相关和进一步阅读资料: 文献、网页链接等. -->

https://clickhouse.tech/docs/en/

## 其他备注
