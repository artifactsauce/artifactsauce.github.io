---
title: クソアプリ実行プラットフォームとしてのBitBar
updated: 2016-12-09 01:07:00+09:00
---

この記事は [クソアプリ Advent Calendar 2016](http://qiita.com/advent-calendar/2016/kusoapp) の9日目の記事です。

技術ネタが中心ではありますが、若干ポエミーです。

## BitBar

### Macのメニューバー拡張ユーティリティ

[BitBar](https://getbitbar.com/) というアプリケーションをご存知ですか？端的に言うと「Mac OS Xのメニューバーを拡張するユーティリティ」です。メニューバーの拡張の方法は数あれど、BitBarがちょっと革命的だったのは、その拡張プラグインの実装方法です。拡張プラグインはbashを始めとするスクリプト言語で実装でき、スクリプトが標準出力に出力した内容によってメニュー項目が構成されるという簡便さです。またメニュー項目の更新は、定期的にそのプログラムを再実行するという直感的すぎる方法であり、その実行間隔の指定もルールに従ったファイル名にするだけです。

#### 標準出力で構成されるメニュー項目

具体的にその出力内容を見ていきましょう。

前述の通り、プラグインは基本的にスクリプト言語で実装できます。

下記のように出力するだけでメニューバーに "hoge" という文字列が表示されるプラグインの完成です。

```bash
echo "hoge"
```

下記のように出力すると "hoge" と "fuga" がメニューバーにローテーションで表示されます。

```bash
echo "hoge"
echo "fuga"
```

下記のように出力すると、メニューバーに "Menu" という文字列が表示され、この文字列をクリックすると、メニュー項目として "hoge" と "fuga" が作成されます（この状態ではこのメニュー項目を選択することはできません）。

```bash
echo "Menu"
echo "---"
echo "hoge"
echo "fuga"
```

下記のように出力すると、日時（`date`コマンドの出力）が項目名となったメニューが作成されます（同じく選択するはできません）。

```bash
echo "Menu"
echo "---"
date
```

更なる機能の説明は省きますが、どれほど簡単にメニューが作成できるかということはご理解いただけたかと思います。

#### ファイル名の命名規則

プラグインは定期的に実行されることを前提としています。プラグインとして機能させるためには、スクリプトをルールに従ったファイル名で保存し、 **実行権限を付け** 、所定のディレクトリ（変更可）に配置する必要があります。そのルールとは、下の3つのパートをピリオドで区切るというものです。

1. 任意の名称
2. 時間間隔
3. 拡張子

例えば `hoge-fuga.1m.sh` ならば、1分ごとに実行されます。実行される度にメニューの内容は更新されます。

### 素晴らしすぎるプラグインの紹介

[BitBarの本家サイト](https://getbitbar.com/)では、ユーザーから提供されたプラグインを公開しています。その一つを見てみましょう。

- [Unix Time on BitBar - Put anything in your Mac OS X menu bar](https://getbitbar.com/plugins/Time/unixtime.5s.sh)

そして、そのソースコードがこちらです。

```bash
#!/bin/bash

# <bitbar.title>Unix Time</bitbar.title>
# <bitbar.version>v1.0</bitbar.version>
# <bitbar.author>Mat Ryer</bitbar.author>
# <bitbar.author.github>matryer</bitbar.author.github>
# <bitbar.desc>Displays unix time.</bitbar.desc>
# <bitbar.image>http://i.imgur.com/h2cyuYu.png</bitbar.image>

date +%s
```

…え？

メニューバーにUNIXタイムを表示する…だけ？

それって、何が嬉しいの…？

しかもこれ、BitBarの作者本人の提供したプラグインだ…。

これ…他人様に見せるようなもの？

さすがにこれは誰も使わないんじゃない？

本家サイトにはそれなりの数のプラグインが公開されていますが、その内の実にかなりの数のものが「これ、なんで公開したの？」というものだったりします。何の役にも立たないとまでは言いませんが、少なくともメニューバーに常に表示させておく意味は無いんじゃないかと思われるものが多々あります。

## 俺得アプリ

良いんです！！

それで、良いんです！！！

（博多華丸風に読み取ってください）

自分しか使わないようなプラグインを作ったって、良いじゃないですか？誰でも簡単に使えるプラットフォームがあるんだったら、むしろ誰も実装してくれないひどく個人的な要望を叶えるツールを作りましょうよ。あなたしか使わないようなツールを誰が書いてくれるでしょうか？あなたが書きましょうよ。

「メニューバーは基本的には常に表示されている」という点だけに着目し、新たに俺得なプラグインを考えてみましょう。私はここ最近、そのような観点で作成した以下のようなプラグインを作り、メニューバーに常駐させています。

- [BitBarで自分の人生の残り時間を常に意識する](http://qiita.com/artifactsauce/items/1caca90e91c6ca90ebbf)
- [BitBarで自分のやるべきタスクを常に意識する](http://qiita.com/artifactsauce/items/2f933a1dbddf19ab008a)
- [BitBarでMySQLサーバーの状態を常に意識する](http://qiita.com/artifactsauce/items/d111bb1e6676f235094d)
- [BitBarでPostgreSQLサーバーの状態を常に意識する](http://qiita.com/artifactsauce/items/445535ea8a58253c2f28)

この内、「人生の残り時間」以外のものは[BitBar Plugin](https://github.com/matryer/bitbar-plugins)にPull Requestを送り、すべてAcceptされて本家サイトのリストにも掲載されました。俺得な上に承認欲求も満たされて感無量です。

## なぜそんな俺得アプリを公開するのか？

俺得アプリを書くのは良いとして、それを公開することにどれだけの意味があるのでしょう？

それが非常に特殊な要望であったとして、全く同じ要望を持っている人は居ないかもしれませんが、同じような要望を持っている人は居ると思います。事実、私が前述の「タスクを常に意識する」で作ったToDoリストのプラグインは、本家のリストにも含まれている [Simple Todo Tracker on BitBar](https://getbitbar.com/plugins/Lifestyle/todo.30s.sh) を元にしています。残念ながらコードとして利用している箇所はありませんが、そのコンセプトを見ることで、私が求めている要件がBitBarで実現できそうだということに気がつくことができました。

そのままでは利用されなかったとしても、そのアイデアは誰かの役に立つかもしれない。自分しか使わないようなものであったとしても、コードを公開する価値はあるのです。

## そして新しいプラグイン - その１

最近、また新しいプラグインを書きました。頻繁に使うファイルやディレクトリを、目的に合わせたアプリケーション（今回はFinderまたはAtom）で開くためのショートカットです。

```bash
#!/usr/bin/env bash

set -eu

cat <<EOD
:open_file_folder:
---
Open in default | color=#c06547
  家の図面 | bash=/usr/bin/open param1=$HOME/Dropbox/Layouts/home.png terminal=false trim=false
Open in Atom | color=#c06547
  Package list | bash=/usr/local/bin/atom param1=$HOME/src/github.com/artifactsauce/proglets/share/mpkg terminal=false trim=false
  BitBar Plugin | bash=/usr/local/bin/atom param1=$HOME/Dropbox/BitBar terminal=false trim=false
---
Refresh | refresh=true color=#C0C0C0
EOD
```

すべてをヒアドキュメントにしてあり、それを `cat` で標準出力に出力しています。言ってみれば、手書きでメニューを書いているだけです。しょうもないプラグインですが、こういう使い方もできるんだということを示すためには良いサンプルかと思います。

本家サイトの基準からすると公開しても良さそうなレベルかもしれませんが、プライドが邪魔してPull Requestを出すまでには至っていません。ですので、ここで公開します。

## そして新しいプラグイン - その２

もう一つご紹介します。 `.DS_Store` ファイルを削除するプラグインです。[`.DS_Store`](https://en.wikipedia.org/wiki/.DS_Store)とは、言わずと知れたmacOSの迷惑（？）な隠しファイルです。もちろんそれなりに役割はあるのですが、迷惑を被ったことはあれど恩恵に預かった覚えがないこのファイルは非常に目障りです。目に触れる度に削除していたのですが、それも面倒なのでプラグインを書きました。自動で削除しようかとも思ったのですが、念のために確認した後で削除することにしました。

```bash
#!/usr/bin/env bash

set -eu

FIND_CMD_BASE="find $HOME -path \"$HOME/Library\" -prune -o -name '.DS_Store'"

if [[ $# -eq 1 && $1 = "delete-all" ]]; then
  eval ${FIND_CMD_BASE} -exec rm -f {} +
  exit $?
fi

LF=$'\x0A' # return code
ITEMS=""
declare -i FILE_CNT=0

while read
do
  [[ -z $REPLY ]] && continue
  ITEMS="$ITEMS$REPLY | bash=/bin/rm param1=-f param2=\"$REPLY\" terminal=false$LF"
  (( FILE_CNT++ ))
done <<EOS
$(eval ${FIND_CMD_BASE} -print)
EOS

echo ":black_joker::$FILE_CNT"
echo "---"

if [ $FILE_CNT -ne 0 ]; then
  echo -n "$ITEMS"
  echo "---"
  echo "Delete all | color=red bash=$0 param1=delete-all terminal=false refresh=true"
else
  echo "Cleaned up"
fi
echo "---"
echo "Refresh | refresh=true color=#C0C0C0"
```

結果的には自動で削除しても良かったんじゃないかと思うほど "Delete All" を実行して全て削除しています。

心の健康に非常に貢献しています。鬱陶しいゴミが存在していることも、それがどこに存在してるのかわからないことも、非常に強いストレスでしたが、このプラグインで完全に解消されました。でも、そんな病的な感覚を持っているのは世界中見渡しても僕だけなんじゃないでしょうか？これもまた、俺得なプラグインと呼んで差し支えないかと思います。

## 最後に

誰の役にも立たず、自分のためだけに存在するようなアプリケーションを書くことを恥じる必要はありません。むしろそれを公開してしまいましょう。それが誰かのアイデアの種になるかもしれませんし、千人に一人くらいは喜んで使ってくれる人がいるかもしれません。恥ずかしい生き方かもしれませんが、少なくとも僕はそうやって生きています。まだ死んでいないので、それでも生きられるということの証明の一つかと思います。
