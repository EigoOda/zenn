---
title: "Kubernetesクラスタを安全に操作する"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "tmux", "aws"]
published: true
---

:::message
インフォメーション
本記事は [ZOZO Advent Calendar 2022](https://qiita.com/advent-calendar/2022/zozo) の23日目の記事です
:::

本記事では、tmuxのセッションごとにKubernetesクラスタを固定することでAWS EKSを安全に操作する方法について説明します。
カスタマイズしていただければ、GKE、AKSなどでもご利用いただけるかと思いますし、ほかにも方法はあるのでその中の1つとして参考にしていただければと思います。



## なぜそうしたのか？

私は、普段の業務で複数のクラスタを1つのターミナルから操作しています。
Kubernetesクラスタを管理している多くの方が、同じような状況ではないでしょうか。

そこで問題となるのが、Kubernetesクラスタの切り替えです。
kubectlはデフォルトで`$HOME/.kube/config`に設定された`current-context`を参照します。
つまり、クラスタごとにターミナルやタブを分けてもすべてのターミナルが同じコンフィグを参照するため、ターミナルを分ける意味がなくなってしまいます。
また、操作するクラスタが変わるたびにコマンドを実行し切り替える必要があり大変ですし、切り替え忘れて`kubectl delete`をしてしまった場合はもっと大変なことになりそうな予感がします。

それらを防ぐために今回は、私がどのような方法をとっているか紹介したいと思います。

以下の方向けの記事です
- 普段tmuxを使っていて、セッション、ウィンドウごとにKubernetesクラスタを固定したい
- プロンプトをカスタマイズしていて、いい感じに[kubie]が使えない

ちなみにプロンプトやコンフィグをデフォルトで利用している方は、[kubie]がおすすめです。

簡単にどのような実装となっているか説明し、その後具体的な設定内容を紹介します。



## 実装の簡単な説明

AWS CLIのコンフィグからプロファイルをfzfで選択し、選択したプロファイルに存在するEKS一覧が表示されるため、その中から操作したいクラスタを選択します。

次にtmuxのセッションごとに選択したクラスタのkubeconfigを作成し、ログインシェルの環境変数`KUBECOFIG`を設定することで操作するクラスタを固定します。
また、tmuxのセッション環境変数を設定することによって、新しいウィンドウやペインを作成した際、引き継がれるようになっています。

接続先を切り替えるごとにkubeconfigを生成するため、/tmpに作成することで、PCリスタート時に削除します。

では、どのようになっているか見ていきたいと思います。



## 実装内容

以下functionを`.zshrc`や`.bashrc`などに登録し、リロードしてください。

:::message alert
Shell scriptではうまく動かないので注意です。
function名やコンフィグのパスなどは、必要に応じて変更してください。
:::

```bash
# ローカルに登録されているAWSプロファイルを表示
aws_profiles () {
    [[ -r "${AWS_CONFIG_FILE:-$HOME/.aws/config}" ]] || return 1
    grep --color=auto --exclude-dir={.bzr,CVS,.git,.hg,.svn,.idea,.tox} --color=never -Eo '\[.*\]' "${AWS_CONFIG_FILE:-$HOME/.aws/config}" | sed -E 's/^[[:space:]]*\[(profile)?[[:space:]]*([-_[:alnum:]\.@]+)\][[:space:]]*$/\2/g'
}

function up() {
  # /tmpに作成するコンフィグ用のディレクトリを作成
  # PCをリスタートした際、削除されるように/tmpに配置
  ! test  -d "/tmp/kubeconfigs" && mkdir /tmp/kubeconfigs

  # デフォルトのコンフィグ($HOME/.kube/config)に戻れるオプションも用意
  if [[ $1 == "reset" ]]; then
    export KUBECONFIG="$HOME/.kube/config"
    return 0
  fi

  # AWSプロファイルを環境変数に設定
  AWS_PROFILE=$(aws_profiles | fzf) \
    && export AWS_PROFILE \
    && export AWS_DEFAULT_PROFILE="${AWS_PROFILE}"

  # AWSプロファイル内に存在するEKSクラスタを取得
  cluster_name=$(aws eks list-clusters | jq -r '.clusters[]' | fzf)

  # 取得したEKSクラスタのkubeconfigを作成
  export KUBECONFIG="$(mktemp -q /tmp/kubeconfigs/"$cluster_name"-XXXXXXX)"
  aws eks update-kubeconfig --name "$cluster_name" --kubeconfig "$KUBECONFIG"

  # tmuxのセッション環境変数に追加
  # 新しいウィンドウ、ペインを作成しても引き継がれます
  tmux setenv KUBECONFIG "$KUBECONFIG"
  tmux setenv AWS_PROFILE "$AWS_PROFILE"
  tmux setenv AWS_DEFAULT_PROFILE "$AWS_DEFAULT_PROFILE"
}
```

## 最後に

いかがだったでしょうか？
ニッチな情報となりましたが、少しでもローカル環境構築の役に立てれればと思います。

ほかにもこんなツールがあるよ など教えていただけると大変嬉しいです。

[kubie]: https://github.com/sbstp/kubie
[p10k]: https://github.com/romkatv/powerlevel10k
[ktx]: https://github.com/vmware-archive/ktx
