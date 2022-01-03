---
title: "Kubernetesの非推奨(deplicated api)を自動検出する"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "pluto", "OSS"]
published: false
---

あけましておめでとうございます。
普段、インフラエンジニアをしていてKubernetesやCI/CD周りを主に触っています。

私の環境では、Kubernetesの環境はPRD/STG/DEVで3環境あり、Clusterのバージョンが異なっていることが多々あります。
また、最近はアプリケーションを`Helm`から`Argo CD`に移管したため
Deplicated apiの検出がChange logを見る以外では、なかなか把握しきれていなかったり、気づきにくい状態となっています。

以前は、`helm install/upgrade`の際、deplicated apiがterminalに表示されていたため、気づけていましたが、Argo CDによるGitOpsに移行したため
気づくタイミングがとても少なくなってしまいました。

そのため、CIに組み込み、自動で検知することにしました。

実装する前に調査した際、あまりこういった記事がなかったため
少しでも、Kubernetes deplicated apiを自動で検知する方法を探している方の参考になれば、幸いです。


## 何をどうするか？

とっても簡単です。
PlutoというOSSを使い、CIで1つのjobを作成するのみ。

今回は、静的なHelm chart内で特定のKubernetes versionのdeplicated apiが使われていないか検出します。


### Plutoとは？

[参考](https://github.com/FairwindsOps/pluto)

> Infrastructure-as-Code repos: Pluto can check both static manifests and Helm charts for deprecated apiVersions
> Live Helm releases: Pluto can check both Helm 2 and Helm 3 releases running in your cluster for deprecated apiVersions

静的manifestやHelm chart、Cluster内で動作しているHelm application内のdeplicated apiを検知することができるOSSです。


Plutoをローカルで試す。

#### Plutoのインストール方法

[こちら](https://pluto.docs.fairwinds.com/installation/)を参考に実施します。

#### Quickstart

公式の[QuickStart](https://pluto.docs.fairwinds.com/quickstart/)を参考に適当なmanifestを用意し、検証してみる

* helm chartを用意
```bash
$ helm create pluto-sample
$ cd pluto-sample
$ vi values.yaml
# ingress.enabledを `false` -> `true`に変更
```

* deplicated apiではない、apiVersionのmanifestを検証

```bash
# template/ingress.yamlを変更
$ vi template/ingress.yaml
--- 9 ~ 15行目
{{- if semverCompare ">=1.19-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: networking.k8s.io/v1
{{- else if semverCompare ">=1.14-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: networking.k8s.io/v1beta1
{{- else -}}
apiVersion: extensions/v1beta1
{{- end }}
---
を
---
apiVersion: networking.k8s.io/v1
---
に変更

# Cluster version 1.22.0を想定し、pluto コマンドでHelm chartを検証
$ helm template . | pluto detect --target-versions version=v1.22.0 -
There were no resources found with known deprecated apiVersions.
```

上記の結果から、対象のhelm chartでdeplicated apiを使っていないことがわかります。

* deplicated apiのapiVersionのmanifestを検証

```bash
# template/ingress.yamlを変更
$ vi template/ingress.yaml
---
apiVersion: networking.k8s.io/v1
---
を
---
apiVersion: networking.k8s.io/v1beta1
---
に変更

$ helm template . | pluto detect --target-versions version=v1.22.0 -
NAME                        KIND      VERSION                     REPLACEMENT            REMOVED   DEPRECATED
RELEASE-NAME-pluto-sample   Ingress   networking.k8s.io/v1beta1   networking.k8s.io/v1   true      true
```

上記の結果から、対象のhelm chartでdeplicated apiが使われていることがわかります。


## 実装方法

#### Gitlab CIの場合

QuickStartで実行したコマンドをCIで実行するだけです。とても簡単です。

#### Github Actionの場合

専用のActionsが用意されているのでこちらもとても簡単ですね。
[こちら](https://github.com/FairwindsOps/pluto#github-action-usage)に用意されています。


## 番外編

### outputを変更する方法

* markdown形式でoutputする

```bash
$ helm template . | pluto detect --target-versions version=v1.22.0 --output markdown -
|           NAME            | NAMESPACE |  KIND   |          VERSION          |     REPLACEMENT      | DEPRECATED | DEPRECATED IN | REMOVED | REMOVED IN |
|---------------------------|-----------|---------|---------------------------|----------------------|------------|---------------|---------|------------|
| RELEASE-NAME-pluto-sample | <UNKNOWN> | Ingress | networking.k8s.io/v1beta1 | networking.k8s.io/v1 | true       | v1.19.0       | true    | v1.22.0    |
```

### manifestやdeployされているhelmに対して実行する方法を紹介する

Quickstartにも書かれていますが、実行するフラグによって何を検証するか変えられます

* manifestを検証する場合
`pluto detect-files`

* Clusterにデプロイされているhelmを検証する場合
`pluto detect-helm`

* 静的helm chartを検証する場合
`pluto detect`