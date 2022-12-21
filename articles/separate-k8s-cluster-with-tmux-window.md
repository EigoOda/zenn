---
title: "Kubernetes クラスタを安全に操作する"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "tmux", "aws"]
published: false
---

:::message
インフォメーション
本記事は [ZOZO Advent Calendar 2022](https://qiita.com/advent-calendar/2022/zozo) の23日目の記事です
:::

本記事では、安全にKubernetesクラスタを操作する方法について説明します。
ほかにも方法はあるので、その中の1つとして参考にしていただければと思います。

私は、普段の業務で複数のクラスタを1つのターミナルから操作しています。
同じ様にKubernetesクラスタを管理している多くの方が、複数のクラスタをターミナルから操作しているのではないかと思います。

そこで問題となるのが、Kubernetesクラスタの切り替えについてです。
kubectlはデフォルトで`$HOME/.kube/config`に設定された`current-context`を参照するため、単純にターミナルやタブを分け、ktxなどのツールで切り替えてもすべてのターミナルで同じクラスタを参照することになります。
そのため、操作するクラスタが変わるたびにコマンドを実行し切り替える必要があり大変ですし、切り替え忘れて`kubectl delete`をしてしまった場合はもっと大変なことになりそうな予感がします。
それらを防ぐために今回は、私がどのような方法をとっているか紹介したいと思います。

以下の方向けの記事です
- AWS EKSを使っている方
- 普段tmuxを使っていて、セッション、ウィンドウごとにKubernetesクラスタを固定したい
- プロンプトをカスタマイズしていて、いい感じに[kubie]が使えない


簡単にどのような実装となっているか説明し、その後具体的な設定内容を紹介します。

## 実装

tmuxのセッションごとにkubeconfigを作成し、ログインシェルの環境変数`KUBECONFIG`を設定します。
kubectlが参照するconfigを`$HOME/.kube/config`ではなく、作成したconfigを参照することでログインシェルごとに参照するクラスタを固定します。

## 設定内容

functionを`.zshrc`や`.bashrc`などに登録します。
function名やconfigのパスなどは、必要に応じて変更してください。

```bash
# ローカルに登録されているAWSプロファイルを表示
aws_profiles () {
    [[ -r "${AWS_CONFIG_FILE:-$HOME/.aws/config}" ]] || return 1
    grep --color=auto --exclude-dir={.bzr,CVS,.git,.hg,.svn,.idea,.tox} --color=never -Eo '\[.*\]' "${AWS_CONFIG_FILE:-$HOME/.aws/config}" | sed -E 's/^[[:space:]]*\[(profile)?[[:space:]]*([-_[:alnum:]\.@]+)\][[:space:]]*$/\2/g'
}

function up() {
  # /tmpにconfigを保管するディレクトリを作成
  ! test  -d "/tmp/kubeconfigs" && mkdir /tmp/kubeconfigs

  # デフォルトのconfigに戻れるオプションも用意
  if [[ $1 == "reset" ]]; then
    export KUBECONFIG="$HOME/.kube/config"
    return 0
  fi

  # AWSアカウントを環境変数に設定
  AWS_PROFILE=$(aws_profiles | fzf) \
    && export AWS_PROFILE \
    && export AWS_DEFAULT_PROFILE="${AWS_PROFILE}"

  # Get EKS Cluster cluser_name
  cluster_name=$(aws eks list-clusters | jq -r '.clusters[]' | fzf)

  # Create temporary kubeconfig file
  temp_config=$(mktemp -q /tmp/kubeconfigs/"$cluster_name"-XXXXXXX)
  aws eks update-kubeconfig --name "$cluster_name" --kubeconfig "$temp_config"

  # Set variable(KUBECONFIG) config
  export KUBECONFIG="$temp_config"

  # Set tmux session environment
  tmux setenv KUBECONFIG "$temp_config"
  tmux setenv AWS_PROFILE "$AWS_PROFILE"
  tmux setenv AWS_DEFAULT_PROFILE "$AWS_DEFAULT_PROFILE"
}
```


## Appnedix
- aws cli



[kubie]: https://github.com/sbstp/kubie
[p10k]: https://github.com/romkatv/powerlevel10k
[ktx]: https://github.com/vmware-archive/ktx
