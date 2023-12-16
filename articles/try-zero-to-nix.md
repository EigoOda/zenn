---
title: "Nix を試してみる part1"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nix"]
published: true
---

:::message
インフォメーション
本記事は [ZOZO Advent Calendar 2023](https://qiita.com/advent-calendar/2023/zozo) の16日目の記事です
:::

毎週、[Infra Weekly](https://infraweekly.substack.com/) を見ているのですが、最近 Nix の記事が多かったり、去年辺りから [NixCon](https://2023.nixcon.org/) というものが開催されてたりで面白そうだったので [Zero to Nix]((https://zero-to-nix.com/)) を試してみます。
良さそうでされば、brew や aqua からの移行も考えています。

ちょうど最近、George Hotz も [Youtube](https://youtu.be/v7lIGYU0onA?si=RMM-G_MKP-qGSsf-) で Zero to Nix をやってたので、動画で見られたい方は参考になると思います。

8ステップありますが、初見かつボリュームが多そうなので2つの記事に分けて書きたいと思います。

## 試してみる

### [1. Get Nix running on your system](https://zero-to-nix.com/start/install)

まずは、Nix をインストールします。
大体のプラットフォームに対応してそうです。

![](/images/try-nix/nix-platform.png)

```bash
$ curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install
...
Nix was installed successfully!

$ nix --version
nix (Nix) 2.18.1
```

## [2. Run a program with Nix](https://zero-to-nix.com/start/nix-run)

Nix を使ってプログラムを実行します。

```bash
$ echo "Hello Nix" | nix run "nixpkgs#ponysay"
error: cannot open SQLite database '/Users/eigo.oda/.cache/nix/fetcher-cache-v1.sqlite': unable to open database file
```

エラーになりました...
https://discourse.nixos.org/t/nix-flake-and-fetcher-cache-v1-sqlite/8958 を見ると `~/.cache/nix` の所有者を変更するとあるので、現在の所有者を見てみます。

```bash
$ ls -la ~/.cache/
...
drwxr-xr-x@    - root     16 Dec 15:40 nix
```

root になっていたので、chown で今ログインしているユーザに変更し、`nix run`コマンドで`ponysay`を実行します。
初回の場合、ビルドしたりダウンロードしたりストアに格納したりで実行が遅くなりますが、気長に待ちましょう。

```bash
$ chown eigo.oda ~/.cache/nix

$ ls -la ~/.cache
...
drwxr-xr-x@    - eigo.oda 16 Dec 16:14 nix

$ echo "Hello Nix" | nix run "nixpkgs#ponysay

 ___________
< Hello Nix >
 -----------
       \
        \
         \
         ▄▄▄▄ ▄▄▄▄
   ▄▄▄▄▄████▄▄████▄▄▄
   ██████▄▄▄▄███▄█████
    ▀▄██▄▄█▄▄██▄▄▄██▄██
     ▀▄█▄███▄▄▄█████▄██
     ▀▀▀ ▀▄█▄▄▄▄▄▄███▄▄▄
        ▄▄▄███▄▄▄█▄███▄██
       ▄▄▄▄█▄▄█▄▄▄██▄▄██      ▄▄▄▄▄
       ▀▄▄▄██▄▄██████▄▀     ▄▄███▄▄▄
         ▀▄▄▄▄▄▄▄███▄        ▀▄▄▄▄▄▀
              ██▄▄▄██▄▄▄▄▄█▄▄▄▄▄▀
              █▄█▄▄▄▄▄█████████▄▄
              █▄▄███▄▄██▄████████
             ▄▄█▀▄▄▄█████▄▄████▄▀
             ███ ███████▄█▄▄████▄
             ▀▀▀ ██████▀ ▀▄████▄██
                ▄▄█▄▄▄█   ████▄▄▄▄▄
                ███████   ▄▄▄█▄▄▄▄█
              ▄█▄▄▄▄▄▄▄  ▄▄█████████
              █▄███████  █▄▄▄▄██████
               █▄▄▄▄▄█      █▄▄▄▄▄▄▀

```

無事、`ponysay`を実行することができました。
`nix run`の詳細は[こちら](https://zero-to-nix.com/start/nix-run#explanation)をご確認ください。

## [3. Explore Nix development environments](https://zero-to-nix.com/start/nix-develop)

Nix の development environments という機能を使います。
shell システムを構築するもののようで、ホストから分離されているため他の環境やツールから変更を加えることができない環境です。

```bash
$ nix develop "github:DeterminateSystems/zero-to-nix#example"

(nix:zero-to-nix-env) xxxx:nix eigo.oda$
```

`nix develop`を実行すると新しい shell のプロンプトが現れました。
この shell には、curl と git が含まれているので、確認してみます。

```bash
(nix:zero-to-nix-env) xxxx:nix eigo.oda$ type git
git is /nix/store/mgp3sykmxzaf8r42fwc51239qh4j27cy-git-2.40.1/bin/git

(nix:zero-to-nix-env) xxxx:nix eigo.oda$ type curl
curl is /nix/store/80cz8a3bvkps2ynphvhimbvdygdk0hyw-curl-8.1.1-bin/bin/curl

(nix:zero-to-nix-env) xxxx:nix eigo.oda$ type kubectl
kubectl is /Users/eigo.oda/.local/share/aquaproj-aqua/bin/kubectl

```

ローカルホストには、curl と git をインストールしていますが、Nix で用意されたものが表示されました。
kubectl はローカルホストにはインストールされているのですが、`nix develop`で用意した shell にはインストールされていないので、ローカルホストのパスが表示されました。

development environments で用意された shell のコマンドパスが奇妙に見えますが、以下のような命名規則となっています。

![](/images/try-nix/nix-store.png)

また、shell にログインせず development environments にコマンドを実行することもできます。

```bash
$ nix develop "github:DeterminateSystems/zero-to-nix#example" --command git help

$ nix develop "github:DeterminateSystems/zero-to-nix#example" --command curl https://example.com
```

以下コマンドで Go の development environments を構築できます。

```bash
$ nix develop "github:DeterminateSystems/zero-to-nix#go"
Welcome to a Nix development environment for Go!

(nix:nix-shell-env) xxxx:nix eigo.oda$ type go
go is /nix/store/sf7mynigbcqs2j18j4cpqnfrzg2rf5am-go-1.19.12/bin/go

(nix:nix-shell-env) xxxx:nix eigo.oda$ go version
go version go1.19.12 darwin/arm64
```

特定のパッケージを組み合わせた環境も構築できます。
以下の環境は、Pyton、kubectl、Terraform、OpenSSL が入った環境です。

```bash
$ nix develop "github:DeterminateSystems/zero-to-nix#multi"
```

自分が利用するパッケージのみがインストールされた環境の構築方法が気になりました。

## [4. Build a package using Nix](https://zero-to-nix.com/start/nix-build)

ついにパッケージ管理まで来ました。パッケージ（bat）をビルドします。

```bash
mkdir build-nix-package && cd build-nix-package
nix build "nixpkgs#bat"
```

ディレクトリに result という名前のシンボリックリンクが作成され、Nix store のパッケージと紐づいています。

```bash
$ ls -la
lrwxr-xr-x@ 54 eigo.oda 17 Dec 04:12 result -> /nix/store/jvzjjz8nrgqayqhmfha26a77s2kha82j-bat-0.24.0

$ ./result/bin/bat --help
A cat(1) clone with syntax highlighting and Git integration.

Usage: bat [OPTIONS] [FILE]...
       bat <COMMAND>
```

bat コマンドが叩けることを確認できました。
development environments 同様、こちらも命名規則があります。

![](/images/try-nix/nix-package.png)


# 終わりに

Zero to Nix の前半4ステップを試してみました。
`nix run` に関しては docker などのコンテナプラットフォームと似たようなものを感じましたが、コンテナと比べてホストのコマンドも実行できるのが嬉しいですね。
次回は、後半の4ステップを実行しようと思います。
