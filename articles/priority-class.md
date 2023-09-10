---
title: "PriorityClassを設定する必要があるのか？"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes"]
published: false
---

皆さんはPriorityClassを設定していますか？

PriorityClassは、Podのpriorityを設定するものです。
Podがスケジュールできない際に優先度が低いPodが追い出され、優先度が高いPodがスケジュールされます。

PriorityClassを設定していない場合や設計を間違えている場合、以下のことが考えられます。
- メトリクスを取得しているPodの代わりにアプリケーションPodが追い出される
- Nodeのリソース不足によりアプリケーションPodがスケジュールされない
  - Cluster AutoScalerの場合約2分、Karpenterの場合約1分ほどNodeがReadyになるまでに時間がかかるため、その間スケジュールされない

このようなことからすべてのケースに言えないかもしれませんが、私はシステムを正常に稼働させるために設定しておいたほうがいいと考えています。

## PriorityClass とは

cluster scopedのオブジェクトでオブジェクト名と優先度を表す整数値やその他オプションなどを定義します。
優先度は10億以下の任意の32ビットの整数値を設定することができ、10億より大きい値は重要なシステム用Podのために予約されています。
例えば、system-cluster-critical, system-node-critical です。以下のように確認することができます。

```bash
$ k get priorityclass
NAME                      VALUE        GLOBAL-DEFAULT   AGE
system-cluster-critical   2000000000   false            17d
system-node-critical      2000001000   false            17d
```

また、任意で`globalDefault`、`description`、`preemptionPolicy`を設定可能です。

### globalDefault

priorityClassNameが指定されていないPodに設定されるPriorityClassであることを意味します。
デフォルトは`true`です。

### description

どのようなPodに対して設定されるべきのものであるかなどの説明を記述することができます。

### preemptionPolicy

PriorityClassが設定されたPodがプリエンプションを行うかどうかのポリシーで`Never`と`PreemptLowerPrioirty`が設定可能です。
デフォルトは`PreemptLowerPrioirty`です。

`Never`はスケジューリングのキューにおいて他の優先度の低いPodよりも優先されますが、スケジューリングされているPodをプリエンプトすることはありません。
`PreemptLowerPrioirty`は名前の通り、優先度の低いPodをプリエンプトするための設定です。








[PriorityClass]: https://kubernetes.io/ja/docs/concepts/scheduling-eviction/pod-priority-preemption/#priorityclass
[Podの優先度とプリエンプション]: https://kubernetes.io/ja/docs/concepts/scheduling-eviction/pod-priority-preemption/
