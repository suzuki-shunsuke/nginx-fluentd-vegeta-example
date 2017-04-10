# nginx-fluentd-vegeta-example

* vegeta を使ってNginxにリクエストを投げる
  * ログをチェックして投げられていることを確認
* nginxのログを別のfluentdのコンテナに投げる
  * fluentdのほうでログ収集出来ていることを確認

## 関連リンク

* https://github.com/tsenart/vegeta
* http://knowledge.sakura.ad.jp/tech/1336/
* http://www.fluentd.org/
* http://blog.a-know.me/entry/2016/02/02/202742
* https://docs.docker.com/engine/admin/logging/fluentd/
* https://github.com/docker/docker/issues/20370#issuecomment-185229591

## 1. vegetaを使ってみる

```
$ brew install vegeta
$ docker-compose up -d
$ docker-compose logs -f
$ echo "GET http://localhost:8080/" | vegeta attack -rate=100 -duration=5s | vegeta report
```

## 2. Nginxのログをfluentdに流す

```yaml
  nginx:
    image: nginx:1.11.3
    ports:
    - "8080:80"
    logging:
      driver: fluentd
      options:
        fluentd-address: 172.21.0.2:24224
        tag: docker.{{.FullID}}
    depends_on:
    - fluentd
  fluentd:
    image: fluent/fluentd:v0.14.14-debian
    ports:
    - "24224:24224"
    - "24224:24224/udp"
    volumes:
    - ./data:/fluentd/log
```

fluentd-address でipアドレスをハードコードしているが、これについては後述する。

こうするとnginxのログがfluentdに渡される。
fluentd log driverを使うと `docker-compose logs` コマンドによってログは確認できなくなる。
fluentdのイメージの設定ファイルはこちら

https://github.com/fluent/fluentd-docker-image/blob/master/v0.14/debian/fluent.conf

ログファイルに書き込んでいるのでそのファイルをtailしながらnginxにリクエストを投げればfluentdにログが収集できているのが確認できる。

### fluentd-address について

docker の fluentd log driver で fluentdのdns名としてdocker compose のサービス名は使えないらしい。
つまり、次の設定はエラーになる。

```
fluentd-address: fluentd:24224
```

https://github.com/docker/docker/issues/20370#issuecomment-185229591

しかたないので fluentdのコンテナを先に作成し、`docker inspect` コマンドでipを調べてハードコードしている。

```
$ docker-compose up -d fluentd
$ docker ps  # コンテナ名確認
$ docker inspect <fluend コンテナ名>  # ipアドレスを確認
$ vi docker-compose.yml  # fluentd-address を修正
$ docker-compose up -d nginx
```

場合によってはipアドレスを固定するのも良いかもしれない。

https://docs.docker.com/compose/compose-file/#ipv4address-ipv6address
