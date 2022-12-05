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

本記事では、[Kubeshark](https://github.com/kubeshark/kubeshark)という、KubernetesのAPI trafficを可視化するツールでどのようなことができるか試してみたいと思います。

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



## 準備

### Kubernetes Cluster, Applicationの準備

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

さっそくコマンドを実行し、kubesharkを立ち上げ、先程デプロイしたNginx Podの通信を見てみます。

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

画面を見るとNginx deploymentの[liveness/readiness][nginx-health-check]の通信が発生しているのがわかります。

画面左側のトラッフィクが流れているところにマウスのカーソルを合わせスクロールするか、添付イメージのようにボタンをクリックすることでliveを一時的に止めることができます。

live streamingを停止する

![](/images/kubeshark-tutorial/stream-paused.png)

live streamingを再開する

![](/images/kubeshark-tutorial/stream-live.png)


次に別のPodからNginx Podへの通信を確認してみます。
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

test-podからリクエストを送るテストをする前にKubeshark側で少し設定をします。

対象Podからのリクエストのみに絞るため、取得したIPを`syntax field`に入力します。そうすることで特定のSource IPからの通信に絞ることができます。
![](/images/kubeshark-tutorial/syntax-field.png)

Destination IPも別の方法で絞ってみます。
対象の項目にカーソルを合わせて`+`をクリックしても`syntax field`に追加することが可能です。
![](/images/kubeshark-tutorial/syntax-field-click.png)

ではさっそく、以下3つのcurlコマンドをtest-podから実行し、kubesharkでどの様に表示されるか見てみます。

```bash
1: curl nginx.default/index.html # liveness
2: curl nginx.default/50x.html # readiness
3: curl nginx.default/error.html # There is no file(error.html).
```

想定通り、200と404が返ってきました。
![](/images/kubeshark-tutorial/curl-results.png)

対象のリクエストをクリックすることで返ってきたリクエストの中身も見ることができます。これは、コマンド3のリクエスト結果です。
![](/images/kubeshark-tutorial/error-html.png)

kubeshark コマンドを実行するだけでいい感じに通信の可視化ができることがわかりました。

次に`Service Catalog`、`Service Map`、`Traffic Stats`を見てみたいと思います。


### Service Catalog

対象のサービスにどのようなエントリーが存在するか確認することができます。
※ 画面全体を載せたかったため、文字等が小さくなってしまっています。

![](/images/kubeshark-tutorial/service-catalog.png)

全体のリクエスト数や成功/失敗のリクエスト数、平均のレスポンスタイムなどが記載されており、負荷検証時に使うとかなり便利そうだと感じました。



### Service Map

サービス間の繋がりが可視化されているMapが表示されます。
ほかのSaaSやOSSなどを使って可視化する場合、エージェントやsidecarを導入しなければなりませんが、Kubesharkの場合は自分たちでなにかをする必要がないのでとても嬉しいです。

![](/images/kubeshark-tutorial/service-map.png)

最初開いたときにずっとくるくる回って見づらかったですが、左上の「Refresh」をクリックすると止まりました。



### Traffic Stats

時間ごとのtraffic量やprotocolごとの割合を見ることができます。

![](/images/kubeshark-tutorial)


## 感想

Kubesharkを一通り使ってみた感想です。
- **負荷試験でめちゃくちゃ使えそう**
- OSSでKubernetesのネットワークトラッフィクが可視化され、わかり易い/見やすい
  - SaaS APMのように長期間ネットワークトラフィックを可視化するツールではないので注意
- コストの問題でテスト環境や検証環境にAPMを導入していない環境で動作確認する際に役に立ちそう
- どこまでtraceしたいかにもよるが、簡単な負荷試験やテスト、検証などはKubesharkだけで完結しそう


[Kubeshark]: https://kubeshark.co/
[nginx-health-check]: https://github.com/dubs11kt/kubernetes-manifests/blob/zenn/kubeshark-tutorial/helm/nginx/templates/deployment.yaml#L38-L52

