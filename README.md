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

## 1. vegetaを使ってみる

```
$ brew install vegeta
$ docker-compose up -d
$ docker-compose logs -f
$ echo "GET http://localhost:8080/" | vegeta attack -rate=100 -duration=5s | vegeta report
```

## 2. Nginxのログをfluentdに流す

TODO
