# Database

## Basis

- Notion: Schema/Attribue/Tuple, OLAP/OLTP
- Classification: SQL/NoSQL/NewSQL
- Architecture: Standalone/Sharding/Distributed
- E-R Model
- Normalization/Functional Dependency/Closure
- Join/Sort

## Index

- Classification
  - B+ tree/Hash Index
  - Clustered(Primary)/Non-Clustered(Secondary) Index
  - Primary/Unique/Prefix Index
  - Multiple-Column Index
- Why use B+ tree?
  - Difference between page in db and page in os
  - BST->AVL->B Tree->B+ Tree
- Leftmost Prefix Principle
- Index Condition Pushdown
- Index Failure

## SQL

## Transcation

## Concurrency Control

## MySQL

- Data Types
  - INT/FLOAT/DOUBLE/DECIMAL
  - DATE/TIME/DATETIME/TIMESTAMP/YEAR
  - CHAR/VARCHAR/TEXT/BLOB
- Function
  - COUNT(*) vs COUNT(1) vs COUNT(col_name)
  - CONCAT()
- Architecture: Client/Server/Storage
- Workflow
  - Connection: tcp(jdbc/local sql command line interface)
  - Paser: ast
  - Optimizer
  - Executor
- Storage Engine
  - InnoDB: default engine, supports transcation, locking on row level and foreign key, uses clustered index and stores data and index together
  - MyISAM: does not support transcation, only supports locking on table level, uses non-clustered index and stores data and index separately
  - Buffer Pool: cache page & lru list
