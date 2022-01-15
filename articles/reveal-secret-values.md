---
title: "KubernetesのSecretに格納された値を簡単に取り出す"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "Go"]
published: true
---

最近、Secretの作成や値の確認(decode)を繰り返していて、作成したSecretの値を確認するために
`kubectl get secret test-secret -ojsonpath='{.data.hogehoge} |base64 -d`をするのがめんどくさくなってしまったので、GoでCLIを作成してみました。

Goは初心者でこれが最初のGo CLIなので、全然完成度は高くないですが、
Goを勉強している方やSecretの値取得のために長いコマンド打つのがめんどくさくなってしまった方の助けになれれば嬉しいです。


## [Github(reveal-secret-values)](https://github.com/dubs11kt/reveal-secret-values)

### 使い方(README)

#### 何ができるか？

KubernetesのSecretにデプロイされているリソース１つのkey/value(decodeしたもの)を表示する

## どうやって使うか？

```
# コマンド確認用にSecret リソース作成
$ kubectl create secret generic test-secret -n default --from-literal=foo=bar --from-literal=boo=far --from-literal=moo=gar

# ビルド
$ git clone https://github.com/dubs11kt/reveal-secret-values
$ go build

# 実行
$ ./reveal-secret-values --namespace default --secret test-secret
boo : far
foo : bar
moo : gar

```

