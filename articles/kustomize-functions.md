---
title: "Kustomize の良さそうな機能"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Terminal", "Wezterm", "tmux", "Neovim"]
published: false
---

皆さんは Kubernetes manifests のパッケージ化やテンプレート化に何を使っていますか？
Helm、Kustomize、CUE などあると思いますが、私は普段 Kustomize を使っています。
ここ最近 Kustomize の改善を行ったりした際に少し調べたので、便利そうだった機能や普段から利用している機能を紹介しようと思います。

## [resources](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/resource/)

自分たちで作成し、管理している manifest は以下のように kustomization.yaml を書くことが多いと思います。

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - secrets.yaml
  - service-account.yaml
```

resources に kustomization.yaml が置かれているディレクトリや GitHub リリースの manifest を指定することで簡単にリモートレポジトリの manifest を利用することができます。
わざわざローカルレポジトリに manifest を持ってこなくても良かったり、バージョンアップはバージョンを変更するだけなので変更はとても容易です。

kustomization.yaml が置かれているディレクトリ を指定する例

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - https://github.com/kubernetes/kube-state-metrics?ref=v2.6.0
```

GitHub リリースの manifest を指定する例

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.2/components.yaml
```

kustomization.yaml がリモートレポジトリに置かれている場合は前者、Helm Chart から生成された manifest が GitHub リリースに置かれている場合は後者 となる場合が多かった印象です。

## [ImageTagTransformer](https://kubectl.docs.kubernetes.io/references/kustomize/builtins/#_imagetagtransformer_)

リモートレポジトリの manifest を利用していたり、ある特定の image のみ変更したい場合に便利です。
[patches](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/patches/) でも同じことができますが、image だけの変更であれば ImageTagTransformer のほうがコード量が少なくて済みます。

少し前になりますが、Kubernetes が使用するレジストリが k8s.gcr.io から registry.k8s.io に変更となりました。
リモートレポジトリの manifest には、k8s.gcr.io が使われているけど registry.k8s.io にもイメージがプッシュされている場合を想定します。

image(name) と変更後の image(NewName) を images に定義することで `k8s.gcr.io/metrics-server/metrics-server` を `registry.k8s.io/metrics-server/metrics-server` に置き換えることが可能です。

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.2/components.yaml
images:
  - name: k8s.gcr.io/metrics-server/metrics-server
    NewName: registry.k8s.io/metrics-server/metrics-server
```

また、以下のように定義することで`postgres`を`my-registry/my-postgres:v1`に変更することが可能です。

```bash
images:
- name: postgres
  newName: my-registry/my-postgres
  newTag: v1
```

## [LabelTransformer](https://kubectl.docs.kubernetes.io/references/kustomize/builtins/#_labeltransformer_)

commonLabels に定義した label と selector が全リソースに追加されます。すでに同じ key で定義されている場合、上書きされます。
commonLbales を変更するし、Deployments やServices などのリソースの selector は、リソースがクラスタに適用された後は変更することができないため、設計時に注意が必要です。

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

commonLabels:
  Name: serivce-a
  owner: EigoOda
```

selector を追加する必要がない場合は、[labels](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/labels/) を使うことで label のみを追加することができます。
commonLabels と同様、リソースがクラスタに適用された後は変更することができません。


## [PrefixSuffixTransformer](https://kubectl.docs.kubernetes.io/references/kustomize/builtins/#_prefixsuffixtransformer_)

全リソースに prefix や suffix を追加することができます。
以下のようにすることで resources の全リソースの名前に zozo- prefix が付与されます。

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namePrefix: zozo-

resources:
- deployment.yaml
- secrets.yaml
```

## [replacement](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/replacements/)

特定のスキーマを置換することができます。
以下の例では、Deployment リソースの metadata.name を my-resource に置換します。

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

replacements:
- source:
    kind: Deployment
    fieldPath: metadata.name
  targets:
  - select:
      name: my-resource
```

以上、Kustomize の便利な機能でした！

[Kustomize]: https://kustomize.io/