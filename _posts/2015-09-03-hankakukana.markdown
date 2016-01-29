---
layout: post
title:  "名前を半角カナで入力してしまった"
date:   2015-09-03 15:26:12 +0900
categories: sql
---
名前欄に半角カナを入力してしまった。半角カナが悪いわけではないが、いろいろと面倒なので全角に戻したい。 新規に入力する際には全角で入力してもらうが、すでに登録済のデータをどうするか迷った 今回使用しているデータベースはPostgresql。これには便利なConvert関数などはないので自力でなんとかするほかないのだが、探してみると使えそうなストアドプロシージャーがあったので利用させてもらった

PostgreSQL でカタカナ変換（ユーザ定義関数）
<http://qiita.com/gold1/items/5b54761697b4ade1f8e9>

ローカルのデータベースで動作確認して、次は本番環境での実行。 本番環境はDockerで動かしているので、一手間必要だった。

この際に参考にしたのは以下のサイト。

DockerでWebサーバー/コンテナホストからDBサーバー接続
<http://qiita.com/tsutorm/items/9a707e38038daa6002c0>

コンテナのIPアドレスを調べて、そのポートに対して接続する

   $ docker inspect (コンテナの名) | grep IPAddress
   $ psql -U postgres -h (コンテナのIPアドレス) -p 5432 (データベース名) < mb_convert_kana.sql
database=# update (テーブル名) set name = mb_convert_kana(name, 'KV'); `

うん、手打ちで入力して方が早かったね。
