---
title: "ファイル名の一部を一括で変更したい" # 記事のタイトル
emoji: "" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Windows10", "ファイル操作"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 忙しい人向け

* PowerToysをインストール
    * https://github.com/microsoft/PowerToys/releases
* 変更したいファイルを右クリックしてPowerRenameをクリック


# はじめに

ちょいちょいファイル名の一部を一括で変更したい事があったのですが、方法が分からず、
頻度も高くないことから結構手作業でリネームをしていました。
でも今回、ついにそれだと生きるのが辛くなってしまったので良い方法が無いか調べました。

# 方法

* PowerToysをインストール
    * https://github.com/microsoft/PowerToys/releases
* 名前を変更したいファイルを選択し、右クリック
* その中の「PowerRename」をクリック
    * PowerToysをインストールすると追加される

取り敢えずGUI上だとこれで十分便利だったので暫くこれでいこうと思います。

# 他のやり方

## Linuxの場合

renameコマンドがあります。
manページにはこんな感じの例があります。

```shell
rename 's/\.bak$//' *.bak
```

正規表現が使えるのは便利ですね。

## Windowsのコマンドライン

RENコマンドがありますが…Linuxのrenameほど精度は良くないようです。
コマンドラインリファレンスを見ると

```bat
ren *.txt *.doc
```

で拡張子を「txt」から「doc」へ一括で変更出来る…らしいです。
(今だと色々違和感のある例ですが、docもテキストファイルだった頃からのヘルプなのかも知れません)
ただ、注意点があって
>Filename2のワイルドカード文字によって表される文字は、 filename1内の対応する文字と同じになります。

だそうです。
なので、拡張子の変更は例としては良くないと思います…。


# さいごに

PowerToysが懐かしすぎて…
正直その事でテンションが上がってそのままの勢いで書きました。

# 参考

* https://4thsight.xyz/16468
* https://4thsight.xyz/16373
* https://docs.microsoft.com/ja-jp/windows-server/administration/windows-commands/ren
