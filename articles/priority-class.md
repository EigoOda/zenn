---
title: "PriorityClassを設定しする必要があるのか？"
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

このようなことからすべてのケースに言えないかもしれませんが、私はシステムを正常に稼働させるためには、設定しておいたほうがいいと考えています。

