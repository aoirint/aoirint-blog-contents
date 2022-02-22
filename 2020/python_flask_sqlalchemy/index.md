---
# moved from https://aoirint.hatenablog.com/entry/2020/03/16/144207
title: Flask, sqlalchemy
date: '2020-03-16 14:42:07'
draft: false
channel: 技術ノート
category: Python
tags:
- Python
- Markdown
- SQL
- Web
---
# Flask, sqlalchemy

### Flask
- [ウェブアプリケーションフレームワーク Flask を使ってみる - Qiita](https://qiita.com/ynakayama/items/2cc0b1d3cf1a2da612e4)
- [Flaskで任意のデータを引数として受け取る方法 - Qiita](https://qiita.com/5zm/items/c8384aa7b7aae924135c)
- [python - redirect while passing arguments - Stack Overflow](https://stackoverflow.com/questions/17057191/redirect-while-passing-arguments)
- [#url_for API — Flask 0.10.1 documentation](https://flask-doc.readthedocs.io/en/latest/api.html?highlight=url_for#flask.url_for)
- [python - How to serve static files in Flask - Stack Overflow](https://stackoverflow.com/questions/20646822/how-to-serve-static-files-in-flask)
- [python - Unable to retrieve files from send_from_directory() in flask - Stack Overflow](https://stackoverflow.com/questions/17681762/unable-to-retrieve-files-from-send-from-directory-in-flask)
- [python - Flaskで変数を文字列ではなくHTMLのタグとして読み込ませたい - スタック・オーバーフロー](https://ja.stackoverflow.com/questions/46630/flask%E3%81%A7%E5%A4%89%E6%95%B0%E3%82%92%E6%96%87%E5%AD%97%E5%88%97%E3%81%A7%E3%81%AF%E3%81%AA%E3%81%8Fhtml%E3%81%AE%E3%82%BF%E3%82%B0%E3%81%A8%E3%81%97%E3%81%A6%E8%AA%AD%E3%81%BF%E8%BE%BC%E3%81%BE%E3%81%9B%E3%81%9F%E3%81%84)

### sqlalchemy
- [【PythonのORM】SQLAlchemyで基本的なSQLクエリまとめ - Qiita](https://qiita.com/tomo0/items/a762b1bc0f192a55eae8)
- [Object Relational Tutorial — SQLAlchemy 1.3 Documentation](https://docs.sqlalchemy.org/en/13/orm/tutorial.html)
- [Python: SQLAlchemy + mysqlclient (MySQLdb) でマルチバイト文字を扱う - CUBE SUGAR CONTAINER](https://blog.amedama.jp/entry/2016/06/07/234106)
- [Transactions and Connection Management — SQLAlchemy 1.3 Documentation](https://docs.sqlalchemy.org/en/13/orm/session_transaction.html)
- [Object Relational Tutorial — SQLAlchemy 1.3 Documentation](https://docs.sqlalchemy.org/en/13/orm/tutorial.html)
- [SQLAlchemy で Delete するには？ - Qiita](https://qiita.com/nskydiving/items/eedd5cea88b5afdbfc49)
- [Basic Relationship Patterns — SQLAlchemy 1.3 Documentation](https://docs.sqlalchemy.org/en/13/orm/basic_relationships.html)
- [python - How to remove all items from many-to-many collection in SqlAlchemy? - Stack Overflow](https://stackoverflow.com/questions/7888900/how-to-remove-all-items-from-many-to-many-collection-in-sqlalchemy)
- [python - Flask-SQLAlchemy - model has no attribute 'foreign_keys' - Stack Overflow](https://stackoverflow.com/questions/19205290/flask-sqlalchemy-model-has-no-attribute-foreign-keys)
- [python - sqlalchemy dynamic filtering - Stack Overflow](https://stackoverflow.com/questions/41305129/sqlalchemy-dynamic-filtering)

### tagify
- [Tagify - demo](https://yaireo.github.io/tagify/)

### python-markdown
- [Library Reference — Python-Markdown 3.2.1 documentation](https://python-markdown.github.io/reference/)
- [Extensions — Python-Markdown 3.2.1 documentation](https://python-markdown.github.io/extensions/)

### MathJax
- [Getting Started — MathJax 2.7 documentation](https://docs.mathjax.org/en/v2.7-latest/start.html)
- [Combined Configurations — MathJax 2.7 documentation](https://docs.mathjax.org/en/v2.7-latest/config-files.html#common-configurations)

### highlight.js
- [How to use highlight.js](https://highlightjs.org/usage/)
- [highlight.js demo](https://highlightjs.org/static/demo/)
- [highlight.js/src/styles at master · highlightjs/highlight.js](https://github.com/highlightjs/highlight.js/tree/master/src/styles)
- [highlightjs-line-numbers.js - cdnjs.com - The best FOSS CDN for web related libraries to speed up your websites!](https://cdnjs.com/libraries/highlightjs-line-numbers.js)
- [wcoder/highlightjs-line-numbers.js: Line numbering plugin for Highlight.js](https://github.com/wcoder/highlightjs-line-numbers.js)

### Ace Editor
- [Ace - The High Performance Code Editor for the Web](https://ace.c9.io/#nav=embedding&api=editor)
- [ace/lib/ace/theme at master · ajaxorg/ace](https://github.com/ajaxorg/ace/tree/master/lib/ace/theme)
- [javascript - How do I get value from ACE editor? - Stack Overflow](https://stackoverflow.com/questions/8963855/how-do-i-get-value-from-ace-editor)

### Unique/Random ID
- [[Python] UUIDを生成するuuid.uuid4()はどうやってUUIDを生成しているのか？ ｜ Developers.IO](https://dev.classmethod.jp/server-side/how-generate-uuid-python-uuid4/)
  - [#os.urandom os --- 雑多なオペレーティングシステムインタフェース — Python 3.8.2 ドキュメント](https://docs.python.org/ja/3/library/os.html#os.urandom)

```python
os.urandom(NUM_BYTES).hex()
```
