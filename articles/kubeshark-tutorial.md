---
title: "Kubernetes用に再発明されたAPIトラフィックビューアであるKubesharkを試してみる"
emoji: "🥶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "Network"]
published: false
---

:::note info
インフォメーション
本記事は ZOZO Advent Calendar 2022 の7日目の記事です
:::

本記事では、Kubesharkでどのようなことができるか試してみたいと思います。

環境
| 環境 | バージョン |
| --  | --        |
| kind |      |
| PC   | macOS |

## Kubeshark について

KubernetesのAPI trafficを可視化するツールで、Kubernetes内の全てのAPI trafficの可視化と監視する機能が提供されています。
Chrome Dev Tools、TCPDump、Wiresharkを組み合わせて、Kubernetes用に作り直したものとイメージしていただけるとわかりやすいかもしれません。


## 準備

### kubesharkのインストール

kubesharkのインストールはGitHubの[Download](https://github.com/kubeshark/kubeshark#download)にかかれている通り、shellscriptを実行します。

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

実行し、少し待つと以下のようにWEBブラウザにkubeconfigのcurrent-contextに設定されたnamespaceのlive trafficが表示されます。
![kubeshark-image](../images/kubeshark-tutorial/top-image.png)

現在は、Nginx deploymentの[liveness/readiness](https://github.com/dubs11kt/kubernetes-manifests/blob/zenn/kubeshark-tutorial/helm/nginx/templates/deployment.yaml#L38-L45)の通信が発生しているのがわかります。
