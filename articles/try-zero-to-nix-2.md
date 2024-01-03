---
title: "Nix を試してみる part2"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nix"]
published: false
---

:::message
インフォメーション
本記事は、[Nix を試してみる part1](https://zenn.dev/johnn26/articles/try-zero-to-nix) の続きです
:::


### [5. Search for Mix packages](https://zero-to-nix.com/start/nix-search)

Nix packeges は 80,000 以上あるとのこと。
本セクションでは Nixpkgs から nix search コマンドでパッケージを検索する方法を学びます。

Nixpkgs とは、Nix エコシステム用のパッケージやビルドユーティリティが含まれている大きなコレクションです。Nix standard library と呼ばれることもあるそうです。
NixOS モジュールや Nix システムのビルドやデプロイに必要なパッケージも含まれています。

ドキュメント通り、コマンドを実行し Nixpkgs から cargo を検索します。

```bash
$ nix search nixpkgs cargo

* legacyPackages.aarch64-darwin.cargo (1.74.0)
  Downloads your Rust project's dependencies and builds your project

...
* legacyPackages.aarch64-darwin.rust-audit-info (0.5.2)
  A command-line tool to extract the dependency trees embedded in binaries by cargo-auditable
```

nix search の初回実行だったためか、とても実行に時間がかかりました。
パッケージの検索結果は、legacyPackages.{system}.{package} の形式で表示されており、上記実行コマンドの1行目の結果をみると aarch64-darwin(Applie Silicon) のパッケージがヒットしていることがわかります。

検索結果を json で出力したい場合は、`nix search nixpkgs cargo --json`としてください。


### 6. Turn your project into a flake

Nix flake を自分のプロジェクトで有効にする方法を学びます。
前のステップで何度かでてきた [Nix flake](https://zero-to-nix.com/concepts/flakes) とは、[Nix code](https://zero-to-nix.com/concepts/nix-language)（Nix の設定ファイルやパッケージビルドなどで使用される言語） を参照、共有するシステムだそうですが、未だに理解しきれていません。

とりあえずドキュメントどおりにコマンドを実行してみます。

```bash
$ nix run "https://flakehub.com/f/NixOS/nixpkgs/*.tar.gz#fh" -- init

Let's build a Nix flake!
> An optional description for your flake:
> Which systems would you like to support? You selected 1 system: aarch64-darwin
> Which Nixpkgs version would you like to include? latest stable (currently 23.05)
> This seems to be a JavaScript/TypeScript project. Would you like to initialize your flake with some standard dependencies for JavaScript/TypeScript? No
> Add any of these standard utilities to your environment if you wish
> Would you like to add our recommended Nix formatter (nixpkgs-fmt) to your environment? Yes
> Would you like to add doc comments to your flake that explain the meaning of different aspects of the flake? Yes
> Would you like to add any environment variables? No
> Would you like to add a shell hook that runs every time you enter your Nix development environment? No
> Would you like to support legacy Nix commands like `nix-build` and `nix-shell`? No
> Would you like to add your new Nix file to Git? No
> Would you like to add a .envrc file so that you can use direnv in this project? No
Your flake is ready to go! Run `nix flake show` to see which outputs it provides.

```
