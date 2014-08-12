# TypeScript と Mocha で CommonJS をテストする

TypeScript, Mocha, Gulp で CommonJS のテストをして Browserify でクライアント JavaScript を書き出す。まとまった知見がなかったので環境整備してひな形作った。

[client-plugin-skeleton.ts](https://github.com/MisumiRize/client-plugin-skeleton.ts)

```
npm install
node_modules/.bin/tsd reinstall -s
npm test
```

clone してテストを実行すると dist にクライアント JavaScript が生成される。

## TypeScript の reference, import

TypeScript の reference, import がどう働くかをざっくりと理解しないとすぐにハマる。というか TypeScript 使いはじめて真っ先にハマった。古いシンタックスの情報も残ってて混乱する。

以下は TypeScript 1.0 かつ CommonJS での情報なので、環境が変わった際も同じパターンで対処可能である保証はない。

前提として、 TypeScript のコンパイラコマンドには --module オプションで CommonJS, AMD として出力ができる。 CommonJS, AMD ともに外部 JavaScript を使う場合には、 .d.ts ファイルを参照する必要がある。

```
/// <reference path="../typings/mocha/mocha.d.ts" />
```

このとき、参照の対象として TypeScript ファイルを直接指定することもでき、外部 TypeScript ファイルを参照して TypeScript を書く時に必要である、とまとめられている入門が多い。 AMD としてコンパイルする際はこの方法で問題ないが、 CommonJS ではこの方法は通用しない。 

CommonJS で外部 TypeScript ファイルを参照する際、 import を使わなければならない。

```
import External = require('./external');
```

一見すると CommonJS の require と同じもののように見えるが、 .js ファイルを指定しても容赦なく require に失敗する。import した場合には、 import したファイルの reference は不要である。

しかし、 CommonJS においても外部 JavaScript を使用する場合には依然として、その .js に対する定義を reference することが必要となる。ここが混乱の最大の原因である。

TypeScript に関する情報を収拾していると、 reference に基づいてよしなに concat された状態にコンパイルしてくれるように書かれている情報もあるが、こと CommonJS での出力に関しては一切そんなことはない。外部 JavaScript に対する定義がないとコンパイルにこけることはあっても、定義によってコンパイルの結果が変わるということはない（当然）。

export 接頭詞がついた class, module 、あるいは export に代入した class, module が require の対象となる。
export 接頭詞がついた class, module は require したオブジェクトの要素として、 export に代入した class, module は require したオブジェクトそのものとして使用できる。

## TypeScript, Mocha, CommonJS

TypeScript には CoffeeScript のような Mocha にロードして .coffee ファイルをそのままテストする仕組みはない。
コンパイラを実行して出力された JavaScript コードを Mocha に読み込ませることになる。

```
tsc --module commonjs test/person.ts
```

依存関係を解決した上で、ディレクトリ構成を保ったままコンパイルされるので、そのまま Mocha で実行できる。

## Gulp と Browserify のパス解決が微妙に違う

* Gulp は src 指定の際 ./ がなくても cwd としてパス解決できる
* Browserify は ./ を入れないと cwd としてパス解決できない

## まとめ

TypeScript, Mocha, Gulp, Browserify 個別の情報はそれなりあるけどまとまったハマり情報は少ない。
