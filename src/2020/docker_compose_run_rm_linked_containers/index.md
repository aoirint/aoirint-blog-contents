---
title: 'docker-compose run --rm終了時に他のコンテナも削除する'
# og_image:
# twitter_card: summary_large_image
og_description: 'docker-compose run --rm終了時に他のコンテナも削除する'
date: '2020-09-28T10:10:00+09:00'
draft: false
channel: 技術ノート
category: Docker
tags:
  - Docker
  - Docker Compose
---
# docker-compose run --rm終了時に他のコンテナも削除する

`docker-compose run --rm app`を実行するとサービスappが起動してコンテナが作られ、実行が終わり次第コンテナは削除される。
このとき削除されるコンテナはforegroundで実行されたもののみで、
`depends_on`などの設定からリンクを通じて起動した他サービスのdetachedコンテナは削除されず残り続ける。

- [docker-compose run &lt;container&gt; --rm does not rm links · Issue #2791 · docker/compose](https://github.com/docker/compose/issues/2791 "docker-compose run <container> --rm does not rm links · Issue #2791 · docker/compose")

このissueでlinked containersを同時に削除する機能について議論されているが、
`run`コマンドにオプションを増やしたくない、ということで却下されている。

> As mentioned in this thread, docker-compose rm -f --all and docker-compose down already allow cleaning up a project's containers / resources. We won't overload run with more options at that point.

ひとまず`docker-compose run`終了後に`docker-compose down`を実行するシェルスクリプトで対応する。

compose_run.sh

```shell
#!/bin/bash

docker-compose run --rm "$@"
EXIT_CODE=$?
docker-compose down
exit $EXIT_CODE
```
