---
title: 'PostgreSQL 13 ユーザ（ロール）のパスワード変更'
date: '2022-02-06T09:36:47+09:00'
draft: false
noindex: false
channel: 技術ノート
category: PostgreSQL
tags:
  - PostgreSQL
---
# PostgreSQL 13 ユーザ（ロール）のパスワード変更

```shell
psql --user myuser postgres
```

```sql
ALTER ROLE myuser WITH PASSWORD 'new_password';
```

- [https://www.postgresql.jp/document/13/html/sql-alterrole.html](https://www.postgresql.jp/document/13/html/sql-alterrole.html)
- [https://www.postgresql.org/docs/13/sql-alterrole.html](https://www.postgresql.org/docs/13/sql-alterrole.html)
- [https://stackoverflow.com/questions/12720967/how-to-change-postgresql-user-password](https://stackoverflow.com/questions/12720967/how-to-change-postgresql-user-password)
