---
title: "Nodelocal DNS Cache の導入と成果"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "DNS", "CoreDNS"]
published: false
---

# 概要

本記事では、[NodeLocal DNS Cache (NDC)](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/nodelocaldns) を導入した際の目的や成果を整理します。
システムの規模感や特徴によって原因や対処方法が異なりますので、その点ご了承ください。

# 導入の目的

NDC 導入前は、CoreDNS Pod のスケール時に **DNS Lookup Failed** や **i/o timeout**が発生していました。
一時的な対応として `ndot = 1` を設定し改善は見られましたが、完全に解消できませんでした。

そのため、**NDC を導入することで DNS Lookup Failed をほぼゼロにすること**を目的としました。
また、その他のメリットとして、以下のことが考えられます。
- CoreDNS へのリクエストを削減
- CoreDNS 依存度が低下
- CoreDNS 障害時にも各サービスへの影響を最小化可能

# Nodelocal DNS Cache とは

クラスターノード上で DaemonSet として DNS キャッシュエージェントを実行することにより、DNSパフォーマンスの向上を目的として作成された OSS。
パフォーマンス向上やキャッシュ統計、その他のメトリクスを可視化する機能の追加を実現できる。

## アーキテクチャの変化

NDC を導入した際のアーキテクチャ

![ndc-arch](/images/nodelocal-dns-cache/ndc-arch.png)

引用元: https://kubernetes.io/ja/docs/tasks/administer-cluster/nodelocaldns/


## CoreDNS 障害時における NDC のカバー範囲

CoreDNS の障害時、NDC でカバーできる範囲はありますが、すべてをカバーできません。
考えられることに関してカバーできるできないを洗い出しました。

### NDC でカバーできる不具合

- CoreDNS の QPS 制限やスケール問題
- CoreDNS Pod の一時的ダウン（RollingUpdate / OOMKill）

### NDC でカバーできない不具合

- ConfigMap の設定ミスやプラグインの誤設定
- 外部 DNS サーバとの接続障害
- TTL による古いキャッシュ保持
- CoreDNS 内部バグ（クラッシュ・メモリリークなど）

## アップグレード手法

NDC にはアップグレード時の課題がありました。

バージョンアップなどの際に発生する `Rollout restart` による Pod 再作成時、**DNS timeout**が発生。
DaemonSet の `GracefulTerminationSeconds` などの調整や[NodeLocal DNSCache の設定](https://cloud.google.com/kubernetes-engine/docs/how-to/nodelocal-dns-cache?hl=ja)を参考に設定しても改善しませんでした。
ただし、`kubectl drain` による Node 終了ではエラーが発生しないことを確認できました。

そのため、RollingUpdate より時間はかかりますが、安全性を優先し、以下の戦略を採用しました。

- UpdateStrategy を `OnDelete` に設定
- Karpenter の drift 機能で Node を入れ替え

詳細な内容についは、TBU で紹介してますので、よろしければ見てみてください。

# 成果

8月上旬に NDC の導入が完了

## DNS Lookup Failed の解消

現在、DNS 関連のエラーをメトリクスで出力できているマイクロサービスは多くはないですが、NDC の導入前後でエラーがなくなりました。

![deploy-ndc](/images/nodelocal-dns-cache/deploy-ndc.png)

## i/o timeout の減少

また別のサービスでは、i/o timeout 数を以下のように削減できました。

- エンドポイントA
  - 7月：176件 → 8月：95件
  - **46% 削減に成功**
- エンドポイントB
  - 7月：159件 → 8月：78件
  - **51% 削減に成功**

# Appendix

本番導入前の検証でレイテンシ改善を確認できたのですが、本番環境では大きな改善は見られませんでした。

## まとめ

- NDC 導入により DNS Lookup Failed や i/o timeout を大幅に削減
- システムの安定性向上に貢献
- CoreDNS 障害の一部をカバー可能だが、根本的な問題は引き続き CoreDNS 側の対策が必要
