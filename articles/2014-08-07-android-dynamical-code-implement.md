# 世界一 IQ の低いコードそのもの動的に定義する

元ネタは一週間ぐらい前に話題になった[これ](https://twitter.com/vjroba/status/494882208788660226)。

リフレクションを使うとか、ルックアップテーブルを定義するとかは散々出尽くしてるので、あえて「世界一 IQ の低いコードをリフレクションから動的に定義して実行する」という回り回ったものを実装した。

## 答え

[DexMaker](https://code.google.com/p/dexmaker/) 使う。

## 作った

[LowestIQ](https://github.com/MisumiRize/LowestIQ) 

DexMaker のための疑似コードを起こすにあたって、若干元のコードに修正が加えられているが許してほしい。

```
  String keyName;

  keyName = "KEYCODE_XXX";
  if (keyCode != KeyEvent.KEYCODE_XXX) goto label;
  return keyName;

label:
  // 延々と比較が続く
```

こんな感じにひっくり返して書いてある。
