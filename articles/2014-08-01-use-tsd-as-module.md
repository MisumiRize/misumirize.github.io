# TSD (TypeScript / DefinitelyTyped) をモジュールとして使う

* [DefinitelyTyped](https://github.com/borisyankov/DefinitelyTyped)
* [TSD](https://github.com/DefinitelyTyped/tsd)

TypeScript の型定義情報を集積してるやつと管理するやつ。コマンドラインから使う方法は散々既出だったが、モジュールとして使う方法もしれっと書いてあった。

Gulp から直で呼べる。やったぜ。

なお、TSD 0.5.7 における情報なのであとで変わっているかもしれない。

## TSD をコマンドラインとして使う

```
npm install -g tsd
```

グローバルにインストールして、 .d.ts ファイルを生成する。

```
tsd init
tsd query mocha -a install -s
```

このときに生成される tsd.json は .d.ts ファイルのインストール情報になるのでリポジトリにコミットする。
typings ディレクトリの中身は tsd.json から再インストール可能なので gitignore した。

新しい環境で .d.ts をセットアップする時は再インストールコマンドを使う。

```
tsd reinstall -s
```

## TSD をモジュールとして使う

```javascript
var tsd = require("tsd");
var api = new tsd.API(new tsd.Context());
```

ここから DefinitelyTyped に対してクエリを投げることができる。が、先に readConfig() しておかないと今までの設定が吹っ飛ぶ。
readConfig() の戻りは promise になっているので then() で後の処理を続ける。

例えば tsd.json から情報を取得して .d.ts ファイルを再生成するのは以下のようなコードになる。

```javascript
api.readConfig().then(function() {
    api.reinstall(tsd.Options.fromJSON({ saveToConfig: true }));
});
```

もちろんこういう風に書いてしまえば Gulp から直接呼べる。

## まとめ

TSD をモジュールとして使えば tsd.json から .d.ts を再生成する Gulp タスクもシンプルに組める。
ただしリファレンスはないので開けたり閉めたりが必要。
