---
title: Mastodon on Docker Composeで全文検索エンジンElasticsearchを有効化する
date: '2022-11-19 18:00:00'
draft: false
noindex: false
channel: 技術ノート
category: Mastodon
tags:
  - GitHub
  - Python
---
# Mastodon on Docker Composeで全文検索エンジンElasticsearchを有効化する

- Mastodon on Docker Composeを立てる記事: <https://blog.aoirint.com/entry/2020/mastodon_docker/>

上の記事では、諸般の事情によりElasticsearchを無効化していた。


## Elasticsearchのライセンス変更

- SSPL
- マネージドサービスの提供に関する制限


## GeoIPの更新エラー

- <https://www.elastic.co/guide/en/elasticsearch/reference/current/geoip-processor.html>

```
{"@timestamp":"2022-11-19T08:03:05.924Z", "log.level":"ERROR", "message":"exception during geoip databases update", "ecs.version": "1.2.0","service.name":"ES_ECS","event.dataset":"elasticsearch.server","process.thread.name":"elasticsearch[86daa7202fdd][generic][T#3]","log.logger":"org.elasticsearch.ingest.geoip.GeoIpDownloader","elasticsearch.cluster.uuid":"hheOc5duSHeHSeLuX2UJuw","elasticsearch.node.id":"7Bqa7bvcRvq_xxEEyVwflw","elasticsearch.node.name":"86daa7202fdd","elasticsearch.cluster.name":"es-mastodon","error.type":"java.net.SocketTimeoutException","error.message":"Connect timed out","error.stack_trace":"java.net.SocketTimeoutException: Connect timed out\n\tat java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:539)\n\tat java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:585)\n\tat java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)\n\tat java.base/java.net.Socket.connect(Socket.java:666)\n\tat java.base/sun.security.ssl.SSLSocketImpl.connect(SSLSocketImpl.java:304)\n\tat java.base/sun.net.NetworkClient.doConnect(NetworkClient.java:178)\n\tat java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:531)\n\tat java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:636)\n\tat java.base/sun.net.www.protocol.https.HttpsClient.<init>(HttpsClient.java:264)\n\tat java.base/sun.net.www.protocol.https.HttpsClient.New(HttpsClient.java:378)\n\tat java.base/sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.getNewHttpClient(AbstractDelegateHttpsURLConnection.java:193)\n\tat java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect0(HttpURLConnection.java:1241)\n\tat java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect(HttpURLConnection.java:1127)\n\tat java.base/sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.connect(AbstractDelegateHttpsURLConnection.java:179)\n\tat java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream0(HttpURLConnection.java:1661)\n\tat java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream(HttpURLConnection.java:1585)\n\tat java.base/java.net.HttpURLConnection.getResponseCode(HttpURLConnection.java:529)\n\tat java.base/sun.net.www.protocol.https.HttpsURLConnectionImpl.getResponseCode(HttpsURLConnectionImpl.java:308)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.HttpClient.lambda$get$0(HttpClient.java:46)\n\tat java.base/java.security.AccessController.doPrivileged(AccessController.java:569)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.HttpClient.doPrivileged(HttpClient.java:88)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.HttpClient.get(HttpClient.java:40)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.HttpClient.getBytes(HttpClient.java:36)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloader.fetchDatabasesOverview(GeoIpDownloader.java:155)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloader.updateDatabases(GeoIpDownloader.java:143)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloader.runDownloader(GeoIpDownloader.java:274)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloaderTaskExecutor.nodeOperation(GeoIpDownloaderTaskExecutor.java:102)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloaderTaskExecutor.nodeOperation(GeoIpDownloaderTaskExecutor.java:48)\n\tat org.elasticsearch.server@8.5.1/org.elasticsearch.persistent.NodePersistentTasksExecutor$1.doRun(NodePersistentTasksExecutor.java:42)\n\tat org.elasticsearch.server@8.5.1/org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingAbstractRunnable.doRun(ThreadContext.java:892)\n\tat org.elasticsearch.server@8.5.1/org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:26)\n\tat java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144)\n\tat java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)\n\tat java.base/java.lang.Thread.run(Thread.java:1589)\n"}
```


## Elasticsearchのパスワード設定

docker-compose.yml

```yaml
  es:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.5.1
    restart: always
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "cluster.name=es-mastodon"
      - "discovery.type=single-node"
      - "bootstrap.memory_lock=true"
    networks:
      - internal_network
    healthcheck:
      test: ["CMD-SHELL", "curl --silent --fail localhost:9200/_cluster/health || exit 1"]
    volumes:
      - ./elasticsearch:/usr/share/elasticsearch/data
    ulimits:
      memlock:
        soft: -1
        hard: -1

  web:
    image: tootsuite/mastodon:v4.0.2
    restart: always
    env_file: .env.production
    command: bash -c "rm -f /mastodon/tmp/pids/server.pid; bundle exec rails s -p 3000"
    networks:
      - external_network
      - internal_network
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy=off localhost:3000/health || exit 1"]
    ports:
      - "127.0.0.1:3000:3000"
    depends_on:
      - db
      - redis
      - es
    volumes:
      - ./public/system:/mastodon/public/system
```

.env.production

```env
# ElasticSearch (optional)
# ------------------------
ES_ENABLED=true
ES_HOST=es
ES_PORT=9200
#ES_USER=elastic
#ES_PASS=
```

Elasticsearchのデータベース更新

```shell
sudo docker compose run --rm web bundle exec bin/tootctl search deploy
```

```
bundler: failed to load command: bin/tootctl (bin/tootctl)
/opt/mastodon/vendor/bundle/ruby/3.0.0/gems/elasticsearch-transport-7.13.3/lib/elasticsearch/transport/transport/base.rb:218:in `__raise_transport_error': [401] {"error":{"root_cause":[{"type":"security_exception","reason":"missing authentication credentials for REST request [/chewy_specifications/_search]","header":{"WWW-Authenticate":["Basic realm=\\"security\\" charset=\\"UTF-8\\"","ApiKey"]}}],"type":"security_exception","reason":"missing authentication credentials for REST request [/chewy_specifications/_search]","header":{"WWW-Authenticate":["Basic realm=\\"security\\" charset=\\"UTF-8\\"","ApiKey"]}},"status":401} (Elasticsearch::Transport::Transport::Errors::Unauthorized)
```

Elasticsearchを`docker compose up -d`で初回起動すると、ターミナルが割り当てられないため、パスワードが設定されない（以下、Elasticsearchのログ）。

```
Auto-configuration will not generate a password for the elastic built-in superuser, as we cannot  determine if there is a terminal attached to the elasticsearch process. You can use the `bin/elasticsearch-reset-password` tool to set the password for the elastic user.
```

指示に従ってパスワードを設定する（自動生成）。

```shell
sudo docker compose exec es bin/elasticsearch-reset-password -u elastic -a
```

## 参考

- <https://sleeplessbeastie.eu/2022/05/02/how-to-take-advantage-of-docker-to-install-mastodon/>
- <https://nonylene.hatenablog.jp/entry/2022/07/22/012824>
- <https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-dev-mode>
