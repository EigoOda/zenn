---
title: "Shellscript(bash) TIPS"
emoji: "🤮"
type: "tech"
topics: ["bash", "shellscript"]
published: true
---

## 想定する読者

* shellscriptをたまに書く方
* shellscript初心者

### #!(シェバン)

日本語の記事多くでは、shellscriptの1行目を以下のように書かれているものが多いかと思います。
`#!/bin/bash`
これは、/bin/bashにbashがあることが前提になっていて、bashのpathが/bin/bashでないユーザが`./shellscript.sh`のように実行した場合、エラーになってしまうため、あまり良くないです。
できれば、`#!/usr/bin/env bash`としたほうが望ましいです。

[参考](https://stackoverflow.com/questions/10376206/what-is-the-preferred-bash-shebang)


### set コマンド

shellのオプションを設定(set)したり、解除(unset)したりします。

書き方としては、`set -u`や`set -eux`などと書き
不思議ですが、-uでuオプションを設定し、+uでuオプションを解除します。

代表的なところだと
e: コマンドが0以外のステータスコードを返した場合、エラーとなる。
u: 実行したコマンドに定義されていない変数が入っていた場合、エラーとなる。

[参考](https://linuxcommand.org/lc3_man_pages/seth.html)

簡単に説明すると
```bash
#!/usr/bin/env bash
set -u

echo $FOO <- エラーとなる

set +u

echo $FOO <- エラーにならず、ターミナルには何も出力されない
```



### if 構文

知らず使っている方がいるかも知れませんが、
if 文の以下`[ -z "$FOO" ]`の部分ですが、if構文の機能ではなく、[testコマンド](https://linuxjm.osdn.jp/html/GNU_sh-utils/man1/test.1.html)を実行しています。
そのため、ググるときは、`if オプション`ではなく、`testコマンド`で調べたほうが希望の結果がでるかもしれません。

```bash
if [ -z "$FOO" ]; then
  echo "$FOO"
fi
```

ちなみにtestコマンドの`-z`フラグは、`引数に渡された変数が空の場合、正`です。

```bash
$ test -z $FOO <- FOOに何も入っていない場合、正
$ echo $?
0
$ test ! -z $FOO <- FOOに何か入っていた場合、正(!は否定を意味します)
$ echo $?
1
```

書くのが途中になってしまいましたが、モチベがなくなってしまったので、とりあえず公開だけしておきます。
気が向いたら、付け足します。
