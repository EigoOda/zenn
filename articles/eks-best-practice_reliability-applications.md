---
title: "【要約】EKS Best Practice Guides - Reliability - Applications"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "アプリケーション", "Reliability"]
published: false
---

[EKS Best Practices Guides](https://aws.github.io/aws-eks-best-practices/) の [Reliability](https://aws.github.io/aws-eks-best-practices/reliability/docs/) の要約です。
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


