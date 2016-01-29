  * [ホーム](http://www.showway.biz/ "Go to homepage" )
  * 内容

# [昌永ウェブサービス](http://www.showway.biz/ "昌永ウェブサービス" )

ウェブサイト構築・ソフトウェア開発を行っております

検索:

  * [ホーム](http://www.showway.biz/)
  * [紹介](http://www.showway.biz/about)
  * [実績リスト](http://www.showway.biz/actual_list)
  * [お問い合わせ](http://www.showway.biz/contact_form)

# [ウェブサーバをNginxに変えてみた](http://www.showway.biz/nginx.html "Permanent Link to ウェブサーバをNginxに変えてみた" )

12月 23rd, 2015 by onoue  [ コメントはまだありません » ](http://www.showway.biz/nginx.html#respond)

## Nginxのインストールおよび設定

  * ArchLinuxのWikiは新しく正しい情報が多いので、まずはそちらを参照する。
  * [nginx – ArchWiki](https://wiki.archlinuxjp.org/index.php/Nginx)
  * phpinfo()が動作することを確認

## dokuwiki

  * まずはデータベースを使わないウェブアプリで試す
  * [dokuwiki – ArchWiki](https://wiki.archlinuxjp.org/index.php/Dokuwiki)
  * nginx での設定方法が載っているので、こちらもさくっと動作確認

## wordpress

  * WordPress Codex 内の Nginx 情報がよさげだったのでそれを参照。
  * [nginx – WordPress Codex 日本語版](https://wpdocs.osdn.jp/Nginx#.E3.83.A1.E3.82.A4.E3.83.B3.E3.82.B9.E3.82.BF.E3.83.BC.E3.83.88.E3.82.A2.E3.83.83.E3.83.97.E3.83.95.E3.82.A1.E3.82.A4.E3.83.AB)
  * でものっけから、「nginxを検討する前に、PHP APCや、WordPressのキャッシュプラグインが、単純にApacheをnginxに変更するよりも大きなパフォーマンス向上をもたらしてくれるかもしれないことに留意しましょう。」とか書いてあり複雑な心境。
  * ほぼ上記設定ファイルがそのまま使えたのですが、php-fpm.sock の場所が違っていたので変更してあります。
  * でも、WordPressの記事の更新したりした感触では、さして早くなっていない印象。
  * ちゃんと外部からアクセスしたら軽く感じられたので一応満足。ストレステストをかけてみる予定

## できあがった設定ファイル



    worker_processes  auto;

    events {
        worker_connections  1024;
    }

    http {
        include       mime.types;
        default_type  application/octet-stream;

        sendfile        on;

        keepalive_timeout  65;

        client_max_body_size 13m;
        index              index.php index.html index.htm;

        fastcgi_intercept_errors        on;
        fastcgi_ignore_client_abort     off;
        fastcgi_connect_timeout 60;
        fastcgi_send_timeout 180;
        fastcgi_read_timeout 180;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_temp_file_write_size 256k;

        # Upstream to abstract backend connection(s) for PHP.
        upstream php {
    	server unix:/run/php-fpm/php-fpm.sock;
        }
        include sites-enabled/*;
    }




    server {
    	server_name www.showway.biz;
    	root /srv/http/showway;

    	if ($http_host != "www.showway.biz") {
       	    rewrite ^ http://www.showway.biz$request_uri permanent;
          	}

    	include global/restrictions.conf;
    	include global/wordpress.conf;

    	access_log /var/log/nginx/showway_access.log main;
    	error_log  /var/log/nginx/showway_error.log;
    }




    server {
            listen 80;
    	server_name wiki.showway.biz;
    	root /srv/http/dokuwiki;
    	location / {
    	    index index.php index.html index.htm;
    	}
    	location ~^/(data|conf|bin|inc)/ { deny all; }
    	location ~^/\.ht { deny all; }
    	location ~^/lib/^((?!php).)*$ { expires 30d; }
    	location ~ \.php$ {
    	    fastcgi_pass  unix:/run/php-fpm/php-fpm.sock;
    	    fastcgi_index index.php;
    	    include fastcgi.conf;
    	}
    	access_log /var/log/nginx/wiki_access.log main;
    	error_log /var/log/nginx/wiki_error.log;
    }




    location = /favicon.ico {
    	 log_not_found off;
    	 access_log off;
    }

    location = /robots.txt {
    	 allow all;
    	 log_not_found off;
    	 access_log off;
    }

    location ~ /\. {
    	 deny all;
    	 access_log off;
    	 log_not_found off;
    }




    location / {
    	try_files $uri $uri/ /index.php?$args;
    }

    rewrite /wp-admin$ $scheme://$host$uri/ permanent;

    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
    	expires 24h;
    	log_not_found off;
    }

    location ~ \.php$ {
    	try_files $uri =404;
    	fastcgi_split_path_info ^(.+\.php)(/.+)$;
    	include fastcgi_params;
    	fastcgi_index index.php;

    	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

    	fastcgi_pass php;
    }


[コメントはまだありません »](http://www.showway.biz/nginx.html#respond)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

# [WordPressでMarkdownを使ってみた](http://www.showway.biz/markdown.html "Permanent Link to WordPressでMarkdownを使ってみた" )

11月 20th, 2015 by onoue  [ コメントはまだありません » ](http://www.showway.biz/markdown.html#respond)

# Markdownテスト

  * Jetpackに付属しているmarkdown機能を使ってみるテスト
  * セットアップはJetpackで有効にするだけ
  * 投稿の設定にmarkdownの設定とかは見つからなかった
  * コメントの設定にはmarkdownの設定があった。
  * さて、**プレビュー**してみるか。
[Markdown](https://ja.wikipedia.org/wiki/Markdown) 段落がいまいちよくわからない。空行で段落分けになるはずだけどならないし。 引用はこんな感じか。

> ” Markdownは、文書を記述するための軽量マークアップ言語のひとつである。もとはプレーンテキスト形式で手軽に書いた文書からHTMLを生成するために開発された。現在ではHTMLのほかパワーポイント形式やLaTeX形式のファイルへ変換するソフトウェア（コンバータ）も開発されている。各コンバータの開発者によって多様な拡張が施されるため、各種の方言が存在する。)”
o
[コメントはまだありません »](http://www.showway.biz/markdown.html#respond)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

Tags: [WordPress](http://www.showway.biz/tag/wordpress) [コンピューター](http://www.showway.biz/tag/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc) [プラグイン](http://www.showway.biz/tag/%e3%83%97%e3%83%a9%e3%82%b0%e3%82%a4%e3%83%b3)

# [Vagrant(ベイグラント)は便利です](http://www.showway.biz/vagrant.html "Permanent Link to Vagrant\(ベイグラント\)は便利です" )

11月 19th, 2015 by onoue  [ コメントはまだありません » ](http://www.showway.biz/vagrant.html#respond)

vagrantがいかに便利かを説明します。開発者にはとても便利なツールです。

Vagrantを使って仮想マシンを作るのってこれだけですよ。これだけ。



    $ sudo pacman -S virtualbox       #virtualboxパッケージのインストール
    $ sudo pacman -S vagrant          #vagrantパッケージのインストール
    $ vagrant plugin install vagrant-vbguest vagrant-share #プラグインのインストール
    $ mkdir -p ~/dev/vagrant/jessie64 #作業フォルダ作成
    $ cd ~/dev/vagrant/jessie64       #作業フォルダに移動
    $ vagrant box add debian/jessie64 #仮想イメージのダウンロード
    $ vagrant init debian/jessie64    #仮想環境の初期化
    $ vagrant up                      #仮想環境の起動
    $ vagran ssh                      #仮想環境へのログイン


次にまた起動したいときもこれだけ。



    $ cd ~/dev/vagrant/jessie64       #作業フォルダに移動
    $ vagrant up                      #仮想環境の起動
    $ vagran ssh                      #仮想環境へのログイン


lxcを使うときもこんだけ



    $ pacman -S lxc arch-install-scripts #パッケージのインストール
    $ vagrant plugin install vagrant-lxc #プラグインのインストール
    $ mkdir ~/dev/vagrant/jessie64lxc    #作業フォルダ作成
    $ cd ~/dev/vagrant/jessie64lxc       #作業フォルダに移動
    $ vagrant init glenux/jessie64-lxc   #仮想環境の初期化
    $ vagrant up                         #仮想環境の起動
    $ vagran ssh                         #仮想環境へのログイン


(lxcを使えるようにするまでは、もう一手間必要ですが。それでも素のlxcを使うよりは断然楽です)

[Vagrent – Archlinux](https://wiki.archlinuxjp.org/index.phphttps://atlas.hashicorp.com/boxes/search?utm_source=vagrantcloud.com&vagrantcloud=1/Vagrant#.E3.83.97.E3.83.AD.E3.83.93.E3.82.B8.E3.83.A7.E3.83.8B.E3.83.B3.E3.82.B0)

「debian/jessie64」「glenux/jessie64-lxc」とかは、「Base Box」といって、以下のサイトのリストからよさそうなものを選びます。

[Vagrant Cloud](https://atlas.hashicorp.com/boxes/search?utm_source=vagrantcloud.com&vagrantcloud=1)

ちょろっと見てみましたが、
「dpsxp/windows-ie10」とか「dpsxp/windows-ie9」とかもあるのですね。ボリュームライセンスがあれば、テストがはかどりそうですね。

追記

作った仮想環境の削除方法



    $ cd ~/dev/vagrant/jessie64lxc #作業フォルダに移動
    $ vagrant destroy              #仮想環境の停止と削除


インストールすると、Base Box が保存されるので、そのファイルは上記コマンドでは削除されないので、手作業で削除します。場所は「~/.vagrant.d/boxes」です。これを知らずに、知らぬまにハードディスクの残り容量が0になってことがありました。

[コメントはまだありません »](http://www.showway.biz/vagrant.html#respond)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

Tags: [コンピューター](http://www.showway.biz/tag/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc) [仮想マシン](http://www.showway.biz/tag/%e4%bb%ae%e6%83%b3%e3%83%9e%e3%82%b7%e3%83%b3)

# [Pebbleアプリを公開しました](http://www.showway.biz/pebble_bso.html "Permanent Link to Pebbleアプリを公開しました" )

10月 27th, 2015 by onoue  [ コメントはまだありません » ](http://www.showway.biz/pebble_bso.html#respond)

[![3](http://www.showway.biz/wp-content/uploads/2015/10/3.png)](http://www.showway.biz/wp-content/uploads/2015/10/3.png) [BSO Baseball Indicator](http://apps.getpebble.com/en_US/application/561cb30d2116352d6600003d) [GitHubリポジトリ](https://github.com/kusanaginoturugi/bso) 野球の審判のためのインジケータを作成しました。 ボール・ストライク・アウトをカウントして表示するだけのアプリです。 ぶっちゃけ、HelloWorldに毛が生えた程度のシロモノですが、実際に使うとかなり便利で、普通のインジケータより使い勝手のよいものになりました。

[コメントはまだありません »](http://www.showway.biz/pebble_bso.html#respond)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

Tags: [IoT](http://www.showway.biz/tag/iot) [Pebble](http://www.showway.biz/tag/pebble) [コンピューター](http://www.showway.biz/tag/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

# [名前を半角カナで入力してしまった](http://www.showway.biz/hankakukana.html "Permanent Link to 名前を半角カナで入力してしまった" )

9月 3rd, 2015 by onoue  [ コメントはまだありません » ](http://www.showway.biz/hankakukana.html#respond)

名前欄に半角カナを入力してしまった。半角カナが悪いわけではないが、いろいろと面倒なので全角に戻したい。 新規に入力する際には全角で入力してもらうが、すでに登録済のデータをどうするか迷った 今回使用しているデータベースはPostgresql。これには便利なConvert関数などはないので自力でなんとかするほかないのだが、探してみると使えそうなストアドプロシージャーがあったので利用させてもらった PostgreSQL でカタカナ変換（ユーザ定義関数）
<http://qiita.com/gold1/items/5b54761697b4ade1f8e9> ローカルのデータベースで動作確認して、次は本番環境での実行。 本番環境はDockerで動かしているので、一手間必要だった。 この際に参考にしたのは以下のサイト。 DockerでWebサーバー/コンテナホストからDBサーバー接続
<http://qiita.com/tsutorm/items/9a707e38038daa6002c0> コンテナのIPアドレスを調べて、そのポートに対して接続する ` $ docker inspect (コンテナの名) | grep IPAddress
$ psql -U postgres -h (コンテナのIPアドレス) -p 5432 (データベース名) < mb_convert_kana.sql
database=# update (テーブル名) set name = mb_convert_kana(name, 'KV'); ` うん、手打ちで入力して方が早かったね。

[コメントはまだありません »](http://www.showway.biz/hankakukana.html#respond)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

Tags: [Postgresql](http://www.showway.biz/tag/postgresql) [コンピューター](http://www.showway.biz/tag/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc) [サーバー](http://www.showway.biz/tag/%e3%82%b5%e3%83%bc%e3%83%90%e3%83%bc) [データベース](http://www.showway.biz/tag/database)

# [メール送信管理アプリを作成しました](http://www.showway.biz/sendmail_manage.html "Permanent Link to メール送信管理アプリを作成しました" )

8月 26th, 2015 by onoue  [ コメントはまだありません » ](http://www.showway.biz/sendmail_manage.html#respond)

メールからの情報漏洩が騒がれる今日このごろですが、ちょうどそれに対応するアプリを作成しました。 Windowsのアプリケーションで、メール送信すると、送信内容をチェックして、設定済みのルールに合致するメールの場合は、ポップアップ表示を出して注意を喚起したり、送信メールを管理用サーバーに送信して、履歴を残すといった処理を行います。

メールの受信には Willpe/SMTP を使用

<https://github.com/willpe/SMTP> 受け取ったメールのパースには MimeKit を使用 <https://github.com/jstedfast/MimeKit> メールの送信には MailKit を使用 <https://github.com/jstedfast/MailKit> これらのライブラリを使うことで、メール固有のノウハウにはあまり触れることなく実装することができました。 デバッグ中に MimeKit の不具合を発見して、プログラムを一部修正して、githubにフォークしたプロジェクトを変更したところ、オリジナルの作者の jstedfast 氏が日本語の適当なコメントにもかかわらず反応してまして、おわっ、これはマズいと再度適切な文章を同僚に作成してもらってコメントを追加して、ようやく作者さんにも問題を理解してもらい、最終的にはオリジナルのプロジェクトの方も修正してもらえたのは、貴重な体験でした。 <https://github.com/kusanaginoturugi/MimeKit/commit/3f1d0c9c83a2fb46d51d82446afd4869a069839a#commitcomment-12613986> オープンソースプロジェクトはこうして使われていく事で磨かれていくのだなと実感しました。 今回は Windows のクライアントアプリだけではなく、サーバーサイドのプログラムも実装しました。サーバーサイドはRailsで作りました。 このあたりにも技術的に面白い話はたくさんあるのですが、長くなるので割愛します。 次もこんな開発をやりたいものです。

[コメントはまだありません »](http://www.showway.biz/sendmail_manage.html#respond)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

Tags: [コンピューター](http://www.showway.biz/tag/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc) [サーバー](http://www.showway.biz/tag/%e3%82%b5%e3%83%bc%e3%83%90%e3%83%bc)

# [AmazonEC2上でDockerを使ってRailsアプリを構築しました](http://www.showway.biz/amazonec2_docker_rails.html "Permanent Link to AmazonEC2上でDockerを使ってRailsアプリを構築しました" )

4月 9th, 2015 by onoue  [ コメントはまだありません » ](http://www.showway.biz/amazonec2_docker_rails.html#respond)

いやあ、紆余曲折ありましたがようやくAWS上にRailsアプリを構築する事ができました。 ちなみに紆余曲折とは以下の通り。

  1. heroku用に作っていたRailsアプリをDocker用の修正
  2. ローカル環境下でDockerで動くことを確認
  3. AWSでDockerを動かすならElasticBeastalkやでと聞く
  4. Beanstalkだと一つのインスタンスで複数のDockerコンテナを収容できないことが判明
  5. さらにRDSを使って設定を間違えて無駄に費用を発生させてみたり
  6. EC2でUbuntuを立ちあげてみる
  7. おう、Ubuntuに入っているDockerのバージョンが1.01。こんな化石使えるかい
  8. 不安にかられながらもAmazonLinuxに移行
  9. おー、AmazonLinuxはDocker1.5だわ。ふつうにDockerが使える幸せ
  10. PostgresをDockerのオフィシャルリポジトリからコンテナ起動
  11. RailsアプリのためにDockerfileを数行書いてposgresにリンクしてコンテナ起動
でな具合です。 わかっていれば、数分の作業に随分時間を取られてしまって残念感たっぷりですが、BeanstalkからRDSまで一通り試せたので経験値がっぽり稼げた感じですね。 Beanstalk+RDSはアプリの規模がスケールアップした時は楽だろうと思いますが、今回の案件は小規模なので不要でした。(ebコマンドの手探り感というか暗中模索感というかかゆいところに手が届かない感がつらかった。ebコマンド使うくらいならec2で直接dockerコマンドを使った方が良いです) 一応、将来の拡張時の保険にはなったので良い勉強でした。でも一年も立つとまた更に便利でパワフルなサービスが出てくるかもしれないので、たいした保険にはならないとは思いますが。 エンジニア的には、サービスをあれこれ使うより、コンソールに入って何でもできる方が効率が良いですね。 EC2も二年前に試した時からパワフルになっていて、コンソールでのレスポンスも良いです。 今回、Dockerを使用したのですが、Dockerを使う事により、サーバー上でのRubyのセットアップ等の作業がまったく必要なく、今後のバージョンアップや更新時にもそのあたりの構築やらなにやらの手間がほぼなくなったのは非常に良いですね。構築スクリプトとか構築手順書とはおさらばです。

[コメントはまだありません »](http://www.showway.biz/amazonec2_docker_rails.html#respond)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

Tags: [コンピューター](http://www.showway.biz/tag/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc) [サーバー](http://www.showway.biz/tag/%e3%82%b5%e3%83%bc%e3%83%90%e3%83%bc) [仮想マシン](http://www.showway.biz/tag/%e4%bb%ae%e6%83%b3%e3%83%9e%e3%82%b7%e3%83%b3)

# [Gnucash使ってますか？](http://www.showway.biz/did_you_use_gnucash.html "Permanent Link to Gnucash使ってますか？" )

3月 8th, 2015 by onoue  [ 1 comment » ](http://www.showway.biz/did_you_use_gnucash.html#comments)

Gnucashとはフリーの財務ソフトウェアです。
<http://www.gnucash.org/index.phtml?lang=ja_JP> 確定申告の時期になると(というかその時期だけ)とても利用頻度の高まるアプリケーションです。 私も使いはじめて今年で三年目となります。 さすがに三年目ともなると、使い方にも慣れてはくるのですが、
どうしても許せない点があります。 それは日付入力時にキーボードのプラス(+)を押すと日付が一週間後になるという点です。テンキーでの動作は問題ないのですが、Shiftキーと併用して入力するプラスの場合だけ動作がおかしいのです。
確定申告の締切直前にこんなことを調べてしまう私もバカだと思いますが、ソースを調べてみましたよ。
たしかにこれだとJISキーボードの場合に+を押すと一週間後になってしまいますね。さらに=の場合も一週間後なので、JISキーボードの場合は一日後を押せるキーがないというオチ。JIS配列が腐ってるのも確かだけど、このコードの書き方はないわ。英語キーボード使ってないやつなんているの？って事じゃないか。 ソースに注意書きは書いてはあるものの直っていない様子だし。
とりあえず7日後は必要ないので、7を1に変えてビルドしなおして今日のところは気持ち良く作業を進めたいと思います。(実はこの場所を探すのにえらく手間取りました) src/gnome-utils/dialog-utils.c



        /*
         * Check those keys where the code does different things depending
         * upon the modifiers.
         */
        switch (event->keyval)
        {
        case GDK_KP_Add:
        case GDK_plus:
        case GDK_equal:
            if (event->state & GDK_SHIFT_MASK)
                g_date_add_days (&gdate, 7);
            else if (event->state & GDK_MOD1_MASK)
                g_date_add_months (&gdate, 1);
            else if (event->state & GDK_CONTROL_MASK)
                g_date_add_years (&gdate, 1);
            else
    	    g_date_add_days (&gdate, 1);
            g_date_to_struct_tm (&gdate, tm);
    	return TRUE;


<http://gnucash.sourcearchive.com/documentation/1:2.4.2-1ubuntu1/dialog-utils_8c_source.html>

[1 comment »](http://www.showway.biz/did_you_use_gnucash.html#comments)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

Tags: [コンピューター](http://www.showway.biz/tag/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

# [PayPal支払いを追加しました](http://www.showway.biz/paypal_payment.html "Permanent Link to PayPal支払いを追加しました" )

12月 10th, 2014 by onoue  [ コメントはまだありません » ](http://www.showway.biz/paypal_payment.html#respond)

[![](https://farm2.staticflickr.com/1146/1367190548_c9b2bac32f_m.jpg)](http://www.igosso.net/flk/1367190548.html)
PayPal employee shtuttle / Richard Masoner / Cyclelicious

PayPal(ペイパル)支払いを追加しました。

こちらから支払いが可能です。支払い前にメールで申し込みをお願いします(見積りも作成いたします)。

Web Site Maintainance
---
ウェブコンテンツの修正 : ¥20,000 JPY – 毎月 AdWordsへの広告出稿 : ¥20,000 JPY – 毎月 SEO対策 : ¥20,000 JPY – 毎月
![](https://www.paypalobjects.com/ja_JP/i/scr/pixel.gif)

[コメントはまだありません »](http://www.showway.biz/paypal_payment.html#respond)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

Tags: [Google](http://www.showway.biz/tag/google) [paypal](http://www.showway.biz/tag/paypal) [SEO](http://www.showway.biz/tag/seo) [コンピューター](http://www.showway.biz/tag/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

# [Excelのアドインを作成しました](http://www.showway.biz/excel_addin.html "Permanent Link to Excelのアドインを作成しました" )

10月 10th, 2014 by onoue  [ コメントはまだありません » ](http://www.showway.biz/excel_addin.html#respond)

[![](https://farm4.staticflickr.com/3602/3449717651_74f62f4994_m.jpg)](http://www.igosso.net/flk/3449717651.html)
Excel / Jon Juan

システムのバージョンアップに伴ない、アプリケーションのテストを実施する事になりました。

この作業は動作確認をしつつ、ひたすら画面のキャプチャーを取得してExcelに貼りつけるという単純な作業になります。とはいえ、完全に自動化して実施できるほど簡単でもなく、何度もやる必要のない仕事です。

そこで、すこしでも楽をしようとExcelのアドインを使って、キャプチャーした複数の画像ファイルをエクセルに一括で貼りつけるアドインを使用しました。 アドインを使うと、提出用のExcelブックをマクロで汚すことなく、マクロと同じように作業を自動化できます。

もっともアドインを作って登録するまでが、かなり煩雑な手順が必要なのですが…。

ポイントとしては最近のExcelだと画像を挿入すると、画像そのものが貼られるのではなく、画像のリンクが貼られてしまうので、そのままでは提出用としては使いずらいものとなります。そこで一度貼りつけた画像をクリップボードにコピーして、再度同じ場所にペーストする事で、画像がリンクになる事を回避しています。(Excel上では後からではリンクを内部画像に変換できないようです)

元ネタはどこかの掲示板で取得したマクロです。先人の努力に感謝です。 以下にそのソースを掲載します。

[» Read more: Excelのアドインを作成しました](http://www.showway.biz/excel_addin.html#more-340)

[コメントはまだありません »](http://www.showway.biz/excel_addin.html#respond)

Posted in [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

Tags: [Excel](http://www.showway.biz/tag/excel) [Microsoft](http://www.showway.biz/tag/microsoft) [コンピューター](http://www.showway.biz/tag/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc)

[« Older Entries](http://www.showway.biz/page/2)

  *     * [ Subscribe Subscribe to my blogs feed ](http://www.showway.biz/feed "Subscribe to my feed - You'll be happy!" )
  * About

    * 山梨県笛吹市でソフトウェア業を営んでおります。
  * 固定ページ

    * [紹介](http://www.showway.biz/about)
    * [実績リスト](http://www.showway.biz/actual_list)
    * [お問い合わせ](http://www.showway.biz/contact_form)
  * アーカイブ

    * [2015年12月](http://www.showway.biz/2015/12)
    * [2015年11月](http://www.showway.biz/2015/11)
    * [2015年10月](http://www.showway.biz/2015/10)
    * [2015年9月](http://www.showway.biz/2015/09)
    * [2015年8月](http://www.showway.biz/2015/08)
    * [2015年4月](http://www.showway.biz/2015/04)
    * [2015年3月](http://www.showway.biz/2015/03)
    * [2014年12月](http://www.showway.biz/2014/12)
    * [2014年10月](http://www.showway.biz/2014/10)
    * [2014年3月](http://www.showway.biz/2014/03)
    * [2013年9月](http://www.showway.biz/2013/09)
    * [2013年8月](http://www.showway.biz/2013/08)
    * [2013年1月](http://www.showway.biz/2013/01)
    * [2012年11月](http://www.showway.biz/2012/11)
    * [2012年9月](http://www.showway.biz/2012/09)
    * [2012年8月](http://www.showway.biz/2012/08)
    * [2011年5月](http://www.showway.biz/2011/05)
    * [2011年2月](http://www.showway.biz/2011/02)
    * [2011年1月](http://www.showway.biz/2011/01)
    * [2010年11月](http://www.showway.biz/2010/11)
  * カテゴリー

    * [コンピューター](http://www.showway.biz/category/%e3%82%b3%e3%83%b3%e3%83%94%e3%83%a5%e3%83%bc%e3%82%bf%e3%83%bc) (20)
  * サービス

    * [24hサーバー監視](https://www.cman.jp/network/)
    * [商用無料の写真検索さん](http://www.nairegift.com/freephoto/)
    * [私的MyDns.jp](http://www.mydns.jp "信頼性の高いダイナミックDNSのサービスです。" )
  * ポートフォリオ

    * [PIOソフトウェア](http://pio-base.herokuapp.com "歯科医師学会へ提出する論文に添付するサンプルアプリケーションです" )
    * [株式会社アースワークス](http://ews.jp "ウェブサイトの構築のお手伝いをさせていただきました。" )
    * [雨宮建築計画事務所](http://amemiya-archi.biz/ "ウェブサイトの構築のお手伝いをさせていただきました。" )
  * 会員

    * [クラウドワークス](https://crowdworks.jp/public/employees/3772)
    * [コンピュータ利用促進協同組合](http://www.ccp.or.jp/)
    * [ランサーズ](http://www.lancers.jp/affiliate/track?id=87015&link=%2F)
  * 協業

    * [コンピュータ利用促進協同組合](http://www.ccp.or.jp/)
    * [株式会社アイウィーヴ](http://www.iweave.jp/ "一緒に仕事をさせていただいております。" )
    * [茂手木歯科医院](http://www8.plala.or.jp/motegishikaiin/)
  * 職歴

    * [GPSゴルフナビ](http://golfnavi.jp/)
    * [冬水社オンラインショッピングサイト](https://www.tosuisha.com/)

Back to Top

© 2016 昌永ウェブサービス · Proudly powered by [WordPress](http://wordpress.org/ "Blogsoftware by Wordpress" ) &amp; [Green Park 2](http://cordobo.com/green-park-2/ "Cordobo Green Park 2 Beta 5" ) by [Cordobo](http://cordobo.com/ "Webdesign by Cordobo" ).

Valid XHTML 1.0 Transitional | Valid CSS 3

![Cordobo Green Park 2 logo](http://www.showway.biz/wp-content/themes/cordobo-green-park-2/img/logo-cgp2.png)
