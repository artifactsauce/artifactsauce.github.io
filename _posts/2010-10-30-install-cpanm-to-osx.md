---
title: cpanmをMac OS Xにインストールする
updated: 2010-10-30 23:14:00
---

## 概要

YAPC::Asia 2010に参加してみると、何人もの講演者が「cpanmを使え」と唱えている。「cpanmを使わない理由はない」とまで言うので、cpanmをインストールしてみることにした。ちょうどいろいろあってOSを再インストールしたばかりだったため、割とクリーンな状態で実験できた。

### 参考

- [App::cpanminus](http://search.cpan.org/dist/App-cpanminus/)

## テスト環境

- Mac OS X 10.6.4 (Snow Leopard)
- MacPorts 1.9.1
- perl v5.12.2

## インストール

### 必要なパッケージのインストール

今回はMacPortsでインストールしたv5.12.2を使用しているが、[App::cpanminus](http://search.cpan.org/dist/App-cpanminus/lib/App/cpanminus.pm)のページよると、perlのバージョンは5.8以降なら問題ないそうだ。

また、App::cpanminusのページの例ではファイル取得にcurlを使っているが、実際は何でもいいはずだ。wgetでも問題ないし、その旨も明記されているが、気分的に今回はとりあえずマニュアル合わせるためにcurlをインストールした。

```console
$ sudo port install perl5.12 curl
```

### cpanmのインストール

マニュアル通りにコマンドを実行してみる。

```console
$ curl -L http://cpanmin.us | perl - --sudo App::cpanminus
```

これでちゃんと /opt/local/lib/perl5/site_perl/5.12.2/ にインストールされているし、コマンドも /opt/local/bin/ にインストールされている。とても簡単。

```console
$ perldoc -l cpanm
/opt/local/bin/cpanm
$ perldoc -l App::cpanminus
/opt/local/lib/perl5/site_perl/5.12.2/App/cpanminus.pm
$ which cpanm
/opt/local/bin/cpanm
```

インストールしたら、まずはcpanm自体をアップグレードする。しかし、そもそもこの手法なら最新版をインストールしたはずなのでこの時点でアップグレードされるはずもない。

```console
$ sudo -H cpanm --self-upgrade
App::cpanminus is up to date. (1.0015)
```

最新版を追いかける必要はないが、せっかくだから今回は最新版をインストールした。2010.02にリリースされている新しいモジュールということもあり、現在も頻繁に更新されているそうなので、最新版を入れる利点も大きいと思う。ちなみにDebianやFreeBSDにはcpanmのパッケージが用意されているらしい。

### モジュールのインストール

それでは試しにマニュアルに従ってモジュールをインストールしてみる。

```console
$ sudo -H cpanm Task::Plack
```

これも問題なくインストールされる。もちろんモジュール自身に問題があればインストールは成功しない。

今回は最も単純なCPANからのインストールだったが、他にもローカルファイルやWeb越しからインストールすることができる。詳しくは下記のコマンドで確認してほしい。

```console
$ perldoc cpanm
```
