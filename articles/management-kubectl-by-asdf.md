---
title: "kubectlをサーババージョンに合わせて自動切り替え" 
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "asdf", "cli", "shellscript"]
published: true
---

# 概要

kubectl のバージョンをKubernetes clusterのバージョンに自動的に合わせる方法です。


# 背景

[公式ドキュメント](https://kubernetes.io/ja/docs/setup/release/version-skew-policy/#kubectl)では、以下記述のように kubectl のサポートバージョンが明記されており、
複数クラスタを以下ポリシーに沿って管理していない場合、クラスタごとに kubectl のバージョンを切り替えることが推奨？されています。

```
kube-apiserver の1つ以内のバージョン(古い、または新しいもの)をサポートします。

例)
- kube-apiserverが1.24であるとします
- kubectlは1.25、1.24および1.23がサポートされます
```


# 前提

- OSは、macOS を利用しています
- asdf で kubectl を管理していること
  - Homebrew などで kubectl をインストールしている場合は、asdf でインストールした kubectl を利用するように設定する必要があります。


# 実装内容

簡単に説明すると
クラスタを切り替えるタイミングでサーバとクライアントのバージョンを比較し、差異があった場合は、サーバのバージョンに合わせるようにしました。

Shell script で実装しており、以下コマンドが必要となっています。
- jq
- asdf
- gsed (Linux の場合は、sed でよいかも)

```bash
#!/usr/bin/env bash

[[ -n $DEBUG ]] && set -x

os=$(uname)
if [[ $os == "Darwin" ]]; then
  sed="gsed"
else
  sed="sed"
fi


need_cmd() {
  ! command -v $1 &> /dev/null \
    && echo "$1 could not be found." \
    && exit 123
}

add_plugin() {
  local installed_plugin_list=$(asdf plugin list | $sed -z -e 's@\n@ @g')

  if [[ "${installed_plugin_list}" =~ kubectl ]]; then
    :
  else
    echo -e "$plugin could not be found.\n$plugin will be install."
    asdf plugin add kubectl
    echo "INFO: kubectl was added."
  fi
}

install_plugin() {
  local installed_versions=$(asdf list kubectl)
  [[ -z "$installed_versions" ]] \
    && asdf install kubectl latest \
    && asdf global kubectl latest \
    && echo "INFO: $plugin latest version was installed."
}

main() {

  need_cmd asdf
  need_cmd jq

  add_plugin

  install_plugin

  need_cmd kubectl

  version_info=$(kubectl version -o json)
  client_version=$(echo "$version_info" | jq '.clientVersion.gitVersion' -r | $sed -e 's@v@@g')
  server_version=$(echo "$version_info" | jq '.serverVersion.gitVersion' -r \
    | $sed -e 's@v@@g' -e 's/-.*//g')

  if [[ "$client_version" != "$server_version" ]]; then
    installed_kubectl_versions=$(asdf list kubectl | $sed -z -e 's@\n@ @g')

    ! [[ "$installed_kubectl_versions" =~ "$server_version" ]] \
      && asdf install kubectl "$server_version"

    asdf global kubectl "$server_version"
    echo "INFO: Switch $plugin version $client_version to $server_version."
  fi
}

main "$@"
```

本記事とは、関係なく不要な行もあるので、簡単に実装内容を説明します。

main function に処理を記述しているため、そちらを見ていきます。

### add_plugin

ここではまず、asdf に kubectl plugin があるか確認し、ない場合は、追加しています。

```bash
add_plugin() {
  local installed_plugin_list=$(asdf plugin list | $sed -z -e 's@\n@ @g')

  if [[ "${installed_plugin_list}" =~ kubectl ]]; then
    :
  else
    echo -e "$plugin could not be found.\n$plugin will be install."
    asdf plugin add kubectl
    echo "INFO: kubectl was added."
  fi
}
```


### install_plugin

すでに kubectl plugin の何かしらのバージョンが入っていたほうが都合がいいので
どのバージョンもインストールされていない場合、最新のバージョンをインストールしています。

```bash
install_plugin() {
  local installed_versions=$(asdf list kubectl)
  [[ -z "$installed_versions" ]] \
    && asdf install kubectl latest \
    && asdf global kubectl latest \
    && echo "INFO: $plugin latest version was installed."
}
```


### main処理

サーバ(Kubernetes cluster)とクライアント(kubectl)のバージョンを取得しています。
※ 確か、先頭のvがついたままだとバージョン比較が正しく動作しなかったため、sed/gsed で切り取っています。

```bash
  version_info=$(kubectl version -o json)
  client_version=$(echo "$version_info" | jq '.clientVersion.gitVersion' -r | $sed -e 's@v@@g')
  server_version=$(echo "$version_info" | jq '.serverVersion.gitVersion' -r \
    | $sed -e 's@v@@g' -e 's/-.*//g')
```

main処理です。
サーバとクライアントのバージョンに差分があった場合、サーバと同じバージョンがインストールされているか確認し
なければ、インストールし、設定
あれば、バージョンを設定するだけの処理内容となっています。

```bash
  if [[ "$client_version" != "$server_version" ]]; then
    installed_kubectl_versions=$(asdf list kubectl | $sed -z -e 's@\n@ @g')

    ! [[ "$installed_kubectl_versions" =~ "$server_version" ]] \
      && asdf install kubectl "$server_version"

    asdf global kubectl "$server_version"
    echo "INFO: Switch $plugin version $client_version to $server_version."
  fi
```

# 実用例

実際は、以下のようにfunction 内で使用しています。

```bash
function ktx() {
    kubectl ctx $1
    [Shellscript name]
}
```

kubectl ctx(krew plugin) でクラスタ切り替え後、Shell script を実行


今の構成に不満はありませんが、もっと簡単にできるものがあれば、是非教えて下さい！！！
以上です。
