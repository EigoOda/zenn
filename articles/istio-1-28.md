---
title: "Istio 1.28.0 がリリースされました🎉"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes","Istio","Service Mesh","Network"]
published: true
---

2025年11月、Istio 1.28がリリースされました。本リリースは、AI推論基盤対応とAmbient Meshの強化が大きなテーマです。
Kubernetes上でのAI推論やマルチクラスタ運用が当たり前になる中、Istioは「AI時代のサービスメッシュ」へ本格進化しています。

---

## 🎯 今回のハイライト

- InferencePool v1でAI推論の負荷分散やフェイルオーバーが強化
- Ambient MeshがマルチクラスタでL7ポリシーに対応
- nftablesとDual-stack (IPv4/IPv6) の対応強化
- セキュリティ機能アップデート
- `istioctl`改善による運用性向上
- 対応Kubernetes: 1.29〜1.34

---

## 🤖 AI推論対応: InferencePool v1

推論エンドポイント管理機能である**InferencePool v1**が導入され、AI Serving性能が向上しました。

### 改善点

- 推論Podの負荷考慮ルーティング
- 自動フェイルオーバー精度向上
- KServe / Ray Serve / Triton 等との統合強化

> AI inferenceネットワーク制御にIstioが深く関与する時代へ

---

## 🌐 Ambient Meshの進化

サイドカー不要のAmbient Meshが、**マルチクラスタ対応でL7ポリシー適用可能**に。

### ポイント

- リモートネットワークへのトラフィックルーティング
- アウトライア検知などL7機能の適用

---

## 🔥 nftables対応

Ambient モードでも nftables が利用可能に：

- iptables脱却のモダンLinux環境で有利
- セキュリティ & パフォーマンス強化

---

## 🌍 Dual-stack IPv4/IPv6 がBeta

Dual-stackがBeta昇格し、本番向け安定性が向上。
IPv6ネットワーク移行を考える環境で有効。

---

## 🔐 セキュリティ強化ポイント

- JWTの``spaceDelimitedClaims``対応
- mTLS ingress (FrontendTLSValidation / GEP-91)
- ``global.networkPolicy.enabled=true`` でNetworkPolicy展開
- 不正証明書はbundleから自動除外

Zero-trust強化が進むリリース。

---

## 🛠 運用改善: istioctl

- revision自動検出
- デバッグ改善

Canaryアップデートがより扱いやすくなりました。

---

## ✅ まとめ

| 観点 | 内容 |
|---|---|
AI | InferencePoolでAI mesh化が加速 |
Mesh | Ambientがマルチクラスタ対応へ前進 |
Net | nftables & IPv6対応強化 |
Ops | istioctl改善で運用性向上 |

**Istio = AI時代のサービスメッシュへ**
