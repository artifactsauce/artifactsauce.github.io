---
title: システム領域に複数のperl実行環境をインストールする
updated: 2012-06-02 13:26:00
---

## 動機

複数のperlのWebアプリケーションを1つのサーバー中でホストしている人はどれくらいいるだろうか？僕はやっている。リソースが少ない場合はどうしてもこうなってしまうと僕は思う。1つのサーバーで複数のアプリケーションを運用しているのに、サービスをちょっとでも止めるとなじられる上、代替機も無いような人はそれなりに居ると思う。

この環境の場合、perl実行環境を複数持ちたくなることがある。例えば、モジュールのアップデートやperl本体のアップグレードなど、実行環境に大きな変化がある場合に、トラブルがあればすぐ戻すことができるようになる。

通常はインストールPATHを変更する方法や、全体をgitなどでバージョン管理してしまう方法などが考えられるが、今回はperlbrewを用いた方法を考えたい。

## perlbrew

perlbrewは通常、個人のhomeディレクトリに複数バージョンのperlの実行環境を作成したい場合に用いられる。

### 参考

- [perlbrew - D-6 [相変わらず根無し]](http://blog.endeworks.jp/d-6/2010/08/perlbrew.html)
- [Perlbrew](http://perlbrew.pl/)
- [劉康民 / App-perlbrew - search.cpan.org](http://search.cpan.org/dist/App-perlbrew/)

configure時にPATHを変更してインストールする方法と何が違うのかと言うと、用意されたshellスクリプトを実行するだけで環境変数がちょこちょこっと変わってperlのPATHがうまいこと変更されてしまうということ。手元のperlプログラムのshebangを `/usr/bin/env perl` にしている限り、これを変更する必要はない。

## perlbrewのインストールPATHを変更する

デフォルトでperlbrewをインストールすると、 `$HOME/perl5/perlbrew` というディレクトリを作って、そこにいろいろなファイルを配置する。しかし、 `PERLBREW_ROOT` という環境変数を事前に設定しておくことで、任意のディレクトリにインストールすることも可能だ。これを応用して、みんな大好き `/usr/local` にインストールしてみる。

<div><script src="https://gist.github.com/2573393.js?file=install_perl.sh"></script></div>

```console
[INFO] Success to install perl into /usr/local/perlbrew/perls/perl-5.14.2/bin/perl
```

みたいな表示があれば成功しているはずだ。

ポイントは前述のとおり16行目の `PERLBREW_ROOT` の設定と、20行目の `bashrc` の読み込み、25行目の `perlbrew use` だ。 `switch` は使わないようにしよう。

`/usr/local` にインストールする都合上、root権限で実行することを前提にしている。10-12行目はhomeディレクトリをNFSでマウントしている場合に必要になることがあるという僕の個人的な問題のためにある。

## perlモジュールのインストール

この環境の問題はモジュールをインストールするのが面倒だということだ。そのためにshellスクリプトを作っておいた。

<div><script src="https://gist.github.com/2573393.js?file=install_pm.sh"></script></div>
<div><script src="https://gist.github.com/2573393.js?file=pm_list.txt"></script></div>

ポイントは同様に14-16行目。21行目以降の部分でモジュールのインストールをしている。 `pm_list.txt` というファイルにモジュール名を記載しておけば、順番にインストールしてくれるはずだ。この場合、 `pm_list.txt` は `/var/tmp/build` に置いておこう。先頭に `#` を書いておけばコメントアウトもできる。ちなみに `cpanm` でインストールすることを前提にしている。

### 参考

- [ArtifactSauce: cpanmをMac OS Xにインストールする](/posts/2010/10/30/install-cpanm-to-osx)
- [Tatsuhiko Miyagawa / App-cpanminus - search.cpan.org](http://search.cpan.org/dist/App-cpanminus/)

## Webアプリケーションを起動する

それぞれのWebアプリケーションからperlbrewでインストールしたperlを使う場合には、起動スクリプトを作ってしまうのが一番楽だと思う。

<div><script src="https://gist.github.com/2573393.js?file=webapprc"></script></div>
<div><script src="https://gist.github.com/2573393.js?file=myapp.sh"></script></div>

起動スクリプトはmyapp.shで、内部でwebapprcを読み込んでいる。myapp.shをいくつも作れば、複数のWebアプリケーションを起動するのは簡単だ。ポイントは同様にwebapprcの24-26行目になる。これはいくつかの起動スクリプトのサンプルをパクってまぜこぜにして出来上がったから、もはやオリジナルが何なのかさえわからない。ライセンスとかどうなるんだろう？

環境依存の前提条件がいろいろあるからちょっと説明する。Web Application FrameworkにはCatalyst、Web ServerにはStarman、Super daemonにはServer::Starter、OSにはCentOS5、アプリケーションの起動には専用のユーザーwebappを使用している。PSGIアプリケーションだったら、多少の変更で対応できると思う。

## まとめ

perlbrewを/usr/localにインストールするというレアな要望を実現してみた。今のところトラブルは起こっていない。これを応用すると[こういうこと](https://plus.google.com/112707632660981144842/posts/GJHLSeytBD3)ができるようになった。

## 使用したファイルのリポジトリ

- [gist: 2573393](https://gist.github.com/2573393)
