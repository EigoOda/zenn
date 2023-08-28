---
title: "GitHub workflow dispatchをデフォルトブランチ以外で実行する"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub"]
published: true
---

GitHub Actions（GHA）のworkflow dispatchは普段使っていますか。
私はあまり使っていません。
そのため、workflow dispatchを追加する際の動作確認に少しハマったので実施方法をまとめます。

## TL;DR

- workflow dispatchにon.pushもつけてcommit,push
- GHA workflowが実行され、CLIで実行できるようになるので、`gh workflow run`コマンドで実行
  - https://docs.github.com/ja/actions/using-workflows/manually-running-a-workflow?tool=cli
- on.pushは不要なので忘れないように削除

## workflow dispatchとは

workflowを手動でトリガーできるようにするためのものです。詳しくはGitHub Docsをご確認ください。
ref: https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch

## 動作確認方法

pull_requestと異なりPRを作成しただけでは追加したworkflow dispatchが動作しません。
また、[Web browser](https://docs.github.com/ja/actions/using-workflows/manually-running-a-workflow?tool=webui)からの起動はデフォルトブランチにworkflow fileが存在しないと実行できないため、GitHub CLIでworkflowを実行する必要があります。

https://github.com/EigoOda/zenn で試してみます。

まずは、dispatch workflowの作成です。以下の様なworkflow fileをcommitし、pushします。
add-workflow-dispatch というbranchで作業します。

```yaml
name: workflow dispatch
on:
  push:
    branches:
      - add-workflow-dispatch
  workflow_dispatch: {}

jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - name: hello
        run: "echo Hello"
```

GitHubのActionsに"workflow dispatch"を認識させる必要があるので、on.pushも設定し一度workflowを動作させます。
実際に実行されてしまうので、最初は実行しても問題ないJobを動かすのが良いです。[commit](https://github.com/EigoOda/zenn/pull/5/commits/f6d5a4c37d5894a0838306188d8b2e6a08c363ef)

on.pushを設定しない場合、workflowが実行されないため、Actionsには"workflow dispatch"が存在せず実行できません。
![](/images/gha-workflow-dispatch/actions-gui.png)

GitHub CLIも同様に対象のworklfowが存在しないと返ってきます。
```bash
$ gh workflow run "workflow-dispatch" --ref add-workflow-dispatch

could not find any workflows named workflow-dispatch
```

実際にworkflowを実行し、確認していきたいと思いますが、pushしたworkflowが動作すれば準備はOKです。
- https://github.com/EigoOda/zenn/actions/workflows/worklfow-dispatch.yaml

on.pushを削除し、GitHub CLIからworkflow dispatchを実行してみます。[commit](https://github.com/EigoOda/zenn/pull/5/commits/e76bf8de67e81bbb62608d60dffaae621d3cf874)

デフォルトブランチにマージしないとWeb browserから実行できないので、今回はCLIでworkflowを実行してみます。
```bash
$ gh workflow run "workflow dispatch" --ref add-workflow-dispatch
✓ Created workflow_dispatch event for worklfow-dispatch.yaml at add-workflow-dispatch

To see runs for this workflow, try: gh run list --workflow=worklfow-dispatch.yaml
```

CLIで実行できることが確認できたので、workflowの状態を確認します。

```bash
$ gh run list --workflow=worklfow-dispatch.yaml
STATUS  TITLE                  WORKFLOW           BRANCH                 EVENT              ID          ELAPSED  AGE
✓       workflow dispatch      workflow dispatch  add-workflow-dispatch  workflow_dispatch  5997350385  11s      2m
✓       Add worklfow dispatch  workflow dispatch  add-workflow-dispatch  push               5996858772  12s      55m
```

状態の確認については、Web browserからも確認可能です。
![](/images/gha-workflow-dispatch/actions-result.png)

いかがだったでしょうか、デフォルトブランチにpushしなくてもworkflow_dispatchを動作させることができました。
いままではdefault branchを変更してworkflow dispatchを実行してたりしましたが、安心して動作確認できるようになりました。
