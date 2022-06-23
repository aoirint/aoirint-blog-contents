---
title: pip-compileでMetadataGenerationFailedの例外が起きる
date: '2022-06-24 07:10:00'
draft: false
channel: 技術ノート
category: Python
tags:
  - Python
---
# pip-compileでMetadataGenerationFailedの例外が起きる

pip-compileは、pip-toolsパッケージ（jazzband/pip-tools）に含まれるコマンドで、
Pythonパッケージのリスト`requirements.in`から、
依存ツリーのライブラリバージョンリスト`requirements.txt`を出力してくれる便利なツールである。

- <https://pypi.org/project/pip-tools/> （現在 pip-tools==6.6.2）
- <https://github.com/jazzband/pip-tools>

ところでPythonパッケージは、システムパッケージの事前インストールを要求することがある。
以下、システムパッケージ名はDebian/Ubuntuを想定する。

依存先が動的ライブラリであれば、ライブラリのimport時に例外を投げる作りになっている場合もあるが、
Pythonパッケージのインストールと同時にC言語コードからネイティブバイナリをコンパイルするような作りになっている場合には、
コンパイラや依存関係のヘッダファイルなどが必要になることがある。

このような依存関係が不足しているとき、pip-compileは以下のようなエラーログを出力する。

```
pip._internal.exceptions.MetadataGenerationFailed: metadata generation failed
```

ただし、このエラーログが出力されたとき、必ずしも原因が依存関係の不足とは限らない点は指摘しておく。
いまのところpip-compileには、パッケージのインストール処理が記述されたsetup.pyの実行時に例外が起きたとき、その例外の内容が隠されてしまう問題があり、他に原因がある可能性もある。
他に原因がありそうなときには、venvで仮想環境を作るなどして、一度通常のpip installを実行してみて、エラーログを確認して対処するとよいと思う。

- <https://github.com/jazzband/pip-tools/issues/1583>

ともあれ、このようなpipに管理されていない依存関係は、
ドキュメントが整備されている通常のPythonパッケージであれば、
PyPIやパッケージの公式ドキュメントに利用者向けの記載があるはずなので、
そちらを確認してインストールしてからpip-compileを実行すると解消するだろう。

例えば、PostgreSQLドライバのpsycopg2は、以下のようなパッケージの事前インストールが必要である。

- Cコンパイラ
- Pythonのヘッダ
- Postgresライブラリlibpqのヘッダ

```shell
sudo apt install python3-dev libpq-dev build-essential
```

- <https://pypi.org/project/psycopg2/> （現在 psycopg2==2.9.3）
- <https://www.psycopg.org/docs/install.html#build-prerequisites>

MySQLドライバのmysqlclientは、以下のようなパッケージの事前インストールが必要である。

```shell
sudo apt install python3-dev default-libmysqlclient-dev build-essential
```

- <https://pypi.org/project/mysqlclient/> （現在 mysqlclient==2.1.1）
