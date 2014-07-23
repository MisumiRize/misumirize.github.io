# weblio の検索結果をスクレイピングして peco にパイプする

## TL;DR

* そんなにライフチェンジングではない
* weblio はもう少し辞書ごとの DOM を統一してほしい
* hostname を痛々しくしていると辛い

## 発端

コード書いてるとボキャブラリーが貧困なので結構な頻度で「○○　英語」でググる。大体 weblio が出てくるけど、黒い画面からブラウザに切り替えるのも怠い。ターミナルから検索できて、なおかつクリップボードにコピーできるといい。インクリメンタルに検索できるとハッピーなので peco | pbcopy とパイプする。

## やってみた

### [peco](https://github.com/peco/peco)

パイプしたのをインクリメンタル検索するやつ。Mac だと Homebrew からインストールできる。

```
brew tap peco/peco
brew install peco
```

### スクレイピングするやつ

Go で書いた。モジュール管理楽だしエラー処理何も考えずに適当に書いてもそれなりに動いてくれるので全体的に楽。

```
package main

import (
	"fmt"
	"os"
	"github.com/PuerkitoBio/goquery"
)

var BASE_URL = "http://ejje.weblio.jp/content/"

func main() {
	if len(os.Args) < 2 {
		fmt.Fprintf(os.Stderr, "usage: %s [word]\n", os.Args[0])
		os.Exit(1)
	}
	doc, _ := goquery.NewDocument(BASE_URL + os.Args[1]); _ != nil {
		os.Exit(1)
	}
	doc.Find(".kenjeEnE, .Hypej, .jmdctGls").Each(func(_ int, s *goquery.Selection) {
		fmt.Println(s.Text())
	})
}
```

主要辞書の訳語が入ってる class を [goquery](https://github.com/PuerkitoBio/goquery) で抜いて標準出力に吐き出す。訳語おいてある部分が class も DOM もまちまちなので辛い。

## こうなった

https://twitter.com/Misumi_Rize/status/491925967221166081/photo/1

そんなに便利でもない。でも頻繁に手を動かしてないととにかく埋もれるので書き残す。
