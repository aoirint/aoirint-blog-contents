---
title: Django 3.xから4.xへの更新でPOSTリクエスト時にCSRF検証に失敗する
date: '2022-06-24 13:10:00'
updated: '2022-06-24 13:10:00'
draft: false
noindex: false
channel: 技術ノート
category: Python
tags:
  - Python
  - Django
---
# Django 3.xから4.xへの更新でPOSTリクエスト時にCSRF検証に失敗する

`settings.py`に`CSRF_TRUSTED_ORIGINS`を追加すればよい。

- Origin: e.g. `https://example.com`
  - <https://developer.mozilla.org/ja/docs/Glossary/Origin>

CSRF検証のドキュメントを3.xと比べると、以下の記述が増えている。

> CsrfViewMiddleware verifies the Origin header, if provided by the browser, against the current host and the CSRF_TRUSTED_ORIGINS setting. This provides protection against cross-subdomain attacks.

- 4.x: https://docs.djangoproject.com/en/4.0/ref/csrf/#how-it-works
- 3.x: https://docs.djangoproject.com/en/3.2/ref/csrf/#how-it-works
