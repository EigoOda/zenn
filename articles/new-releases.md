---
title: "ソフトウェアのリリース通知を受け取る"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "GitHub"]
published: true
---

インフラからSREに転職しまして、現在、AWS/Kubernetes/Istio を触っているわけですが、
Kubernetesや関連コンポーネントはリリース頻度が高く
気づいたら最新から3マイナーバージョン遅れている なんてことがよくあるわけです。

Kubernetesと2~3つの関連コンポーネント程度であれば、
なんとか気にして追うことができるのですが
導入しているコンポーネントが10とかを超えてくると
何がどのバージョンを使っているのすらも忘れていたりします。

そのため、新しいバージョンがリリースされた際に通知してくれる **new releases** というサービスがあったので、紹介したいと思います。

https://newreleases.io/

サービス名そのままですが、新しいリリースをお知らせしてくれるサービスとなります。

個人的に嬉しいポイントとしては、
1. 複数のプロバイダー(GitHub,GitLab,Docker Hubなど23つ)から、選べるので、基本的にトラックできる
2. GitHub アカウントでログインすると、Starをつけたrepositoryは1クリックで登録できる
3. Slack,Discord,WebHookなど通知方法を選べる(mailも選べます)

少しだけ各ポイントの説明します。

### 1. 複数のプロバイダー(GitHub,GitLab,Docker Hubなど23つ)から、選べるので、基本的にトラックできる

GitHub で管理されているものは、`owner/project`を登録するだけ
Docker Hubで管理されているものは、`image name`を登録するだけ
Ruby Gemsで管理されているものは、`gem name`を登録するだけで、新しいリリースが出るたびに通知を受け取ることができます。


### 2. GitHub アカウントでログインすると、Starをつけたrepositoryは1クリックで登録できる

GitHubアカウントがStarをつけたrepositoryは
`import from`というセクションに表示され、1クリックで通知を設定することができます。

試してはないのですが
他にもDocker Hub アカウントでログインした場合も、同様にStarをつけたものを簡単に登録できたり
Goのpackageの場合は、go.mod をアップロードするだけで、対象のpackageを登録することもできそうです。


### 3. Slack,Discord,WebHookなど通知方法を選べる(mailも選べます)

ここが一番嬉しかったところですが、
SlackやDiscordとの連携がとても簡単にでき、チャネルに通知することができました。


今のところいい感じなので、ぜひ皆さんも使ってみてください。
