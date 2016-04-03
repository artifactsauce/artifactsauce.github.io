---
title: RedmineをCentOS 5にインストールする
updated: 2010-06-14 0:45
---

## 概要

CentOS 5にRedmineをインストールする。原則として yum でインストールできるものは yum を使う。また、一つのサーバー、一つのホスト名で複数のRailsアプリケーションを運用することも想定する。

## 運用環境

Redmineの運用環境はApache2.2 + mongrel cluster + mod\_proxy\_balancer を採用する。URLが `http://example.com/redmine/` のように、ディレクトリ階層を一つ下げて運用する。

## 必要なパッケージのインストール

このシステムにはPuppetをインストールしてあるため、Rubyもまた既にインストールされている。Ruby on Railsは、通常 gem でインストールするものだが、[EPEL (Extra Packages for Enterprise Linux)](https://fedoraproject.org/wiki/EPEL) にいくつかの gem パッケージが rpm 化されているので、これを利用する。インストールするパッケージは最終的には３つで、コマンドは下記の通り。

```
$ yum install rubygems rubygem-rails rubygem-mongrel_cluster
```

## Redmineの設置

現在の最新版は0.9.4。RubyForgeから取得する。

```
$ wget http://rubyforge.org/frs/download.php/70486/redmine-0.9.4.tar.gz
$ tar zxvf redmine-0.9.4.tar.gz
$ cd redmine-0.9.4
$ cp config/database.yml.example config/database.yml
$ vi config/database.yml
```

環境に合わせて適当に編集する。 Redmineのヘルパースクリプトがこの設定ファイルに従って、データベースにアクセスし、テーブルを作成する。

```
$ rake db:migrate RAILS_ENV="production"
(in /home/username/redmine-0.9.4)
rake aborted!
Could not find RubyGem rack (~> 1.0.1)

(See full trace by running task with --trace)
```

`rack` をインストールしなければ実行できないというメッセージが現れる。EPELには `rubygem-rack` というパッケージがあるのでこれをインストールしてみるが

```
$ rake db:migrate RAILS_ENV="production"
(in /home/username/redmine-0.9.4)
rake aborted!
RubyGem version error: rack(0.4.0 not ~> 1.0.1)

(See full trace by running task with --trace)
```

今度は `rack` のバージョンが古いので実行できないというメッセージ。EPELパッケージを使っている限りは最新版のRedmineは使用できないということがわかる。それではどのバージョンなら使えるのかと言うと、結果的に0.8.7ならいける。

```
$ wget http://rubyforge.org/frs/download.php/67144/redmine-0.8.7.tar.gz
$ tar zxvf redmine-0.8.7.tar.gz
$ cd redmine-0.8.7
$ cp config/database.yml.example config/database.yml
$ vi config/database.yml
```

同様に設置して設定ファイルを編集する。さっきのをコピーしてもいいと思う。編集したらヘルパースクリプトを実行する。

```
$ rake db:migrate RAILS_ENV="production"
(in /home/username/redmine-0.8.7)
rake aborted!
Missing session secret. Please run 'rake config/initializers/session_store.rb' to generate one

(See full trace by running task with --trace)
```

セッションを保存するための処理をしておけということなので、素直に実行する。

```
$ rake config/initializers/session_store.rb
(in /home/username/redmine-0.8.7)
```

もう一度 `rake db:migrate` を行えば、データベースの方にテーブルが作成される。 テーブルが作成されたらサーバープロセスを立ち上げる。

```
$ ruby script/server -e production
```

通常は3000番ポートで起動するので [http://localhost:3000/](http://localhost:3000/) にアクセスして確認する。サービスの起動を確認したら、 Ctrl-C でプロセスを停止する。

コンテンツをユーザーホーム内に置いておくのもナンなので、適当な場所に移動しておく。今回は `/var/www/redmine/` におくことにする。所有者は `apache` ユーザーが適当だと思う。また、バージョンが変わっても他の環境設定に影響を及ぼさないためにリンクを作成しておく。

```
$ sudo mv redmine-0.8.7 /var/www/
$ cd /var/www/
$ sudo chown -R apache:apache redmine-0.8.7
$ sudo ln -s redmine-0.8.7 redmine
```

## mongrel clusterの設定

今回はmongrelで起動するだけではなく、マルチコアを効率的に利用するためにも複数プロセスを起動して運用することにする。EPELでは `mongrel_cluster` を rpm パッケージ化してある上、粋な設定環境を用意してくれてあるので大変助かります。基本的にディレクトリ `/etc/mongrel_cluster/` にmongrel設定ファイルを配置しておく（1つのアプリケーションに1つの設定ファイル）と、起動スクリプト `/etc/init.d/mongrel_cluster` による通常の操作（ `start|stop|restart` ）ですべてのアプリケーション設定ファイルを自動的に読み込んで起動／停止を行ってくれる。もちろん、システム起動時に自動でデーモンを起動するのも簡単。

まずは、mongrel設定ファイルを作成する。これもサポートスクリプトがあるから非常に簡単。まずはそのヘルプを出力してみる。

```
$ mongrel_rails cluster::configure -h
** Ruby version is not up-to-date; loading cgi_multipart_eof_fix
Usage: mongrel_rails <command> [options]
    -e, --environment ENV            Rails environment to run as
    -p, --port PORT                  Starting port to bind to
    -a, --address ADDR               Address to bind to
    -l, --log FILE                   Where to write log messages
    -P, --pid FILE                   Where to write the PID
    -c, --chdir PATH                 Change to dir before starting (will be expanded)
    -o, --timeout TIME               Time to wait (in seconds) before killing a stalled thread
    -t, --throttle TIME              Time to pause (in hundredths of a second) between accepting clients
    -m, --mime PATH                  A YAML file that lists additional MIME types
    -r, --root PATH                  Set the document root (default 'public')
    -n, --num-procs INT              Number of processor threads to use
    -B, --debug                      Enable debugging mode
    -S, --script PATH                Load the given file as an extra config script.
    -N, --num-servers INT            Number of Mongrel servers
    -C, --config PATH                Path to cluster configuration file
        --user USER
                                     User to run as
        --group GROUP
                                     Group to run as
        --prefix PREFIX
                                     Rails prefix to use
    -h, --help                       Show this message
        --version                    Show version
```

様々なオプションが表示されたが、今回使ったものはそう多くない。実際に実行したコマンドは下記の通り。

```
$ sudo mongrel_rails cluster::configure \
-e production \
-p 3000 \
-c /var/www/redmine \
--user apache \
--group apache \
--num-servers 5 \
--config /etc/mongrel_cluster/redmine.yml \
--prefix /redmine
** Ruby version is not up-to-date; loading cgi_multipart_eof_fix
Writing configuration file to /etc/mongrel_cluster/redmine.yml.
```

- `-c`: まず絶対に必要なものはコレ。起動スクリプトを実行する際のカレントディレクトリを指定できる。これはつまり、ログファイルの出力先やアプリケーションの ドキュメントルートなどを指定する際の基準になる場所ということ。これを指定しておけば、ほとんどのオプションはdefaultで問題ない。
- `--config`: 設定ファイルを出力する先のPATH。これを指定しないとカレントディレクトリに作成されるが、カレントディレクトリがRailsアプリケーションの rootでないとエラーになる（と思う）。今回はmongrel_clusterの設定ファイル格納ディレクトリに直接書き込む。
- `--user`, `--group`: これを指定しないと、プロセスのユーザーが `root` になる。セキュリティ的には好ましくないので、何かしらのユーザー（ `apache` で良いと思う）を設定しておくべき。
- `--prefix`: ディレクトリ階層を下げて運用するためには必要。つまり今回の環境では必要。ここで指定した値は後で利いてくる。
- `-e`: Railsのデータベース設定ファイル中のどの項目を使うのかを指定する。ここまで `production` を指定してきたので、ここでも当然 `production` を指定する。
- `-p`: デフォルトで使用するなら必要のない設定。プロセスを複数起動する場合は、これが最初の番号になり、あとは連番になる。
- `--num-servers`: プロセスを複数起動する場合に、いくつ起動するかを指定する。本番運用する場合は注意して指定する必要があるだろうが、ここでは縁起の良い数字を使った。

## Apacheの設定

以下のモジュールが有効になっている必要がある。

- `mod_proxy`
- `mod_proxy_http`
- `mod_proxy_balancer`

設定ファイルはこんな感じ。おそらく最小限の書き方だと思う。前項 `--prefix` で指定した値は、下記の `BalancerMember` および `ProxyPass` で利いてくる。すべて同じ値（今回は `redmine` ）にしなければいけない。これらの値が違っているとリンクが繋がらなくなる。

```apache
<VirtualHost *:80>
    ServerAdmin username@example.com
    ServerName www.example.com

    ErrorLog /var/log/httpd/error.log
    LogLevel warn

    CustomLog /var/log/httpd/access.log combined

    AddHandler cgi-script .cgi

    DocumentRoot /var/www/html
    <Directory />
        Options FollowSymLinks
        AllowOverride None
        Order deny,allow
        Deny from all
    </Directory>
    <Directory /var/www/html>
        Order allow,deny
        Allow from all
    </Directory>

    ProxyRequests Off

    ProxyPass /redmine/ balancer://bl1/
    ProxyPassReverse /redmine/ balancer://bl1/

    <Proxy balancer://bl1/>
        BalancerMember http://127.0.0.1:3000/redmine loadfactor=10
        BalancerMember http://127.0.0.1:3001/redmine loadfactor=10
        BalancerMember http://127.0.0.1:3002/redmine loadfactor=10
        BalancerMember http://127.0.0.1:3003/redmine loadfactor=10
        BalancerMember http://127.0.0.1:3004/redmine loadfactor=10
    </Proxy>

    <Location /redmine>
        Order deny,allow
        Deny from all
        Allow from 192.168.0.0/24
    </Location>
</VirtualHost>
```

アクセス制限部分はついでに書いておいたもの。仮想的なPATHになるから、 `Direcotory` ではなく `Location` を使用する。ディレクトリ名の最後の `/` （スラッシュ）は、有ったり無かったりするが、これが違っていると正常に動作しない。詳しいことはヤヤコシイから自分で調べて欲しい。

## Redmineのアカウント設定

ここからは完全に余談。RedmineはLDAPで運用したいから認証部分に下記を記述しておく。いつも忘れるから書いておく。

- Redmine: `Authentication`
- Name: `Main LDAP Server`
- Host: `ldap.example.com`
- Port: `389`
- Account: `cn=Manager,dc=example,dc=com`
- Password: `********`
- Base DN: `ou=People,dc=example,dc=com`
- Login: `uid`

AccountとはLDAPサーバーの管理者アカウントのこと。Base DNとは、このシステムのユーザー認証で利用するユーザープロファイルの位置のこと。Loginはユーザー名として利用する項目。すべてOpenLDAP のデフォルト値を使用する。Nameは任意の値なので何でもいい。

## その後の運用

なぜだか一日一回は再起動しないとInternal Errorになる。長時間アクセスしないからなのか、時限爆弾みたいなものなのかはわからないし、調べる気もしない。 cron で毎朝4時に再起動するということにする。公開運用はまず無理だなというのが感想。

## 参考

- [mod_proxy_balancer と Rails との連携まとめ](http://wota.jp/ac/?date=20070605)
- [Setting up Rails Production Server using Apache2, Mongrel Cluster on Fedora Core 5](http://nlakkakula.wordpress.com/2007/07/19/setting-up-rails-production-server-using-apache2-mongrel-cluster-on-fedora-core-5/)
- [Scaling Rails with Apache 2.2, mod_proxy_balancer and Mongrel](http://blog.innerewut.de/2006/04/21/scaling-rails-with-apache-2-2-mod_proxy_balancer-and-mongrel)
