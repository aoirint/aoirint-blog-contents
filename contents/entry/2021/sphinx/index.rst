.. article::
  :date: 2021-01-05 07:40:00
  :updated: 2021-11-13 18:00:00
  :category: Blogging
  :tags: Python, Sphinx, Documentation, Blogging
  :draft: false

###############################################
Sphinx
###############################################

Sphinx
=========================================

Python製のドキュメント生成ツール。


* Python 3.8.5
* Sphinx 3.4.2

  * `Sphinx · PyPI <https://pypi.org/project/Sphinx/>`_


.. code-block:: bash

   pip3 install Sphinx


reST（reStructuredText）
-----------------------------------------

SphinxではデフォルトでreStructuredTextというマークアップ言語を使う。

* https://www.sphinx-doc.org/ja/master/usage/restructuredtext/basics.html
* https://atom.io/packages/language-restructuredtext


既存Pythonプロジェクトにドキュメントを追加する
--------------------------------------------------

:code:`setup.py` などが存在するPyPIパッケージプロジェクトを想定する。

プロジェクトのルートに :code:`docs` ディレクトリを作成し、
インタラクティブツール :code:`sphinx-quickstart` を実行する。

.. code-block:: bash

   mkdir docs
   cd docs/
   sphinx-quickstart


:code:`Separate source and build directories (y/n)` と聞かれるので、 :code:`y` 。
プロジェクト名、著者名、言語などを答える。

:code:`source` ディレクトリ、 :code:`build` ディレクトリ、 :code:`Makefile` が生成される。

.. code-block:: bash

   make html

以上のコマンドで :code:`build/html` ディレクトリにHTMLが生成される。

.. code-block:: bash

   python3 -m http.server -b localhost -d build/html

などで確認する。


次はPythonモジュールのdocstringからドキュメントを自動生成する。

* `Sphinx でPythonのAPIドキュメントを自動作成 - Qiita <https://qiita.com/some-nyan/items/1980198a05c12d90e5c3>`_
* `python書くなら絶対に使いたい2つのドキュメント生成ツール - Qiita <https://qiita.com/hatsumi3/items/11c5bc835efe713e4767>`_

.. code-block:: bash

   #!/bin/bash

   SCRIPT_DIR=$(cd $(dirname $0); pwd)

   cd "${SCRIPT_DIR}"
   sphinx-apidoc -f -o "./source/api" "../mymodule"

   make html

:code:`docs/mkdocs.sh` を以上のように作成し、実行する（TODO：Makefileへの入れ込み）。
ここでは、以下のように :code:`docs` ディレクトリと並んで、パッケージとして提供するPythonモジュール :code:`mymodule` のディレクトリがあることを想定している。
なお、 :code:`docs/source/conf.py` は設定ファイルであり、
例えば :code:`html_theme = 'sphinx_rtd_theme'` のような設定を追加し、
:code:`pip3 install sphinx-rtd-theme` してから :code:`make html` することで、Read the Docsスタイルのドキュメントを生成できる。


.. code-block::

  |- setup.py
  |
  |- docs/
    |- Makefile
    |- mkdocs.sh
    |
    |- source/
      |- conf.py
      |- index.rst
    |
    |- build/
      |- html/
        |- index.html
  |
  |- mymodule/
    |- __init__.py
    |- mymodule.py

:code:`sphinx-apidoc` によりPythonモジュールのドキュメントが :code:`source/api` に自動生成され、
見出しにあたるページが :code:`source/api/modules.rst` に生成される。
このページへのリンクがどこにもない、という旨のエラーが表示されているはずなので、
インデックスページの :code:`source/index.rst` にこのページへのリンクを追加する。

.. code-block:: reStructuredText

  .. toctree::
    :maxdepth: 2
    :caption: Contents:

    api/modules


CI,CDを整備する
--------------------------------------------------

自動生成されるHTMLをソースコードと同じブランチでGit管理したくはないので、
:code:`docs/build` ディレクトリを :code:`.gitignore` に追加し、
GitHub ActionsやGitLab CIを使ってドキュメントの生成、GitHub PagesやGitLab Pagesへの自動デプロイを整備する。

Docker化の例
--------------------------------------------------

* https://github.com/aoirint/sphinx-docs-test
