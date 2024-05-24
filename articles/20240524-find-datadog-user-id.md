---
title: "Datadog ユーザの ID を探す方法"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Terraform", "Datadog"]
published: true
---

最近、Datadog に [Datadog Teams](https://docs.datadoghq.com/ja/account_management/teams/) が追加されたので試してみたところ、Datadog ユーザの ID を取得する方法の情報があまりなかったので、記事として残します。

## TL;DR

- https://app.datadoghq.com/organization-settings/users で対象のユーザを検索し、そのユーザをクリックするとアドレスバーに以下のように表示されます
  - https://app.datadoghq.com/organization-settings/users?filter=xxxxxx&user_id=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

## Datadog teams を作成する

Terraform で Datadog team を作成する場合は、[datadog_team](https://registry.terraform.io/providers/DataDog/datadog/latest/docs/resources/team) 、Datadog team にメンバーを参加させたい場合は、[datadog_team_membership](https://registry.terraform.io/providers/DataDog/datadog/latest/docs/resources/team_membership) を利用します。

Datadog team は特に迷うことなく作成できたのですが、Datadog team membership リソースの作成でユーザの ID が必要のため、探すときに手間取りました。

### 試したこと

SaaS のユーザ情報を取得する際に API を叩くことが多いので、[List all users](https://docs.datadoghq.com/api/latest/users/#list-all-users) で試してみましたが最大で取得できる page に制限があったので、欲しい情報が手に入りませんでした。（私のやり方が悪いかもしれません）

コンソールからも取得できたりすることがあるのでそちらも試してみたところ、https://app.datadoghq.com/organization-settings/users から取得することができました。
Users ページでユーザ ID を取得したいユーザを検索し、対象のユーザをクリックするとアドレスバーに以下のように表示されました。user_id key の value がユーザID です。
  - https://app.datadoghq.com/organization-settings/users?filter=xxxxxx&user_id=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
