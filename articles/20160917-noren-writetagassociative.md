---
title: "Writeタグを連想配列的に使う" # 記事のタイトル
emoji: "" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["NOREN"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに

アクションタグで処理を記述するにあたって、Writeタグが結構便利だったりします。
決まった変換規則であればあらかじめ変数に入れておいて呼び出す形にしてやるとソース上すっきりします。
あまりIfタグを使うと訳が分からなくなったりすることがあったので、今回の様な書き方をする様になりました。
※この業務から離れて気付いたらもう数年経ってしまったのでもう今はもっと良い方法があるかも知れません
(Qiitaにも同じ記事がありますがバックアップも兼ねて持ってきました)

# やり方

## 前置き

パッと良い例が思いつきませんが、MulticatActionが使えないバージョンを使っている時は有用かも知れません。
MulticatActionが使えない状態で、下記の様に色々なカテゴリから記事を持ってきて表示させたいとします。
取りたいxが取ってきたい情報とすると、何も意識しないと取りたい情報を取るために、親カテゴリで

```
[[--ActionStart, 親カテゴリ, below=yes--]]
```

みたいな事をやってしまいそうです。
で、

```
親カテゴリ
    +---カテゴリA
    |   +---取りたい1
    |   +---カテゴリA-1
    |   +---カテゴリA-2
    +---カテゴリB
    |   +---カテゴリB-1
    |   +---カテゴリB-2
    |   +---取りたい2
    +---カテゴリC
    |   +---カテゴリC-1
    |   +---カテゴリC-2
    |   +---カテゴリC-3
    |   +---カテゴリC-4
    |   +---カテゴリC-5
    +---カテゴリD
        +---取りたい3

```

こんな時、明らかに他のカテゴリが無駄な情報になってしまうのでダミーで下記の様なサイトカテゴリを作ります。

```
まとめた1
    +---取りたい1d
    +---取りたい2d
    +---取りたい3d
```

こうして、こっちにもコンテンツを取ってくるようにすれば欲しいコンテンツだけ取ってこれる様になります。

ただ、記事の削除とかしたい場合にはアーティクルを全て削除しないといけなくなってしまうので、修正の対象が多数にわたってしまう時には相当な面倒くささになってしまいます。

更に、ダミーのカテゴリにはストーリテンプレートが存在しない(作っても良いですが無駄なhtmlが出来てしまう)ので、元のストーリテンプレートにリンクを張らないといけなくなり、

```
[[--ActionStart, まとめた1, below=yes--]]
    [[--RecordStart, name=scat--]][[--ArtCatID--]][[--RecordEnd--]]
    [[--RecordStart, name=aid--]][[--ArtID--]][[--RecordEnd--]]
    [[--IfStart, is:scat=取りたい1d--]][[--ThenStart--]]
        [[---直接リンクを書いて当該記事が存在しないと即エラーになるので記事の有無を判定する意味で
             1件だけ呼び出すActionを入れておく---]]
        [[--ActionStart, 取りたい1, id:[%aid%]--]]
            <li><a href="[[--ArtAddress--]]">[[--ArtTitle--]]</a></li>
        [[--ActionEnd--]]
    [[--ThenEnd--]][[--ElseStart--]]
        [[--IfStart, is:scat=取りたい2d--]][[--ThenStart--]]
            [[--ActionStart, 取りたい1, id:[%aid%]--]]
                <li><a href="[[--ArtAddress--]]">[[--ArtTitle--]]</a></li>
            [[--ActionEnd--]]
        [[--ThenEnd--]][[--ElseStart--]]
            [[--IfStart, is:scat=取りたい3d--]][[--ThenStart--]]
                [[--ActionStart, 取りたい1, id:[%aid%]--]]
                    <li><a href="[[--ArtAddress--]]">[[--ArtTitle--]]</a></li>
                [[--ActionEnd--]]
            [[--ThenEnd--]][[--IfEnd--]]
        [[--ElseEnd--]][[--IfEnd--]]
    [[--ElseEnd--]][[--IfEnd--]]
[[--ActionEnd--]]
```

こんな感じになります。これだとわかりにくい上に取りたいカテゴリが増えたときに修正箇所も多かったりしてミスの元になります。

## 実際の書き方

ではどうするかと言うと、

```
[[--DefineStart, name=取りたい1d--]]取りたい1[[--DefineEnd--]]
[[--DefineStart, name=取りたい2d--]]取りたい2[[--DefineEnd--]]
[[--DefineStart, name=取りたい3d--]]取りたい3[[--DefineEnd--]]

[[--ActionStart, まとめた1, below=yes--]]
    [[--RecordStart, name=scat--]][[--ArtCatID--]][[--RecordEnd--]]
    [[--RecordStart, name=tcat--]][[--Write, [%tcat%]--]][[--RecordEnd--]]
    [[--RecordStart, name=aid--]][[--ArtID--]][[--RecordEnd--]]
    [[---直接リンクを書いて当該記事が存在しないと即エラーになるので記事の有無を判定する意味で
         1件だけ呼び出すActionを入れておく---]]
    [[--ActionStart, [%tcat%], id:[%aid%]--]]
        <li><a href="[[--ArtAddress--]]">[[--ArtTitle--]]</a></li>
    [[--ActionEnd--]]
[[--ActionEnd--]]
```

こうします。
変数名をダミー側のカテゴリとして、その中身を実際のカテゴリとした変数を予め宣言しておきます。
ダミーのカテゴリと実際のカテゴリで対応づけておくことで、ダミーのカテゴリから記事を取ってきたときに、カテゴリを簡単に変換する事が出来るようになります。
前置きの書き方よりはすっきりしますし、カテゴリが増えたときも変数を増やすだけの対応で済みます。

# その他

この書き方、一応足し算(引き算)にも対応出来ます。
相当変なケースですが、

* 全体で5件表示する
* あるカテゴリ(カテゴリAとします)から2件表示する
* またあるカテゴリ(カテゴリBとします)から1件表示する
* その下に他のカテゴリ(カテゴリCとします)から残り全件表示する

と言った条件の時に使えます。
まず、カテゴリAに対しては簡単で、

```
[[--DefineStart, name:counta--]]0[[--DefineEnd--]]
[[--DefineStart, name:countb--]]0[[--DefineEnd--]]
[[--ActionStart, categoryA, row:2, search:actionfield=xxx&&...--]]
	[[--RecordStart, name:counta--]][[--ArtIndex--]][[--RecordEnd--]]
	[[--アクションタグやらhtmlやら--]]
[[--ActionEnd--]]
```

とすれば、表示件数も含めて取得できます。
問題は「残り全件」と言うところ。5件表示の時もあれば4件の時もありますし、3件の時もあります。
IFで判定させても良いのですが、そうすると悲しいくらいのネストが発生します。

```
[[--ActionStart, categoryB, search:actionfield=yyy&&...--]]
	[[--RecordStart, name:countB--]][[--ArtIndex--]][[--RecordEnd--]]
	[[--アクションタグやらhtmlやら--]]
[[--ActionEnd--]]

[[--IfStart,is:counta=2--]][[--ThenStart--]]
	[[--IfStart, is:countb=1--]][[--ThenStart--]]
		[[--DefineStart, name:left--]]2[[--DefineEnd--]]
	[[--ThenEnd--]][[--ElseStart--]]
		[[--DefineStart, name:left--]]3[[--DefineEnd--]]
	[[--ElseEnd--]][[--IfEnd--]]
[[--ThenEnd--]][[--ElseStart--]]
	:
	:
[[--ElseEnd--]][[--IfEnd--]]
```

ひたすらこんな事を書く羽目になって、そのうちどこで何個If分を開いて閉じたか分からなくなってしまって収集つかなく…(実際の経験談)

これをちょっとマシにするのが無理やり加減算をしてしまう下記の方法。

```
[[---加算準備---]]
[[--DefineStart, name:plus0--]]1[[--DefineEnd--]]
[[--DefineStart, name:plus1--]]2[[--DefineEnd--]]
[[--DefineStart, name:plus2--]]3[[--DefineEnd--]]
[[--DefineStart, name:plus3--]]4[[--DefineEnd--]]
[[--DefineStart, name:plus4--]]5[[--DefineEnd--]]
[[--DefineStart, name:plus5--]]6[[--DefineEnd--]]
[[--DefineStart, name:plus6--]]7[[--DefineEnd--]]
[[--DefineStart, name:plus7--]]8[[--DefineEnd--]]
[[--DefineStart, name:plus8--]]9[[--DefineEnd--]]
[[--DefineStart, name:plus9--]]0[[--DefineEnd--]]

[[---加算処理---]]
[[--DefineStart, name:test--]]4[[--DefineEnd--]]
[[--RecordStart, name:test--]][[--Write, name:plus[%test%]--]][[--RecordEnd--]]

```

何ともやるせない気持ちにはなりますが、If文を狂ったように書くよりはだいぶスッキリするはずです。
