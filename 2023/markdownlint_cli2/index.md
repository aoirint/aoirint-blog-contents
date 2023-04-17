---
title: 'markdownlint-cli2'
date: '2023-04-17T13:00:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: 'Command Utility'
tags:
  - 'Command Utility'
  - Markdown
---
# markdownlint-cli2

- Node.js 18.16.0
- markdownlint-cli2 0.6.0
  - [https://www.npmjs.com/package/markdownlint-cli2](https://www.npmjs.com/package/markdownlint-cli2)

```shell
npm install -g markdownlint-cli2
```

## Config .markdownlint-cli2.yaml

- Example: [https://github.com/DavidAnson/markdownlint/blob/fcb8190781c80b292ac44f6df984326e0e6c69cd/schema/.markdownlint.yaml](https://github.com/DavidAnson/markdownlint/blob/fcb8190781c80b292ac44f6df984326e0e6c69cd/schema/.markdownlint.yaml)

```yaml
# https://github.com/DavidAnson/markdownlint
# https://github.com/DavidAnson/markdownlint-cli2

globs:
  - "**/*.md"

ignores:
  - ".git/**"
  - ".github/**"

config:
  # h1
  MD025: false
  # inline HTML
  MD033: false
```
