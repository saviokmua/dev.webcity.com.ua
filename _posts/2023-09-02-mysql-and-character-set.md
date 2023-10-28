---
layout: post
author: savio
title: Mysql and character set 
categories: ['database']
tags: ['db', 'mysql', 'table', 'column', 'character-set']
image:
  path: /assets/img/posts/mysql-logo.jpg
  alt: Mysql
---

# How to get character-set of database or table or column
###For Schemas/Databases

```sql
SELECT default_character_set_name FROM information_schema.SCHEMATA 
WHERE schema_name = "<database name>";
```

###For Tables

```sql
SELECT CCSA.character_set_name FROM information_schema.`TABLES` T,
       information_schema.`COLLATION_CHARACTER_SET_APPLICABILITY` CCSA
WHERE CCSA.collation_name = T.table_collation
  AND T.table_schema = "<database name>"
  AND T.table_name = "<table name>";
```

###For Columns:

```sql
SELECT character_set_name FROM information_schema.`COLUMNS` 
WHERE table_schema = "<database name>"
  AND table_name = "<table name>"
  AND column_name = "<column name>";
```

# How to change character-set

###For Schemas/Databases
```sql
ALTER DATABASE <database name> CHARACTER SET 'latin1';
```

###For Tables
```sql
ALTER DATABASE <database name>.<table name> CHARACTER SET 'latin1';
```

###For Columns
```sql
ALTER TABLE <database name>.<table name> MODIFY <column name> CHARACTER SET latin1;
```
