# Heroku でゆるく fluentd + elasticsearch を試す

fluentd + elasticsearch を導入して様々なデータを集約かつ検索可能にしたくなった。

類例だと AWS 上に環境構築するハウツーは結構出回っているけれども、Java が必要だったりセットアップ完了までが煩雑だったりでサクッと触るのに敷居が高い。そして何より AWS 無料枠がとっくに切れて課金が発生してるので実構築までは安上がりで済ませたい。

TreasureData が [Heroku 上に fluentd を構築する](http://docs.fluentd.org/ja/articles/install-on-heroku)方法を公開しているので、これを参考に elasticsearch へとデータを流す設定を作る。

Heroku addon として elasticsearch を提供するサービスには [Bonsai](https://addons.heroku.com/bonsai) と [SeachBox](https://addons.heroku.com/searchbox) があり、どちらも最小プランは無料（ただしクレジットカードの登録は必要）。無料枠の限度が多少違っていて、Bonsai は 10,000 Documents まで、SearchBox は 5 MB まで。外部から REST API を叩けたり、日本語形態素解析に必要な Kuromoji が入っていたりは、仕様を斜め読みした限り今のところ同条件。今回は SearchBox を使用した。

[Heroku でゆるく fluentd と elasticsearch を試す Gist](https://gist.github.com/MisumiRize/05c7b7b4553eef49a2e5)

## セットアップ

```
# Clone
$ git clone https://gist.github.com/05c7b7b4553eef49a2e5.git heroku-fluentd-elasticsearch
$ cd heroku-fluentd-elasticsearch
$ rm -fR .git
$ git init
$ git add .
$ git commit -m 'initial commit'

# app を作成して SearchBox を有効化する
$ heroku create --stack cedar
$ heroku addons:add searchbox

# SearchBox のダッシュボードを開く
$ heroku addons:open searchbox
```

app を作成して SeachBox を有効化したら、elasticsearch に index を作成し接続情報を確認する。

fluent-plugin-elasticsearch は特に index_name を指定しない限り、fluentd という名前の index に document が投げられるので、New Index から fluentd という名前で index を作成しておく。

Configuration の `SEARCHBOX_URL` が接続用の URL となるので、td-agent.conf の `YOUR_SEARCHBOX_URL` をこれで置き換える。fluent-plugin-elasticsearch に直で URL を指定するフィーチャは master にまだマージされていないので、pull req 元からインストールするようにしている。そのうちマージされるはず。

```
$ vim td-agent.conf
$ git push heroku master
```

## テスト

ここまで操作すると Heroku 上で fluentd が動作しはじめるので、実際にログが流れているかを確認する。

```
# 標準出力にログが流れるかを確認する
$ curl "http://my-fluentd-on-heroku.herokuapp.com/debug.sample?json=%7B%22json%22%3A%22message%22%7D"
$ heroku logs

# elasticsearch にログを流す
$ curl "http://my-fluentd-on-heroku.herokuapp.com/es.sample?json=%7B%22json%22%3A%22message%22%7D"
```

Configuration の Alternative Url から elasticsearch の REST API にアクセスできるので、正しくデータが投入されたかを Search API から確認してみる。Search がヒットしたら成功。

```
$ curl "http://bofur-us-east-1.searchly.com/api-key/MY_SEARCHBOX_API_KEY/fluentd/_search?q=json:message"
```
