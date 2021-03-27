---
title: "VirtualboxのNATネットワークをVagrantから設定する" # 記事のタイトル
emoji: "" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Vagrant", "Virtualbox", "Vagrantfile"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 設定内容

忙しい人向けに最初にやったことを書きます。諸々は後ほど。
Vagranfileでmodifyvmを使って設定します。

```text:Vagrantfile
Vagrant.configure("2") do |config|
  (中略)
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--nic1", "natnetwork"]
    vb.customize ["modifyvm", :id, "--nat-network1", "NatNetwork"]
  end
  (中略)
end
```
Virtualboxのnic1の設定にNAT ネットワークを指定して、
そのNATネットワークはNatNetworkを使用する設定になります。


# はじめに
Virtualboxに対してVagrantを使って検証用の仮想サーバを色々立てたりしてると、
複数サーバ間の通信をやりたくなったり諸々の理由でNATネットワークを指定したくなりました。
Vagrantfileの書き方を調べてみるも、NATに関する記述はあっても、NATネットワークになると途端に見つからず…
調べては挫折し、挫折しては挫折しを繰り返していました。
今回再度何の気なしに調べてみたらすんなり解決できてしまい、ちょっと嬉しかったので久々に投稿します。  
(Qiitaにも同じ記事がありますがバックアップも兼ねて持ってきました)


環境は下記です。

## Virtualbox

```
c:\> VBoxManage.exe --version
6.1.16r140961
```

## Vagrant

```
c:\> vagrant -v
Vagrant 2.2.14
```

## Windows

![SnapCrab_NoName_2021-2-2_20-5-27_No-00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/51498/c9fadaaf-1b9c-5c0c-e96b-40032139b728.png)

# 設定内容(再掲)と解説

```text:Vagrantfile
Vagrant.configure("2") do |config|
  (中略)
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--nic1", "natnetwork"]
    vb.customize ["modifyvm", :id, "--nat-network1", "NatNetwork"]
  end
  (中略)
end
```
肝は、

```
    vb.customize ["modifyvm", :id, "--nic1", "natnetwork"]
    vb.customize ["modifyvm", :id, "--nat-network1", "NatNetwork"]
```
です。
項目ごとに説明すると下記のような感じです。

* `vb.customize`はVBoxManage.exeを呼び出します
* `modifyvm`はVBoxManage.exeのオプションでVMイメージの設定を変更します(そのまま)
* `:id`はイメージIDを指定するのですが、ここはVagrantがよしなにやってくれます
* `--nic1`は1つ目のNICの設定をするという意味です
    * 後ろの数字は1-4まで設定でき(ると思い)ます
    * 調べていませんがGUIでは4つまでなのできっと他の数字を指定するとエラーになるのかなと…
* `natnetwork`はNATネットワークを指定します(そのまま)
    * ちなみに設定項目としては`none|null|nat|natnetwork|bridged|intnet|hostonly|generic`だけあります
    * GUIの設定項目と同じですね
* `--nat-network1`はNIC1に対して設定したNATネットワークのオプションでどのNATネットワークにするかを指定します
* `NatNetwork`は、事前に設定したNATネットワークを指定します
    * GUIで言う`ファイル->環境設定->ネットワーク`で表示されるNATネットワーク名です

# さいごに

結局のところ公式サイトに書いてありました。
日本語にこだわって探すと良くないですね。再認識しました。
それにしても日本語で書かれていなかったのはもしかして…

* 外に繋ぐならNAT + 内部ネットワークで良いから…?
* あまりNATネットワークって良くなかったりとかある…?

みたいなのがあるのでしょうか。
個人的にはポート周りがひと目で分かって良いので多用していますが…


# 参考にしたもの

* 公式サイト(何故これまでじっくり読まなかったのか…)
    * https://www.virtualbox.org/manual/ch08.html#vboxmanage-modifyvm

