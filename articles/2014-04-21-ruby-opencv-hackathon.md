# 大江戸 Ruby 会議 04 に行けなかったので Ruby/OpenCV でクソコラハッカソンした

のをまとめた。

## 完成品とコンセプト

[え〜〜絵里ちゃんプリクラ知らないの〜〜〜？？？？](http://kke.ayase-e.li)

パターンマッチで顔を検出してかしこくかわいくしてしまおうというやつ。  
写真を入れると自動で顔を検出して例の顔にされてしまったり、本来顔ではない部分が誤検出で置き換えられてしまう偶然の面白さを味わったりできる。  
勢いだけのクソコラを作る作業が不毛であるならば、そのクソコラを作る仕組みを可視化してしまえばよいというコンセプトに基づいて作られた、クソコラを脱構築する意欲作（てきとう）。

## 着想

元々 OpenCV を使って何かやりたいという漠然とした欲求だけあって、２年ぐらい前に OpenCV + Android でカメラ撮影した画像を操作するという物を作りかけたが頓挫。そのときは .apk が 100MB 程度になったので、誰かに遊んでもらうにしても障壁が高いなということで諦め。

最近になって OpenCV 界隈を見てみたら、大本の Gem から fork された [ruby-opencv](https://github.com/ruby-opencv/ruby-opencv) が Ruby 2.0 に対応してて、結構簡単に触れそうだった。インターフェースが web だったらそこそこの人数に触ってもらえるだろうということで手を動かしてみた。

何よりちょうどラブライブ！２期１話でとんでもない素材が出てきたのが大変よろしくなかった。

## インストールして動くものを作る

まず手元の Mac にインストールし、Ruby スクリプトで Ruby/OpenCV の顔検出サンプルを実行する。 Mac だと OpenCV は Homebrew の homebrew-science からインストールできる。

```
$ brew tap homebrew/science
$ brew install opencv
```

Gemfile に ruby-opencv を指定して bundle install し、リポジトリにあったサンプルコードを写経して動くところまで確かめる。ちゃんと動くし反応速度もそんなに悪くない。

このまま続けてもリリースできるものが作れると判断。ここまでで着想から半時間。

## リリースまでの工程を見通す

コアになる技術の裏付けは取れたので、最速でリリースできる最小の構成を考える。

* 変にバズったりしなければ EC2 の t1-micro 無料枠で十分
* OS は使い慣れてる RHEL
* インスタンス管理は使い慣れてる Vagrant + AWS provider
* プロビジョニングは使い慣れてる Chef + Berkshelf
* web サーバは使い慣れてる Nginx
* アプリケーションサーバは設定ファイルの使い回しが利く Unicorn
* 上に載せるのは使い慣れてて軽い Sintra

これなら１日か２日あればなんとかなると考える。これで着想から１時間ぐらい。

## 価値を生み出す部分を早い段階で作る

手元にある人物写真を片っ端から試したかったので、今動いている物をそのまま Sinatra に載せて、Slim で簡単にファイルを POST するフォームを作った（結局時間切れでこれは最後までこのままだった）。いくつか試したところで、面白さを損なってしまう問題のパターンが洗い出された。

* 大きい画像を POST した場合に、検出と置き換えで時間がかかってしまう
* 大きい画像を POST した場合に、誤検出が増えてしまう
* 検出される顔の領域が本当の顔より大きく、単純に重ねるとコラがはみ出してしまう

誤検出については OpenCV::CvHaarClassifierCascade#detect のオプションに検出する領域の最小サイズが指定できることをリファレンス斜め読みで気づく。しかし、他の問題を解決しようとすると使い慣れていない Ruby/OpenCV 一本で処理するよりも、顔検出だけ Ruby/OpenCV に任せて、画像処理は全部 RMagick でやってしまった方が楽なことに気づく。

マシンリソース的には無駄が生じるが、そこで悩み続けるよりは動く物を完成させる。実装完了の時点で約４時間。

## 延々とハマる

サーバを用意する段階になって、yum から最新版の OpenCV をインストールする情報が全くないことに気づく。そこで OpenCV をソースビルドでインストールする [opencv-cookbook](https://github.com/Youscribe/opencv-cookbook) を clone して RedHat 系に対応させることにした。

* t1-micro 上で make するのは日が暮れるレベルで時間がかかる
* EBS の容量が不足して最初からやりなおす
* numpy が最初に古いバージョンが入っていて upgrade しないと make できない
* 最終的に諦めて VitualBox provider 上で make した OpenCV を rsync で送り込む
* make install 完了して意気揚々と rackup したら PNG が読み込めずにこける
* libpng が Ruby から読めなくてダメになっててふりだしに戻る
* recipe の実行順調整して新しいインスタンスでやったら通った

紆余曲折を経て結局２話放送開始ギリギリまで時間を食ってしまう。これで丸２日ぐらい。

## リリースと反省

勢いで作ったバカ作品がそれなりに楽しんでもらえたのでよかった。最初だけバズるのを警戒して ELB 立てて後ろ側に３台サーバ立ち上げていたが、最初のピークで CPU 使用率が 3-40% あたりまでいっただけであと元に戻った。現在は t1-micro 単発で運用している。

関東２話までに完成させるということで実質週末の２日ぐらいしか使えなかったので、UI 部分が相当におざなりになってしまった。反省してます。

クリンナップしたコードはまた後日。