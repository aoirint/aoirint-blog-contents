---
title: 'PostgreSQL 13 データベースの名前変更'
date: '2022-02-06T09:36:47+09:00'
draft: false
noindex: false
channel: 技術ノート
category: PostgreSQL
tags:
  - PostgreSQL
---
# PostgreSQL 13 データベースの名前変更

```shell
psql --user myuser postgres
```

```sql
ALTER DATABASE mydatabase RENAME TO new_mydatabase;
```

- [https://www.postgresql.jp/document/13/html/sql-alterdatabase.html](https://www.postgresql.jp/document/13/html/sql-alterdatabase.html)
- [https://www.postgresql.org/docs/13/sql-alterdatabase.html](https://www.postgresql.org/docs/13/sql-alterdatabase.html)
- [https://www.postgresqltutorial.com/postgresql-rename-database/](https://www.postgresqltutorial.com/postgresql-rename-database/)
