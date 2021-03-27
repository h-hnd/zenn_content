---
title: "pythonを用いた日本語文字列のデコード" # 記事のタイトル
emoji: "" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Python2", "Python3", ""] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


# はじめに
たまに「\xE3\x81\xAA\xE3\x82\x93\xE3\x81...」のような文字列を見かけることがあり、読める文字列に変換する方法を知りたくなったので調べてみました。
ほぼ走り書きのメモレベルです。
(Qiitaにも同じ記事がありますがバックアップも兼ねて持ってきました)

# 実際のコード

## python2の場合

```python
>>> print '\xE3\x81\xAA\xE3\x82\x93\xE3\x81\x8B\xE6\x96\x87\xE5\xAD\x97\xE5\x88\x97'.decode('utf8', 'ignore')
なんか文字列
```
sjisなら

```python
print '(なんか文字列)'.decode('cp932', 'ignore')
```

でそれっぽく変換できることが分かりました。
ignoreを入れておくことで途中上手く変換できないコードがあってもスルーされます。
(無視されて当該文字は消える)
ちなみに、ignoreは使えませんが、base64も同じ要領でデコードできます。


## python3の場合

```python
>>> b'\xE3\x81\xAA\xE3\x82\x93\xE3\x81\x8B\xE6\x96\x87\xE5\xAD\x97\xE5\x88\x97'.decode('utf8', 'ignore')
'なんか文字列'
```

で変換できます。

# 参考
https://docs.python.org/ja/2.7/howto/unicode.html
https://docs.python.org/ja/3/howto/unicode.html
