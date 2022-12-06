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

本記事では、[Kubeshark]という、KubernetesのAPI trafficを可視化するツールでどのようなことができるか試してみたいと思います。

### 環境

今回利用するソフトウェアは以下の通りです

| ソフトウェア | バージョン |
| --  | --        |
| Kubeshark | 37.0 |
| kind | kind v0.17.0 go1.19.2 darwin/arm64 |
| client PC   | Darwin Kernel Version 21.6.0(M1 macOS) |
| Helm | v3.10.2|

Helm chartは以下を使います
- [Nginx](https://github.com/dubs11kt/kubernetes-manifests/tree/zenn/kubeshark-tutorial/helm/nginx)


## Kubeshark について

Kubesharkは、KubernetesのAPI trafficを可視化するツールで、Kubernetes内の全てのAPI trafficの可視化と監視する機能が提供されています。
Chrome Dev Tools、TCPDump、Wiresharkを組み合わせて、Kubernetes用に作り直したものとイメージしていただけるとわかりやすいかもしれません。

また、API traffic viewerだけでなく、Service Catalog、Service Map、Traffic Statsなどの機能も備わっています。

詳細な部分に関しては、Kubesharkの[Docs]をご確認ください。

早速、どのような事ができるか試したいと思います。



## 準備

### Kubernetes Clusterの構築、動作確認用のアプリケーションをデプロイ

kindでKubernetes Clusterを構築した後、動作確認用のNginxをデプロイします。

```bash
$ kind create cluster

$ git clone https://github.com/dubs11kt/kubernetes-manifests.git
$ cd kubernetes-manifests && helm install nginx helm/nginx
```

### kubesharkのインストール

kubesharkのインストールはGitHubの[Download](https://github.com/kubeshark/kubeshark#download)にかかれている通り、shellscriptを実行します。

```bash
sh <(curl -Ls https://kubeshark.co/install)

$ kubeshark version
Version: 37.0 (main)
```



## 実行

### Live traffic streaming

さっそくコマンドを実行し、kubesharkを立ち上げ、先程デプロイしたNginx Podのトラフィックを見てみます。

```bash
$ kubeshark tap
Kubeshark will store up to 200MB of traffic, old traffic will be cleared once the limit is reached.
Tapping pods in namespaces "default"
+nginx-b546f689d-tjbxf
Waiting for Kubeshark Agent to start...
Kubeshark is available at http://localhost:8899
```

何もオプションを渡さない場合は、kubeconfigのcurrent-contextに設定されたnamespaceにデプロイされているpodのlive trafficが表示されます。
`kubeshark tap -A`のように`-A`オプションを渡すと`kubectl`と同じ様に全namespaceが対象となります。

少し待つと以下のような画面がWEBブラウザに表示されます。
![](/images/kubeshark-tutorial/top-image.png)

画面を見るとNginx deploymentの[liveness/readiness][nginx-health-check]のトラフィックが発生しているのがわかります。
画面左側のトラフィックが流れているところにマウスのカーソルを合わせスクロールするか、添付イメージのようにボタンをクリックすることでlive streaamingを一時的に止め、流れてきた特定のトラフィックを確認することができます。

**live streamingを停止する**
![](/images/kubeshark-tutorial/stream-paused.png)

**live streamingを再開する**
![](/images/kubeshark-tutorial/stream-live.png)


次にほかのPodからNginx Podへ疎通し、結果を確認してみます。
curlが実行できるPodをデプロし、そのPodのIPと一番最初にデプロイしたNginx SerivceのIPを取得します。

```bash
$ k run test-pod --image ghcr.io/dubs11kt/dubs11kt/debug-container:latest -it --rm -- bash

# test-podのIPを取得
$ k get pod -owide | grep test-pod
test-pod                 1/1     Running   0          3h     10.244.1.7    kind-worker   <none>           <none>

# nginx serivceのIPを取得
$ k get svc | grep nginx
nginx        ClusterIP   10.96.96.121   <none>        80/TCP    2d16h
```

このままWEBブラウザで確認してもいいのですが、liveness/readinessで発生するトラフィックとtest-podからのトラフィックが混じってしまって確認しにくいため、test-podからテストする前にKubeshark側で少し設定をします。

今回は、test-podからのリクエストのみに絞りたいため、先程取得したIPを`syntax field`に入力し、test-podからのトラフィックに絞ります。
![](/images/kubeshark-tutorial/syntax-field.png)

リクエストを受け付けるNginx podのIPも絞ります。
対象の項目にカーソルを合わせて`+`をクリックし、`syntax field`に追加することも可能です。
![](/images/kubeshark-tutorial/syntax-field-click.png)

ではさっそく、以下3つのcurlコマンドをtest-podから実行し、kubesharkでどの様に表示されるか見てみます。

```bash
No.1: curl nginx.default/index.html # liveness
No.2: curl nginx.default/50x.html # readiness
No.3: curl nginx.default/error.html # There is no file(error.html).
```

想定通り、200と404が返ってきました。
![](/images/kubeshark-tutorial/curl-results.png)

対象のリクエストをクリックすることで返ってきたリクエストの中身も見ることができます。以下のイメージは、No.3のリクエスト結果です。
![](/images/kubeshark-tutorial/error-html.png)

kubeshark コマンドを実行するだけでいい感じに可視化ができることがわかりました。

次にService Catalog、Service Map、Traffic Statsを見てみたいと思います。



### Service Catalog

対象のサービスにどのようなエントリーが存在するか確認することができます。
※ 画面全体を載せたかったため、文字等が小さくなってしまっています。

![](/images/kubeshark-tutorial/service-catalog.png)

全体のリクエスト数や成功/失敗のリクエスト数、平均のレスポンスタイムなどが記載されており、負荷検証時に使うとかなり便利そうだと感じました。



### Service Map

サービス間の繋がりが可視化されているMapが表示されます。
ほかのSaaSやOSSなどを使って可視化する場合、エージェントや関連コンポーネントなどなにかしらの導入をしなければなりませんが、Kubesharkの場合は自分たちでなにかをする必要がないのでとても嬉しいです。

![](/images/kubeshark-tutorial/service-map.png)



### Traffic Stats

時間ごとのトラフィック量やprotocolごとの割合を見ることができます。
少し見づらくなっていますが、12時前後のリクエスト量がほかの時間帯のリクエスト量と比べて多くなっていることがわかります。

![](/images/kubeshark-tutorial/traffic-stats.png)



## 感想

Kubesharkを一通り使ってみた感想です。
- **負荷試験でめちゃくちゃ使えそう**
- 簡単にKubernetesのネットワークトラッフィクを可視化できる状態を準備できる
  - SaaS APMのように長期間ネットワークトラフィックを可視化するツールではないので注意
- コストの問題でテスト環境や検証環境にAPMを導入していない環境で動作確認する際に役に立ちそう
- どこまでtraceしたいかにもよるが、簡単な負荷試験やテスト、検証などはKubesharkだけで完結しそう


[Kubeshark]: https://kubeshark.co/
[nginx-health-check]: https://github.com/dubs11kt/kubernetes-manifests/blob/zenn/kubeshark-tutorial/helm/nginx/templates/deployment.yaml#L38-L52
[Docs]: https://docs.kubeshark.co/en/introduction
