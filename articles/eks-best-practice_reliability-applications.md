---
title: "【要約】EKS Best Practice Guides - Reliability(1/2)"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "アプリケーション", "Reliability"]
published: false
---

[EKS Best Practices Guides](https://aws.github.io/aws-eks-best-practices/) の [Reliability](https://aws.github.io/aws-eks-best-practices/reliability/docs/) 内の [Home]() と [Applications](https://aws.github.io/aws-eks-best-practices/reliability/docs/application/) の要約です。
誤訳している可能性がありますので、ご注意ください。

# Amazon EKS Best Practices Guide for Reliability

EKS 上で実行されるワークロードを回復力を高め、高可用性にするためのガイダンスを提供します。

## Home

このガイドは、EKS で可用性と耐障害性の高いサービスを開発・運用したい開発者やアーキテクトを対象としています。
アプリケーション、コントロールプレーン、データプレーンのトピックで分かれています。

システムの信頼性（Reliability）とは何か。あるシステムが、一定期間の環境の変化にもかかわらず一貫して機能し需要を満たすことができれば、それは信頼性があると呼ぶことができます。これを達成するためには、システムは障害を検出し、自動的に自己回復し、需要に応じて拡張する能力を持たなければなりません。

ミッションクリティカルなアプリケーションやサービスを確実に運用するための基盤として Kubernetes を利用できます。しかし、コンテナベースのアプリケーション設計原則を取り入れる以外にも、ワークロードを確実に実行するには、信頼性の高いインフラストラクチャも必要です。Kubernetes のインフラは、コントロールプレーンとデータプレーンで構成されています。

EKS は、高い可用性と耐障害性を持つように設計されたプロダクショングレードの Kubernetes コントロールプレーンを提供します。
AWS が Kubernetes コントロールプレーンの信頼性に責任を持っており、AWSリージョン内の3つのアベイラビリティゾーンで Kubernetes コントロールプレーンを実行します。Kubernetes API サーバと etcd クラスタの可用性とスケーラビリティを自動的に管理します。

データプレーンの信頼性に対する責任は、利用者と AWS の間で共有されており、EKS は Kubernetes のデータプレーンに3つのオプションを提供しています。最も管理されたオプションである Fargate は、データプレーンのプロビジョニングとスケーリングを行います。2番目のオプションである Managed node groups は、データプレーンのプロビジョニングとアップデートを行います。そして最後に、self-managed node がデータプレーンの最小管理オプションです。AWS マネージドデータプレーンであればあるほど、利用者の責任は少なくなります。

Managed node groups は EC2 （ノード）のプロビジョニングとライフサイクル管理を自動化します。EKS API（EKS コンソール、AWS API、AWS CLI、CloudFormation、Terraform、eksctl）を使用して、作成や拡張、アップグレードできます。EKS は、ノードの更新も管理しますが、更新は自分で行う必要があります。ノードに自動的にタグが付けられ、Cluster Autoscaler で使用できるようになります。

self-managed node を実行する場合、EKS に最適化された Linux AMI を使用してワーカーノードを作成できます。AMI とノードのパッチ適用、アップグレードは利用者の責任です。self-managed node のプロビジョニングには、eksctl、CloudFormation、またはInfrastructure as Codeツールを使用するのがベストプラクティスです。
ワーカーノードをアップグレードするときは、新しいノードに移行することを検討してください。移行プロセスでは、古いノードグループに NoSchedule taints を付与し、新しいスタックが既存のポッドのワークロードを受け入れる準備ができた後にノードをドレインするからです。ただし、self-managed node のインプレースアップグレードを実行できます。

以下、データプレーンごとの責任分界点の図です。
![](./images/eks-best-practice/reliability/responsibility_map.png)

引用: https://aws.github.io/aws-eks-best-practices/reliability/docs/


## Applications

### Running highly-available applications

Kubernetesは、宣言的な管理により、アプリケーションを一度セットアップすれば、継続的に現在の状態と望ましい状態を一致させようとします。

#### Singleton Podの実行を避ける

Singleton （パターン）とは、そのクラスのインスタンスが1つしか生成されないことを保証するデザインパターン[1] とのことですが、Kubernetes においては、アプリケーションを単一 Pod で動かしていることと読み替えることができます。
単一のPodで実行されている場合、そのPodが終了するとアプリケーションは利用できなくなります。そのため、Pod の代わりに Deployment を作成します。Deployment で作成された Pod が失敗したり終了したりすると、Deploymentコ ントローラが新しい Pod を起動して、指定された数の Pod が常に実行されるようになります。

#### 複数のレプリカを実行する

Deployment を使用して複数のレプリカを実行すると、可用性を向上させることができます。Horizontal Pod Autoscaler(HPA) を使用することで、需要に基づいてレプリカを自動的にスケールさせることができます。

### 複数のノードにレプリカをスケジュールする

すべてのレプリカが同じノードで実行されていて、そのノードが利用できなくなった場合、複数のレプリカを実行してもあまり役に立ちません。そのため PodAntiAffinity または、PodTopologySpreadConstraints を使用して、Deployment のレプリカを複数のノードに分散させることを検討してください。
複数のAZにまたがってレプリカを実行することで、アプリケーションの信頼性をさらに向上させることができます。

### Pod anti-affinity を利用する

以下のマニフェストは、Kubernetesスケジューラに対して、別々のノードや AZ にPodを配置することを**優先（preffered）**するように指示します。別々のノードやAZにスケジュールすることを**必須（required）**としないのは、各AZで Pod が実行されている場合、Pod をスケジュールできなくなってしまうからです。アプリケーションが3つのレプリカだけを必要とする場合は、topologyKey: topology.kubernetes.io/zone にrequiredDuringSchedulingIgnoredDuringExecution を使用すれば、Kubernetes スケジューラは同じ AZ に2つ以上の Pod をスケジュールしません。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spread-host-az
  labels:
    app: web-server
spec:
  replicas: 4
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - web-server
              topologyKey: topology.kubernetes.io/zone
            weight: 100
          - podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - web-server
              topologyKey: kubernetes.io/hostname
            weight: 99
      containers:
      - name: web-app
        image: nginx:1.16-alpine
```

### Pod topology spread constraints を利用する











- [1](https://ja.wikipedia.org/wiki/Singleton_%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3)
