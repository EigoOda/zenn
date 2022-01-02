---
title: "Kubernetesの非推奨(deplicated api)を自動検出する"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "pluto", "OSS"]
published: false
---

あけましておめでとうございます。
普段、インフラエンジニアをしていてKubernetesやCI/CD周りを主に触っています。

私の環境では、Kubernetesの環境はPRD/STG/DEVで3環境あり、Clusterのバージョンが異なっていることが多々あります。
また、最近はアプリケーションを`Helm`から`Argo CD`に移管したため
Deplicated apiの検出がChange logを見る以外では、なかなか把握しきれていなかったり、気づきにくい状態となっています。

※以前は、`helm install/upgrade`の際、deplicated apiがterminalに表示されていたため、気づけていましたが、Argo CDによるGitOpsに移行したため


そのため、CIに組み込み、自動で検知することにしました。
少しでも、Kubernetes deplicated apiを自動で検知する方法を探している方の参考になれば、幸いです。


## 実装方法

とっても簡単です。
CIで1つのjobを作成するのみ。
※ 現在、deplicated apiが検知されたときにSlackへ通知するなどは行っておりません。

