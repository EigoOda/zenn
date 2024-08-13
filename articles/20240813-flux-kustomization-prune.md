---
title: "Flux Kustomization の prune 機能"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "Flux"]
published: true
---

Flux の Kustomization で prune 機能を利用してみます。
Flux Kustomization におけるprune 機能とは、以前に適用されたオブジェクトが現在のリビジョンにない場合、それらのオブジェクトがクラスターから削除されるものです。

構成は、ECR に artifact(Kubernetes manifests) を格納し、OCIRepository リソースで ECR から artifact を pull、Kustomization リソースで OCIRepository に pull されている artifact を Kubernetes クラスターに適用するイメージです。

では早速試してみます。

以下利用する Kubernetes manifests です。
ECR と Kubernetes クラスター間は認証周りの設定が完了していて、すでに通信できるようになっている状態を想定しています。

作成したり削除したりする リソース(1)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oddy-test-deployment
  namespace: default
  labels:
    app: oddy-test
spec:
  selector:
    matchLabels:
      app: oddy-test-deployment
  template:
    metadata:
      labels:
        app: oddy-test-deployment
    spec:
      containers:
      - name: oddy-test-deployment
        image: public.ecr.aws/eks-distro/kubernetes/pause:3.7
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: oddy-test
  name: oddy-test-service
  namespace: default
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  selector:
    app: oddy-test-deployment
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: oddy-test-hpa
  namespace: default
  labels:
    app: oddy-test
spec:
  maxReplicas: 2
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: oddy-test-deployment
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: oddy-test-pdb
  namespace: default
  labels:
    app: oddy-test
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: oddy-test-deployment
```

Flux リソース(2)

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: oddy-test
spec:
  interval: 1m
  url: oci://xxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/oddy-test-repository
  insecure: true
  ref:
    tag: v1
  provider: aws
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: oddy-test
spec:
  interval: 1m0s
  path: .
  prune: true
  sourceRef:
    kind: OCIRepository
    name: oddy-test
  suspend: false
```

(1) を ECR に push 後、(2) を Kubernetes クラスターに作成し、Deployment リソースなどを作成します。

```bash
$ k get deployments,service,hpa,pdb -n default -l app=oddy-test
NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/oddy-test-deployment   1/1     1            1           6m41s

NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/oddy-test-service   ClusterIP   172.20.234.0   <none>        80/TCP    6m41s

NAME                                                REFERENCE                         TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/oddy-test-hpa   Deployment/oddy-test-deployment   <unknown>/70%   1         2         1          6m41s

NAME                                       MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
poddisruptionbudget.policy/oddy-test-pdb   1               N/A               0                     6m41s
```

全リソースが作成されたことを確認して PDB リソースを削除してみます。
manifest の PDB 部分をコメントアウトして ECR に push し、Kubernetes クラスターに反映されるのを気長に待ちます。

少し待っていると Kustomization リソースで以下のようなイベントが記録されました。
新リビジョンに PDB が存在しないことを kustomize-controller が検知し、PDB リソースを削除してそうです。

```text
   Normal  ReconciliationSucceeded  53s (x9 over 8m42s)  kustomize-controller  (combined from similar events): Reconciliation finished in 118.668337ms, next run in 1m0s    │
   Normal  Progressing              53s                  kustomize-controller  PodDisruptionBudget/default/oddy-test-pdb deleted
```

実際にリソースを見てみます。

```bash
$ k get deployments,service,hpa,pdb -n default -l app=oddy-test
NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/oddy-test-deployment   1/1     1            1           16m

NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/oddy-test-service   ClusterIP   172.20.234.0   <none>        80/TCP    16m

NAME                                                REFERENCE                         TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/oddy-test-hpa   Deployment/oddy-test-deployment   <unknown>/70%   1         2         1          16m
```

実行結果から PDB が削除されていることが確認できました。便利ですね。

検証用のリソースを整理した際に (2) を削除したところ、(1) のリソースも一緒に削除されていたのでその点は注意が必要です。

```bash
$ k delete -f gitops_toolkit.yaml
ocirepository.source.toolkit.fluxcd.io "oddy-test" deleted
kustomization.kustomize.toolkit.fluxcd.io "oddy-test" deleted

$ k get deployments,service,hpa,pdb -n default -l app=oddy-test
No resources found in default namespace.
```
