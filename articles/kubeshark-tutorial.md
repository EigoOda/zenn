---
title: "KubernetesのAPI traffic viewerであるKubesharkを試してみる"
emoji: "🥶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "Network"]
published: false
---

:::message
インフォメーション
本記事は ZOZO Advent Calendar 2022 の7日目の記事です
:::

本記事では、Kubesharkでどのようなことができるか試してみたいと思います。

### 環境

今回利用するソフトウェアは以下の通りです

| ソフトウェア | バージョン |
| --  | --        |
| kind | kind v0.17.0 go1.19.2 darwin/arm64 |
| client PC   | Darwin Kernel Version 21.6.0(M1 macOS) |
| Helm | v3.10.2|

Helm chartは以下を使います
- [Nginx](https://github.com/dubs11kt/kubernetes-manifests/tree/zenn/kubeshark-tutorial/helm/nginx)


## Kubeshark について

[Kubeshark](https://github.com/kubeshark/kubeshark)は、KubernetesのAPI trafficを可視化するツールで、Kubernetes内の全てのAPI trafficの可視化と監視する機能が提供されています。
Chrome Dev Tools、TCPDump、Wiresharkを組み合わせて、Kubernetes用に作り直したものとイメージしていただけるとわかりやすいかもしれません。

<!-- kubesharkについてすこし説明を入れる -->


## 早速

### kubesharkのインストール

kubesharkのインストールはGitHubの[Download](https://github.com/kubeshark/kubeshark#download)にかかれている通り、shellscriptを実行するのみです。

```bash
sh <(curl -Ls https://kubeshark.co/install)
```

### 実行

以下コマンドを実行し、kubesharkを立ち上げます。

```bash
$ kubeshark tap
Kubeshark will store up to 200MB of traffic, old traffic will be cleared once the limit is reached.
Tapping pods in namespaces "default"
+nginx-b546f689d-tjbxf
Waiting for Kubeshark Agent to start...
Kubeshark is available at http://localhost:8899
```

何もオプションを渡さない場合は、kubeconfigのcurrent-contextに設定されたnamespaceにデプロイされているpodのlive trafficが表示されます。

少し待つと以下のような画面がWEBブラウザに表示されます。
![](/images/kubeshark-tutorial/top-image.png)

現在は、Nginx deploymentの[liveness/readiness](https://github.com/dubs11kt/kubernetes-manifests/blob/zenn/kubeshark-tutorial/helm/nginx/templates/deployment.yaml#L38-L45)の通信が発生しているのがわかります。

