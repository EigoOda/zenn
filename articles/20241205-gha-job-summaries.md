---
title: "GitHub Actions の Job Summaries を使って CI の結果を簡単に確認したい"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Github", "CI"]
published: false
---

# はじめに

GitHub Actions の Job Summaries を知ってますか？
約2年半前の2022年5月9日に発表され、各ジョブで生成された結果をMarkdownで表示できるというものです。
- https://github.blog/news-insights/product-news/supercharging-github-actions-with-job-summaries/

例えばテスト結果の集計と表示やレポートの作成、ログに依存しないカスタム出力などができるようになります。
`terraform plan`の結果やCIの実行結果を表にまとめて表示させるなどの例が理解しやすいのではないでしょうか。

CI の結果を簡単に確認する別のアプローチとして PR のコメントに表示させる方法もあります。
ただ、PR のコメントの文字制限は 64 KB、Job Summaries の文字制限は 128 KB となっており、Job Summaries のほうに分があると言えます。

この記事では、GitHub Actions の Job Summaries を使って CI 結果を簡単に確認する方法を紹介します。

## Job Summaries の使い方

Markdown コンテンツを`$GITHUB_STEP_SUMMARY`という環境変数に出力するだけで GitHub Actions の Job Summaries に表示されます。
簡単ですよね？

Job Summaries の例を何パターンか例を挙げてみます。


### 1. echo コマンドの出力を表示

Job Summaries に `Hello world! 🚀` と表示します。

```yaml
...
jobs:
  changes:
    runs-on: ubuntu-latest
    steps:
      - name: Adding markdown
        run: echo '### Hello world! 🚀' >> $GITHUB_STEP_SUMMARY
```

GitHub Actions サマリーに以下のように表示されます。
![](/images/gha-job-summaries/hello-world.png)


### 2. kustomize build の結果を表示

kustomization.yaml が k8s/{sample, alpha} ディレクトリにあるとして、それぞれのディレクトリの kustomize build の結果を Job Summaries に表示します。

```yaml
jobs:
  k8s:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        dir: [sample, alpha]
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install kustomize command
        uses: imranismail/setup-kustomize@v2
        with:
          kustomize-version: "5.3.0"

      - name: kubectl diff
        run: |
          kustomize build "k8s/${{ matrix.dir }}" | tee "${{ matrix.dir }}.txt"
          {
            echo "\`\`\`diff"
            cat "${{ matrix.dir }}.txt"
            echo "\`\`\`"
          } >> "$GITHUB_STEP_SUMMARY"
```

GitHub Actions サマリーに以下のように表示されます。
![](/images/gha-job-summaries/kustomize-diff.png)

\`\`\` にdiff を指定することで、kubectl diff などが視覚的に確認しやすくなります。
![](/images/gha-job-summaries/kustomize-diff-ci.png)


## Markdown 以外で Job Summaries に表示する方法

複雑なことをしたくなった場合に、Markdown のみだとで対応できないケースがあると思います。
そのような場合は、https://github.com/actions/github-script を使うことで JavaScript を実行できます。
`core.summary`使うことで渡した内容を Job Summaries に出力することができます。

```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    steps:
      - name: Job summaries
        uses: actions/github-script@v7
          with:
            script: |
              await core.summary
                .addHeading("Hello world! 🚀")
              .write()
```

GitHub Actions サマリーに以下のように表示されます。
![](/images/gha-job-summaries/hello-world-js.png)
