---
title: Mastodon on Docker Composeで全文検索エンジンElasticsearchを有効化する
date: '2022-11-23T14:40:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Mastodon
tags:
  - Mastodon
  - docker-compose
---
# Mastodon on Docker Composeで全文検索エンジンElasticsearchを有効化する

- Mastodon on Docker Composeを立てる記事: <https://blog.aoirint.com/entry/2020/mastodon_docker/>

上の記事では、諸般の事情（主にスペック不足）によりElasticsearchを無効化した状態でMastodonインスタンスを立てていた。


## 2021年のElasticsearchのライセンス変更

クラウド事業者によるマネージドサービス提供における、オープンソースコミュニティへのコントリビューションの不足等を背景として、
2021年のバージョン7.11リリース以降、ElasticsearchのライセンスはApache 2.0から独自のElastic License 2.0（およびServer Side Public Licenseのデュアルライセンス）に変更された。

- <https://www.elastic.co/jp/pricing/faq/licensing>

この記事では、Elastic License 2.0によってライセンスされる公式のElasticsearchの配布パッケージを使用する。

- Elastic License 2.0の条文: <https://www.elastic.co/licensing/elastic-license>

ちなみにElastic License 2.0は、Elasticsearchをマネージドサービスとして提供することを禁止しているが、SaaSアプリケーションのバックエンドとして使用すること（Elasticsearch APIへの直接アクセスを提供しないサービス提供）に影響しないという立場が示されている。

> Elasticsearchをバックエンドで使用するSaaSアプリを開発しているけど、どんな影響が生じる？
> 
> 今回のソースコードのライセンス変更はお客様に一切影響しません。Elastic Licenseに基づいて、デフォルトの配布パッケージを使用できるほか、このパッケージをベースに無料でアプリケーションを開発することもできます。Elastic Licenseはsource-available license（ソース利用許諾）であり、コピーレフトの側面を持たず、デフォルトの機能を無料とします。具体的な例として、よろしければMagentoプロジェクトに関する質問への回答をご参照ください。

- <https://www.elastic.co/jp/pricing/faq/licensing#elasticsearch%E3%82%92%E3%83%90%E3%83%83%E3%82%AF%E3%82%A8%E3%83%B3%E3%83%89%E3%81%A7%E4%BD%BF%E7%94%A8%E3%81%99%E3%82%8Bsaas%E3%82%A2%E3%83%97%E3%83%AA%E3%82%92%E9%96%8B%E7%99%BA%E3%81%97%E3%81%A6%E3%81%84%E3%82%8B%E3%81%91%E3%81%A9%E3%80%81%E3%81%A9%E3%82%93%E3%81%AA%E5%BD%B1%E9%9F%BF%E3%81%8C%E7%94%9F%E3%81%98%E3%82%8B%EF%BC%9F>

> I'm using Elasticsearch to put a search box on my cat-picture SaaS product.
> 
> This is permitted under ELv2. Meow!

> I am a Managed Service Provider (MSP) running Elasticsearch and Kibana for my customers.
>
> If your customers do not access Elasticsearch and Kibana, this is permitted under ELv2. If your customers do have access to substantial portions of the functionality of either Elasticsearch and Kibana as part of your service, this may not be permitted.

- <https://www.elastic.co/licensing/elastic-license/faq#can-you-provide-some-examples-around-what-qualifies-as-providing-the-software-to-third-parties-as-a-hosted-or-managed-service-or-not>


## docker-compose.yml の更新

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


## Elasticsearchのパスワード設定

Elasticsearchを`docker compose up -d`で初回起動すると、ターミナルが割り当てられないため、パスワードが設定されない（以下、Elasticsearchのログ）。

```
Auto-configuration will not generate a password for the elastic built-in superuser, as we cannot  determine if there is a terminal attached to the elasticsearch process. You can use the `bin/elasticsearch-reset-password` tool to set the password for the elastic user.
```

指示に従ってパスワードを設定する（ランダム生成）。生成されたパスワードを控えておく。

```shell
sudo docker compose up -d es
sudo docker compose exec es bin/elasticsearch-reset-password -u elastic -a
```


## .env.production の更新

Elasticsearch関連のコメントアウトを解除し、生成したElasticsearchのパスワードを`ES_PASS`に指定する。

```env
# ElasticSearch (optional)
# ------------------------
ES_ENABLED=true
ES_HOST=es
ES_PORT=9200
ES_USER=elastic
ES_PASS=
```


### Elasticsearchのデータベース更新

```shell
sudo docker compose up -d
sudo docker compose exec web bundle exec bin/tootctl search deploy
```


## よくあるエラー

### MastodonのElasticsearchへの認証エラー

以下のようなログが`web`サービスから出た場合、Elasticsearchのユーザ名やパスワードが正しいか、パスワードが設定されているか確認する（Elasticsearchのパスワード設定の項）。

```
bundler: failed to load command: bin/tootctl (bin/tootctl)
/opt/mastodon/vendor/bundle/ruby/3.0.0/gems/elasticsearch-transport-7.13.3/lib/elasticsearch/transport/transport/base.rb:218:in `__raise_transport_error': [401] {"error":{"root_cause":[{"type":"security_exception","reason":"missing authentication credentials for REST request [/chewy_specifications/_search]","header":{"WWW-Authenticate":["Basic realm=\\"security\\" charset=\\"UTF-8\\"","ApiKey"]}}],"type":"security_exception","reason":"missing authentication credentials for REST request [/chewy_specifications/_search]","header":{"WWW-Authenticate":["Basic realm=\\"security\\" charset=\\"UTF-8\\"","ApiKey"]}},"status":401} (Elasticsearch::Transport::Transport::Errors::Unauthorized)
```


### GeoIPの更新エラー

GeoIPデータベースの更新が`exception during geoip databases update`というメッセージでなぜか失敗することがある。

```
{"@timestamp":"2022-11-19T08:03:05.924Z", "log.level":"ERROR", "message":"exception during geoip databases update", "ecs.version": "1.2.0","service.name":"ES_ECS","event.dataset":"elasticsearch.server","process.thread.name":"elasticsearch[86daa7202fdd][generic][T#3]","log.logger":"org.elasticsearch.ingest.geoip.GeoIpDownloader","elasticsearch.cluster.uuid":"hheOc5duSHeHSeLuX2UJuw","elasticsearch.node.id":"7Bqa7bvcRvq_xxEEyVwflw","elasticsearch.node.name":"86daa7202fdd","elasticsearch.cluster.name":"es-mastodon","error.type":"java.net.SocketTimeoutException","error.message":"Connect timed out","error.stack_trace":"java.net.SocketTimeoutException: Connect timed out\n\tat java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:539)\n\tat java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:585)\n\tat java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)\n\tat java.base/java.net.Socket.connect(Socket.java:666)\n\tat java.base/sun.security.ssl.SSLSocketImpl.connect(SSLSocketImpl.java:304)\n\tat java.base/sun.net.NetworkClient.doConnect(NetworkClient.java:178)\n\tat java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:531)\n\tat java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:636)\n\tat java.base/sun.net.www.protocol.https.HttpsClient.<init>(HttpsClient.java:264)\n\tat java.base/sun.net.www.protocol.https.HttpsClient.New(HttpsClient.java:378)\n\tat java.base/sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.getNewHttpClient(AbstractDelegateHttpsURLConnection.java:193)\n\tat java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect0(HttpURLConnection.java:1241)\n\tat java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect(HttpURLConnection.java:1127)\n\tat java.base/sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.connect(AbstractDelegateHttpsURLConnection.java:179)\n\tat java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream0(HttpURLConnection.java:1661)\n\tat java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream(HttpURLConnection.java:1585)\n\tat java.base/java.net.HttpURLConnection.getResponseCode(HttpURLConnection.java:529)\n\tat java.base/sun.net.www.protocol.https.HttpsURLConnectionImpl.getResponseCode(HttpsURLConnectionImpl.java:308)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.HttpClient.lambda$get$0(HttpClient.java:46)\n\tat java.base/java.security.AccessController.doPrivileged(AccessController.java:569)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.HttpClient.doPrivileged(HttpClient.java:88)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.HttpClient.get(HttpClient.java:40)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.HttpClient.getBytes(HttpClient.java:36)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloader.fetchDatabasesOverview(GeoIpDownloader.java:155)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloader.updateDatabases(GeoIpDownloader.java:143)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloader.runDownloader(GeoIpDownloader.java:274)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloaderTaskExecutor.nodeOperation(GeoIpDownloaderTaskExecutor.java:102)\n\tat org.elasticsearch.ingest.geoip@8.5.1/org.elasticsearch.ingest.geoip.GeoIpDownloaderTaskExecutor.nodeOperation(GeoIpDownloaderTaskExecutor.java:48)\n\tat org.elasticsearch.server@8.5.1/org.elasticsearch.persistent.NodePersistentTasksExecutor$1.doRun(NodePersistentTasksExecutor.java:42)\n\tat org.elasticsearch.server@8.5.1/org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingAbstractRunnable.doRun(ThreadContext.java:892)\n\tat org.elasticsearch.server@8.5.1/org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:26)\n\tat java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144)\n\tat java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)\n\tat java.base/java.lang.Thread.run(Thread.java:1589)\n"}
```

データベースの自動更新を停止するには、`docker-compose.yml`の`es`サービスの`environment`に以下を追加する。

```yaml
      - "ingest.geoip.downloader.enabled=false"
```

- <https://www.elastic.co/guide/en/elasticsearch/reference/current/geoip-processor.html>
- <https://discuss.elastic.co/t/how-to-disable-geoip-usage-in-7-14-0/281076>
- <https://stackoverflow.com/questions/72597824/org-elasticsearch-elasticsearchexception-not-all-primary-shards-of-geoip-data>
- <https://github.com/elastic/elasticsearch/issues/76586>


## 参考

- <https://sleeplessbeastie.eu/2022/05/02/how-to-take-advantage-of-docker-to-install-mastodon/>
- <https://nonylene.hatenablog.jp/entry/2022/07/22/012824>
- <https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-dev-mode>
