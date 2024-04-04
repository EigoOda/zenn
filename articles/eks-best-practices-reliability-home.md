---
title: "【要約】EKS Best Practices Guides - Reliability - Home"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "アプリケーション", "Reliability"]
published: true
---

[EKS Best Practices Guides](https://aws.github.io/aws-eks-best-practices/) - [Reliability](https://aws.github.io/aws-eks-best-practices/reliability/docs/) - [Home](https://aws.github.io/aws-eks-best-practices/reliability/docs/) の要約です。
誤訳している可能性がありますので、ご注意ください。

# Amazon EKS Best Practices Guide for Reliability

Reliability セクションでは、EKS 上で実行されるワークロードを回復力を高め、高可用性にするためのガイダンスを提供します。

## Home

このガイドは、EKS で可用性と耐障害性の高いサービスを開発・運用したい開発者やアーキテクトを対象としています。
本セクションは、アプリケーション、コントロールプレーン、データプレーンのトピックで分かれています。

### システムの信頼性（Reliability）とは何か

あるシステムが、一定期間の環境の変化があるにもかかわらず一貫して機能し、需要を満たすことができれば、それは信頼性があると呼ぶことができます。それを達成するためには、システムが障害を検出し、自動的に自己回復し、需要に応じて拡張する能力を持たなければなりません。

ミッションクリティカルなアプリケーションやサービスを確実に運用するための基盤として Kubernetes を利用できます。ただ、コンテナベースのアプリケーション設計原則を取り入れる以外にも、ワークロードを確実に実行するには、信頼性の高いインフラストラクチャも必要です。Kubernetes のインフラは、コントロールプレーンとデータプレーンで構成されています。

### コントロールプレーン

EKS は、高い可用性と耐障害性を持つように設計された Kubernetes コントロールプレーンを提供します。
AWS が Kubernetes コントロールプレーンの信頼性に責任を持っており、AWSリージョン内の3つのアベイラビリティゾーンで Kubernetes コントロールプレーンを実行します。Kubernetes API Server と etcd クラスタの可用性とスケーラビリティを自動的に管理します。

### データプレーン

データプレーンの信頼性に対する責任は、利用者と AWS の間で共有されており、EKS は Kubernetes のデータプレーンに3つのオプションを提供しています。
最も管理されたオプションである Fargate は、データプレーンのプロビジョニングとスケーリングを行います。
2番目のオプションであるマネージドノードグループは、データプレーンのプロビジョニングとアップデートを行います。
そして最後に、セルフマネージドノードがデータプレーンの最小管理オプションです。AWS マネージドデータプレーンであればあるほど、利用者の責任は少なくなります。

### マネージドノードグループ

マネージドノードグループは EC2 のプロビジョニングとライフサイクル管理を自動化します。EKS API（EKS コンソール、AWS API、AWS CLI、CloudFormation、Terraform、eksctl）を使用して、作成や拡張、アップグレードできます。EKS は、ノードのアップグレードも管理しますが、アップグレード作業自体は自分で行う必要があります。ノードに自動的にタグが付けられ、Cluster Autoscaler で使用できるようになります。

### セルフマネージドノード

セルフマネージドノードを実行する場合、EKS に最適化された Linux AMI を使用してワーカーノードを作成できます。AMI とノードのパッチ適用、アップグレードは利用者の責任です。
セルフマネージドノードのプロビジョニングには、eksctl、CloudFormation、またはInfrastructure as Codeツールを使用するのがベストプラクティスです。ワーカーノードをアップグレードするときは、新しいノードに移行することを検討してください。移行プロセスでは、古いノードグループに NoSchedule taints を付与し、新しいスタックが既存のポッドのワークロードを受け入れる準備ができた後にノードをドレインするからです。ただし、セルフマネージドノードのインプレースアップグレードを実行できます。

以下、データプレーンごとの責任分界点の図です。
![responsibility_map](/images/eks-best-practice/reliability/responsibility_map.png)

引用元: https://aws.github.io/aws-eks-best-practices/reliability/docs/
