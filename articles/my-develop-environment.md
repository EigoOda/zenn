---
title: "俺の開発環境"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Terminal", "Wezterm", "tmux", "Neovim"]
published: true
---

:::message
インフォメーション
本記事は [ZOZO Advent Calendar 2023](https://qiita.com/advent-calendar/2023/zozo) の14日目の記事です
:::

私は普段 SRE として働いています。
主に AWS、Kubernetes、CI/CD あたりがメインでほぼほぼ yaml を書いています。（yaml 以外も書きたいなぁ...）
また、複数の AWS アカウント、Kubernetes Cluster、GitHub レポジトリ等を触っています。

開発環境に関しては少しこだわりがありいろんなツールを試したりしていて入れ替えが激しかったのですが、最近落ち着いてきたのでどのようなツールを使っていたり、どのような使い方をしているのか紹介したいと思います。

# 構成

WezTerm + tmux + Neovim の構成でコマンドを実行したり、コードや記事を書いたりしています。

基本的に WezTerm(ターミナル)で tmux を起動しコマンドを実行したり、Neovim(エディタ) でコード書いたりしています。
私のターミナルはこんな感じの色合いになっています。

![](/images/my-develop-environment/terminal.png)

VSCode のみや VSCode + ターミナル の構成を何度か試してみましたが、以下の点で諦めました。
  - Neovim も使っていて VSCode のキーバインディングがなかなか覚えられない（単純に歳かもしれない）
  - 複数の AWS アカウント、 Kubernetes Cluster、 GitHub レポジトリを扱う場合、1つのウィンドウ（≒ アプリケーション）に収まらない
  - 極力マウスは使いたくない

2つ目に関しては、VSCode 機能を知らないだけかもしれないので、1つのウィンドウでいい感じにできるのであればどなたか教えてください。

ターミナルもいろんなものがあると思いますが。1年前くらいの時点でドキュメントが豊富な WezTerm を選びました。lua でコンフィグを書ける点もポイントですね。
元々は Alacritty を使っていましたが、当時は日本語入力が微妙だったり、ドキュメントが少なかったりで苦労していたので乗り換えました。

# 紹介

## [Wezterm](https://wezfurlong.org/wezterm/)

Rust で書かれたターミナルです。
[Multiplexing] に対応しており、少し設定を行うだけで tmux や [screen] と似たような機能を使うことができます。以前 tmux からの乗り換えを検討したのですが、コピーモードの挙動が合わなかったので使用を見送りました。

ドキュメントはとても多く、基本的な情報は公式ドキュメントで足りると思います。

個人的にいいなと思っているポイントは、[カラーのビルドイン](https://wezfurlong.org/wezterm/colorschemes/index.html) がめちゃめちゃ多い（947 カラー）ことと設定ファイルを lua で書けること(Neovim の設定ファイルなどは lua で統一している)、設定ファイルのホットリロードが地味に便利です。
執筆している際に公式ドキュメントを少し眺めてみたのですが、機能がありすぎてまだまだ使いこなせていない感がすごいです。

さらっと見ただけでも [Shell Integration](https://wezfurlong.org/wezterm/shell-integration.html)、[Configuring Saved Searches](https://wezfurlong.org/wezterm/scrollback.html#configuring-saved-searches) などの今すぐにでも使えそうな機能があるので試してみたいと思います。

## [tmux](https://github.com/tmux/tmux)

ターミナル multiplexer として、tmux を使用しています。
コマンド実行ごとに AWS アカウントや EKS cluster を切り替えるのがめんどくさいので、基本的に環境(dev, stg, prd, etc.)や GitHub レポジトリごとにセッションを事前に作成して使っています。
ターミナルを立ち上げたタイミングで以下シェルスクリプトを実行し、セッションを作成し、main セッションにアタッチするようにしています。とても便利です。

```bash
#!/usr/bin/env bash

sessions=(
  main
  session1
  session2
  session3
  session4
  session5
  session6
  session7
)

for session in "${sessions[@]}"; do

  tmux new-session -d -s "${session}"

done

tmux a -t main
```

プラグインマネージャは、[tpm](https://github.com/tmux-plugins/tpm) を使用しています。
あまりカスタマイズはしておらず、カラースキーマを [nord](https://github.com/nordtheme/tmux)、キーバインディングは Neovim に寄せていたりするくらいです。
困っていたり使いづらいわけではないのですが、公式のドキュメント読んでいる感じ tmux もまだまだ使いこなせていない感がすごいです....

## [Neovim](https://github.com/neovim/neovim)

数年前までは Vim を使用していましたが、最近は Neovim を使っています。新しい物好きというだけで特に理由はありません。
最近までは自力で init.lua を書いていたのですが、設定ファイルに割く時間がなくなってきたのをきっかけに [LazyVim](https://www.lazyvim.org/) を導入しました。一瞬でインストールと設定が完了するので名前の通り lazy な方に最適です。元に戻すのもそんなに難しくないと思うので、使おうか悩んでいる方はぜひ試してみるといいかもしれません。

移行自体は、パッケージマネージャーに利用していた [lazy.nvim](https://www.lazyvim.org/configuration/lazy.nvim) が採用されていたり、普段使っているプラグインがデフォルトで入っていたこともあって簡単に移行できました。キーバインディングやデフォルトでインストールされないプラグインを使用している場合は、追加で設定やインストールが必要です。[General Setting](https://www.lazyvim.org/configuration/general) を参考に設定可能です。

デフォルトのまま使ってもいいのですが、インストールされるプラグインが意外と多いので不要なプラグインは、[disable](https://www.lazyvim.org/configuration/plugins#-disabling-plugins) にしています。

## Appearance

### Prompt

プロンプトのカスタマイズは、[Powerlevel10k(p10k)](https://github.com/romkatv/powerlevel10k) を使っています。
[Starship](https://starship.rs/ja-JP/) を使われている方はよく見かけますが、p10k を使われている方を見かけたことがないので、友達募集中です。
以前は環境変数を自力で頑張ったり、Starship を利用したりしていましたが、環境変数は大変なのと Starship は結構重く反応が悪かったので、p10k を使うようになりました。
今は問題なく快適に使えています。

### Font

フォントは、[Udev Gothic](https://github.com/yuru7/udev-gothic)を使っています。
JetBrains Mono や Ubuntu Mono 系が好みで、たまに [iosevka](https://github.com/be5invis/Iosevka) を使ったりします。
フォント探しに数時間使ったりするくらいおすすめなので気になる方はぜひ使ってみてください。

### Color Schema

カラースキーマは、[Nord](https://www.nordtheme.com/) を使っていて、基本的に Nord があるアプリケーションは、Nord を使用しています。
Gruvbox や [iceberg](https://github.com/cocopon/iceberg.vim) なども使ったりします。
フォント同様、カラースキーマ探しに数時間使ったりするくらいおすすめなので気になる方はぜひ使ってみてください。

カラースキーマに関しては、（不満が出て）自作する機会があれば作ってみようと思います。

## CLI

Homebrew と [auqa](https://github.com/aquaproj/aqua) を使っています。
以前は、Homebrew + asdf + aqua みたいな時期もありましたが、なんとか aqua に寄せつつあり移行できるものは、基本的に aqua を使用しています。
使っている理由としては、特定のバージョンを簡単にインストールできたり、設定ファイルで管理できるためです。

設定ファイルは GitHub レポジトリで管理していて、バージョンアップは [Renovate](https://github.com/renovatebot/renovate) で自動化するようにしています。新しいバージョンがリリースされたら Renovate bot が　PR を作成し、PR を自動マージする [automerge](https://docs.renovatebot.com/presets-default/#automergeall) を有効にしています。Major バージョンは自動マージせず、Minor、 patch バージョンのみ自動マージするという設定も可能です。

aqua も Renovate も他のツール同様で全く使いこなせている感じがしません...今後の自分に期待です。

最近、ネットサーフィンしていると数年前にすこし触ったけど諦めた [Nix](https://nixos.org/) の記事が結構あったり、[NixCon](https://2023.nixcon.org/) というなのカンファレンスが開催されるみたいでトレンドがきてそう？なので再挑戦してみたいと思います。

## 最後に

ローカルの作業環境について紹介しました。
気付きとして、基本的にツールを使いこなせていないことがわかり、まだまだ作業効率を挙げられる伸びしろがあるとポジティブに捉えています。

今回、あまり詳細なところまでは書けていないので、気になった部分があればコメントいただければと思います。
また、他にもこんな便利なツールがあるよ等も教えていただけるととても嬉しいです。

あ、ちなみに本記事は、VSCode で書きました。

[Multiplexing]: https://wezfurlong.org/wezterm/multiplexing.html
[screen]: https://en.wikipedia.org/wiki/GNU_Screen
