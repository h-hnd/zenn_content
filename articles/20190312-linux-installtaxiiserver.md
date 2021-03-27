---
title: "TAXIIサーバを入れてみる" # 記事のタイトル
emoji: "" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Python", "セキュリティ", "TAXII", "STIX"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
## STIX/TAXIIに興味を持った
* 何かしら脆弱性情報をチェックしていくのに便利な方法はないものか…
* メールは来ているけど人の目で見て対応するのはコスト高いしメールを整形するのもメンテが大変そう
* STIX/TAXIIって便利かも
* OPENTAXIIを試しに使ってみよう&STIX2.0がどんなものか見てみよう

(Qiitaにも同じ記事がありますがバックアップも兼ねて持ってきました)


## 前提
OSはCentOS 7.4 (Minimumインストール)
適当に仮想環境を作って入れました。

```bash
$ cat /etc/redhat-release
CentOS Linux release 7.4.1708 (Core)
```

# インストール

## pipのインストール
pipはepelに入っているので、まずはepelを入れるところから。

```bash
$ sudo yum install epel-release
```

## ついでにPostgreSQLもインストール
DB使って情報をやりとりしているみたいです。
SQLiteがデフォルトですが、公式サイトにはPostgreSQLがおすすめと書いてあったのでPostgreSQLを使うことにしました。

```bash
$ sudo yum install postgresql
$ sudo yum install postgresql-server postgres-contrib
S sudo su - postgres
$ initdb
```

## OPENTAXIIのインストール
pipでインストールできるのでだいぶ楽です。

```bash
$ sudo pip install opentaxii
```

## ユーザとDB作成

```bash
$ sudo useradd taxii -m
$ su - postgres
$ psql
postgres=# CREATE USER taxii WITH ENCRYPTED PASSWORD '(パスワード)' CREATEDB;
postgres=# \q
$ exit
$ su - taxii
$ createdb
```

## コンフィグファイルの作成
pipでインストールすると、opentaxiiは
```
/usr/lib/python2.7/site-packages/opentaxii
```
にできます。

コンフィグファイルはデフォルトをコピーして編集します。

```bash
$ cd /usr/lib/python2.7/site-packages/opentaxii
$ cp -p defaults.yml taxiitest.yml
$ vi taxiitest.yml
```

変更箇所は下記の通り。

```diff
diff -u defaults.yml taxiitest.yml 
--- defaults.yml        2017-02-17 05:51:57.000000000 +0900
+++ taxiitest.yml       2018-04-23 20:45:39.547623862 +0900
@@ -7,13 +7,13 @@
 persistence_api:
   class: opentaxii.persistence.sqldb.SQLDatabaseAPI
   parameters:
-    db_connection: sqlite:////tmp/data.db
+    db_connection: postgresql://taxii:(パスワード)@localhost:5432/taxii
     create_tables: yes
 
 auth_api:
   class: opentaxii.auth.sqldb.SQLDatabaseAPI
   parameters:
-    db_connection: sqlite:////tmp/auth.db
+    db_connection: postgresql://taxii:(パスワード)@localhost:5432/taxii
     create_tables: yes
     secret: SECRET-STRING-NEEDS-TO-BE-CHANGED
```

ドキュメントにはserver.ymlとcollections.ymlを使うと書いてありますが、リンク先は404...
data-configuration.ymlがそれにあたります。共通で使えます。

```bash
$ sudo opentaxii-create-services -c data-configuration.yml 
$ sudo opentaxii-create-collections -c data-configuration.yml 
```

そしたら…

```
ImportError: No module named psycopg2
```

パッケージが入っていないと言われたので入れます。

```bash
$ sudo pip install psycopg2
$ sudo pip install psycopg2-binary
```

gunicornのインストール

```bash
$ sudo pip install gunicorn
```

gnunicornを起動します。
バックグラウンドで起動しないと待ちっぱなしになってしまうので注意が必要です。

```bash
$ gunicorn opentaxii.http:app --bind localhost:9000 &
```


動作確認

```bash
$ curl http://localhost:9000/management/health
{
  "alive": true
}
```

taxiiのユーザ作成が漏れていたので作成します。

```bash
$ opentaxii-create-account -u taxii -p taxii
2018-04-24T06:27:02.067965Z [opentaxii.utils] info: api.initialized {timestamp=2018-04-24T06:27:02.067965Z, logger=opentaxii.utils, api=opentaxii.persistence.sqldb.SQLDatabaseAPI, event=api.initialized, level=info}
2018-04-24T06:27:02.097309Z [opentaxii.utils] info: api.initialized {timestamp=2018-04-24T06:27:02.097309Z, logger=opentaxii.utils, api=opentaxii.auth.sqldb.SQLDatabaseAPI, event=api.initialized, level=info}
2018-04-24T06:27:02.097767Z [opentaxii.server] info: opentaxii.server_configured {timestamp=2018-04-24T06:27:02.097767Z, logger=opentaxii.server, event=opentaxii.server_configured, level=info}
2018-04-24T06:27:02.244743Z [opentaxii.auth.manager] info: account.created {username=taxii, timestamp=2018-04-24T06:27:02.244743Z, logger=opentaxii.auth.manager, event=account.created, level=info}
2018-04-24T06:27:02.308599Z [opentaxii.cli.auth] info: account.token {username=taxii, level=info, timestamp=2018-04-24T06:27:02.308599Z, token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2NvdW50X2lkIjoxLCJleHAiOjE1MjQ1NTQ4MjJ9.LTOorS4xfHkvYC0fe2iu12KSXXypbQE4paJoOxv26_s, logger=opentaxii.cli.auth, event=account.token}
```

taxiiの通信にはcabbyが簡単らしいのでインストール。

```
$ sudo pip install cabby
```

情報の取得先を取ってみました。

```
$ taxii-discovery --path http://localhost:9000/services/discovery-a
2018-04-24 16:20:06,529 INFO: Sending Discovery_Request to http://localhost:9000/services/discovery-a
2018-04-24 16:20:06,553 INFO: 7 services discovered
=== Service Instance ===
  Service Type: INBOX
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:http:1.0
  Service Address: http://localhost:9000/services/inbox-a
  Message Binding: urn:taxii.mitre.org:message:xml:1.0
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Inbox Service AC: []
  Available: True
  Message: Custom Inbox Service Description A

=== Service Instance ===
  Service Type: INBOX
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:http:1.0
  Service Address: http://localhost:9000/services/inbox-b
  Message Binding: urn:taxii.mitre.org:message:xml:1.0
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Inbox Service AC: ['urn:stix.mitre.org:xml:1.1.1', 'urn:custom.example.com:json:0.0.1']
  Available: True
  Message: Custom Inbox Service Description B

=== Service Instance ===
  Service Type: DISCOVERY
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:http:1.0
  Service Address: http://localhost:9000/services/discovery-a
  Message Binding: urn:taxii.mitre.org:message:xml:1.0
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Available: True
  Message: Custom Discovery Service description

=== Service Instance ===
  Service Type: DISCOVERY
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:https:1.0
  Service Address: https://localhost:9000/services/discovery-a
  Message Binding: urn:taxii.mitre.org:message:xml:1.0
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Available: True
  Message: Custom Discovery Service description

=== Service Instance ===
  Service Type: COLLECTION_MANAGEMENT
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:http:1.0
  Service Address: http://localhost:9000/services/collection-management-a
  Message Binding: urn:taxii.mitre.org:message:xml:1.0
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Available: True
  Message: Custom Collection Management Service description

=== Service Instance ===
  Service Type: COLLECTION_MANAGEMENT
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:https:1.0
  Service Address: https://localhost:9000/services/collection-management-a
  Message Binding: urn:taxii.mitre.org:message:xml:1.0
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Available: True
  Message: Custom Collection Management Service description

=== Service Instance ===
  Service Type: POLL
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:http:1.0
  Service Address: http://localhost:9000/services/poll-a
  Message Binding: urn:taxii.mitre.org:message:xml:1.0
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Available: True
  Message: Custom Poll Service description
```

ひとまず取ることはできそう…なものの取った後の整形とかが大変そうだと思ったので作業が止まっています…  
折をみて再開するつもりです。

# 参考
・Opentaxii
https://opentaxii.readthedocs.io/en/stable/
・IPA「検知指標情報自動交換手順TAXII概説」
https://www.ipa.go.jp/security/vuln/TAXII.html
・他(色々と調べたものの失念…)
