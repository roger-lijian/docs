##TiDB Package Usage Instructions
* ast 

    TiDB Abstract Syntax Tree 相关的定义和实现，包括 ExprNode / DDLNode / DMLNode / FuncNode / StmtNode 等

* column

    TiDB Table Column 相关的定义和实现

* context

    TiDB context 相关的 interface 定义，主要用于 Transaction 和 Args 处理等

* ddl

    TiDB Online DDL 相关的定义和实现

* docs

    TiDB 相关文档

* domain

    Domain Namespace，主要用于映射 store 成独立的环境，用于并发测试

* executor

    新的 TiDB SQL Executor 的定义和实现 (WIP)

* converter

    老的 SQL Executor Converter，主要用于支持新的 SQL Executor 开发和老的 SQL Executor 的兼容

* expression

    老的 Expr Interface 的定义和实现，以后会被移除

    - builtin

        builtin function 的实现

    - subquery

        subquery 的实现

* field

    TiDB Field 和 ResultField 相关的定义和实现

* infoschema

    TiDB Information Schema 相关的定义和实现

* inspectkv

    TiDB SQL Data 和 KV 辅助 check 包，以后会用于外部对于 TiDB KV 的访问，将被重新定义和扩展

* interpreter

    TiDB interpreter 的实现，主要用于 TiDB 的 REPL，不再开发新的功能

* kv

    KV 相关的接口定义和部分实现，包括 Retriever / Mutator / Transaction / Snapshot / Storage / Iterator 等

    - memkv

        B-tree Memory KV 的实现

* meta

    meta information相关

    - autoid

        auto increment allocator 实现

* metric

    metrics 统计数据信息

* model

    TiDB 支持的 ddl / dml 相关的数据结构，包括 DBInfo / TableInfo / ColumnInfo / IndexInfo 等

* mysql

    TiDB 支持的各种兼容 MySQL 的数据类型的定义和实现

* optimizer

    新的 TiDB SQL Optimizer 实现 (WIP)

    - evaluator

        ExprNode Evaluation 实现

    - plan

        执行计划的定义和实现

* parser

    parser / lexer 文件定义

    - coldef

        主要用于Parse SQL Column

    - opcode

        主要用于Parse SQL Operator

* plan

    老的 SQL Executing Plan 的定义，从 Stament 中生成 Plan，以后会被移除

    - plans

        老的 Plan 实现，包括 fields / from / where / groupby / having / limit 等，以后会被移除

* privilege

    privilege 相关

    - privileges

        privilege 相关的实现

* rset

    老的 Recordset interface的定义，用于从老的 Plan 中获取数据，以后会被移除

    - rsets

        老的 Recordset 实现，包括 fields / from / where / groupby / having / limit 等，以后会被移除

* sessionctx

    session context相关

    - autocommit

        autocommit 相关的 context

    - db

        db 相关的 context，主要用于 current db 等

    - forupdate

        forupdate 相关的 context，主要用于 select for update
    
    - variable

        variable 相关的 context，主要用于 session / global variables 的处理

* stmt

    老的 SQL Statement interface的定义，SQL 被 parse 成 Statement, 以后会被移除

    - stmts

        老的 SQL Statement 实现，包括 insert / update /delete / select 等，以后会被移除

* store

    TiDB Store 相关

    - hbase

        基于 hbase 的分布式 store

    - localstore

        单机 store

        - boltdb

            基于 boltdb 的 local store engine 实现

        - engine

            local store engine interface 定义

        - goleveldb

            基于 goleveldb 的 local store engine 实现

* structure

    更丰富的支持 Transaction的 KV 类型，包括 string / list / hash 等

* table

    TiDB table 相关的 interface 定义，包括 Row 的增删改查等

    - tables

        TiDB table 定义的 interface 的具体实现

* terror

    TiDB 最基本的 ErrorClass 和 ErrorCode 定义

* tidb-server

    TiDB Server

    - server

        TiDB SQL Layer Server，包括 MySQL 协议解析，连接和数据处理等

* util

    TiDB 常用的 Utility 包

    - arena

        内存分配器，减少内存分配

    - bytes

        bytes 相关的功能函数，以后可能会被移除

    - charset

        charset / collation 相关

    - codec

        支持对 bytes / decimal / float / number 等类型的 codec，主要用于 SQL 映射 KV 的 key/value codec

    - distinct

        主要用于支持 distinct 语句

    - format

        主要用于支持 explain 语句的显示信息，以后可能会被移除

    - hack

        主要用于优化 []byte 和 string 之间的转化，不需要再分配额外的内存， 以后可能会被移除

    - mock

        主要用于支持 Unit Test

    - printer

        主要用于支持 Interpreter 的信息显示

    - segmentmap

        使用 map slice 替换一个大的 map，用于在并发场景下提升处理效率

    - sqlexec

        restricted sql executor，主要用于在一个 statement 中方便地执行其他 statement

    - stringutil

        string 相关的功能函数，以后可能会被移除

    - testkit

        一个简单的 test framework

    - testutil

        test 相关的功能函数，以后可能会被移除

    - types

        主要用于支持不同数据类型的比较，转化，计算等

