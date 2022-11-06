---
title: "いまさらながらEphemeral Containerを触ってみた"
emoji: "🥶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "ephemeral-container", "debug"]
published: false
---

Kubernetes 1.25でstableに昇格したEphemeral Containerを触ってみました。
Kubernetes Clusterはkindで構築したものを利用します。


# [Ephemeral Container](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)とは？

Podをトラブルシューティングするための仕組みです。
Ephemeral Containerを利用することで、`kubectl exec`できないPod(Container)を比較的用意にデバッグすることができます。

実際にやってみたいとおもいます。


## 試してみる

[Debugging with an ephemeral debug container](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/#ephemeral-container)を実行します。

Ephemeral Container を追加するPodを用意

```bash
$ kubectl run ephemeral-demo --image=registry.k8s.io/pause:3.1 --restart=Never
pod/ephemeral-demo created
```

container imageから想像するにpauseするだけのPodを立ち上げているように見えます。

```bash
$ k get pod
NAME             READY   STATUS    RESTARTS   AGE
ephemeral-demo   1/1     Running   0          8m17s

$ k describe pod ephemeral-demo
Name:             ephemeral-demo
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-worker/172.18.0.2
Start Time:       Sun, 06 Nov 2022 10:28:09 +0900
Labels:           run=ephemeral-demo
...
Containers:
  ephemeral-demo:
    Container ID:   containerd://f5846833554a36b86323cea995e59baaf854c4cb34e6110206388f64a8751ac8
    Image:          registry.k8s.io/pause:3.1
    Image ID:       registry.k8s.io/pause@sha256:f78411e19d84a252e53bff71a4407a5686c46983a2c2eeed83929b888179acea
...
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  8m28s  default-scheduler  Successfully assigned default/ephemeral-demo to kind-worker
  Normal  Pulling    8m28s  kubelet            Pulling image "registry.k8s.io/pause:3.1"
  Normal  Pulled     8m25s  kubelet            Successfully pulled image "registry.k8s.io/pause:3.1" in 2.974039626s
  Normal  Created    8m25s  kubelet            Created container ephemeral-demo
  Normal  Started    8m25s  kubelet            Started container ephemeral-demo
```

用意したPodに`kubectl exec`してみる。

```bash
$ kubectl exec -it ephemeral-demo -- sh
error: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "50b20e6b4758442b0cbbee696ac98080767a431d4ea502d9bf34f624736f20ad": OCI runtime exec failed: exec failed: unable to start container process: exec: "sh": executable file not found in $PATH: unknown
```

shが存在しないため、execができませんでした。
ここでEphemeral Containerを使ってサクッと`ephemeral-demo`にアクセスしたいと思います。

```bash
$ kubectl debug -it ephemeral-demo --image=busybox:1.28 --target=ephemeral-demo
Targeting container "ephemeral-demo". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-j6lb2.
If you don't see a command prompt, try pressing enter.

/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 /pause
   30 root      0:00 sh
   39 root      0:00 ps
```

上記コマンドでbusyboxのContainerを追加し、アタッチしました。
targetパラメータは、他のコンテナのプロセス名前空間を対象とします。shareProcessNamespaceを有効にしていない場合、必要となります。
そうすることで、`ephemeral-demo`で実行されているプロセスをpsで見ることができました。

targetパラメータを利用しない場合、`ephemeral-demo`のプロセスを見ることができませんでした。

```bash
$ kubectl debug -it ephemeral-demo --image=busybox:1.28
Defaulting debug container name to debugger-q57tq.
If you don't see a command prompt, try pressing enter.
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 sh
   10 root      0:00 ps
```

Ephemeral Containerを使って、既存Podのトラブルシューティングを行う方法を試してみました。
普段は、shやbashがインストールされているContainer Imageを使うことが多いため、困ったことはありませんが、distrolessなどの必要な実行ファイルのみ含んだContainer Imageを利用している場合は、役立ちそうだなーと感じました。

