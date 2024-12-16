---
title: "Fluent Bit の概要"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Kubernetes", "Fluent Bit", "Container", "log", "metrics"]
published: true
---

# はじめに

EKS Pod のなどリソースなどから出力されているログをいろんなところに転送するには、Fluent-bit を使うと便利ですよね？
普段使っているのですが、概要すらあまりうまく理解できていなかったので少し調べてみました。

# Fluent Bit について

Fluentd 傘下のサブプロジェクトです。公式ドキュメントは、https://docs.fluentbit.io/manual になります。
Fluentd の後継として書かれていることもありますが、公式には "Fluent Bitは、Fluentdのアーキテクチャと一般的なデザインの最良のアイデアの上に設計され、構築されています。 どちらを選ぶかはエンドユーザーのニーズ次第です"と書かれており後継というわけではないようです（ただし実質的な後継のようなものだと感じてます）。
Fluentd はメモリーの使用量は多いが対応しているプラグインの数も多い。Fluent bit は軽量だが対応しているプラグインの数が Fluentd に比べて少ないという特徴があります。
![](/images/fluent-bit-investigation/fluend-fluentbit.png)
引用元: https://docs.fluentbit.io/manual/about/fluentd-and-fluent-bit

Linux、macOS、Windows、BSD 系 OS のログ、メトリクス、トレース用の高速で軽量な遠隔測定エージェントで複雑さを伴わずにさまざまなソースからの遠隔測定データの収集と処理を可能にするため、性能に重点を置いて作られています。

# アーキテクチャ

Source と Destination の間に Fluent Bit が入り、送受信を行います。

全体像
![](/images/fluent-bit-investigation/fluent-bit-overview.png)
引用元: https://fluentbit.io/

Fluent Bit の内部処理
![](/images/fluent-bit-investigation/fluent-bit-inside.png)
引用元: https://docs.fluentbit.io/manual/stream-processing/overview

Data Pipeline は、 Input, Parser, Filter, Store, Router, Output という流れとなります。
詳細は、https://docs.fluentbit.io/manual/concepts/data-pipeline をご確認ください。


# コンセプト

## キーコンセプト

Event or Record, Filtering, Tag, Timestamp, Match, Structured Message がキーコンセプトです。

**Event or Record**

Fluent Bit によって取得されたログまたはメトリックに属するすべての受信データはイベントまたはレコードとみなされ、イベントは以下の要素によって構成されます。
- timestamp
- key/value metadata(v2.1.0 以上)
- payload

イベントのフォーマットは、`[[TIMESTAMP, METADATA], MESSAGE]`となります。要素の説明は以下の通りです。
- *TIMESTAMP*: integer か float で表されます
- *METADATA*: イベントメタデータを含んだオブジェクトで空である可能性もあります
- *MESSAGE*: イベントのボディを含むオブジェクトです

:::message
 v2.1.0 以前は、（[TIMESTAMP, MESSAGE]）となり、METADATA はありません。
:::

**Filtering**

イベントの変更、追加、削除するプロセスです。以下のようなことができます。
- IPアドレスやメタデータのような特定の情報を追加
- イベントコンテンツの特定部分の選択
- 特定のパターンに一致するイベントを削除

**Tag**

Fluent Bit によって取り込まれたすべてのイベントにはタグが割り当てられます。これらはどの Filter や Output フェーズを通過しなければならないかを決定するために、後の段階で Router が使用する内部文字列になります。
ほとんどのタグはコンフィグレーションで手動割当ができます。もしタグが指定されていなかった場合、Fluent Bit はそのイベントが生成された Input プラグインの名前を割り当てます
タグ付けされたレコードには必ずマッチング・ルールが必要です。 タグとマッチの詳細については、[ルーティング](https://docs.fluentbit.io/manual/concepts/data-pipeline/router)を参照してください。

**Timestamp**

タイムスタンプは、イベントが作成された時間を表します。すべてのイベントにはタイムスタンプがあり、入力プラグインによって設定されるか、データ解析処理によって発見されます。
タイムスタンプは、`SECONDS.NANOSECONDS`で表されていて小数整数の数値となっています。

**Match**

Fluent Bitでは、収集・処理したイベントを1つまたは複数の宛先にルーティングできます。Matchは、タグが定義されたルールに一致するイベントを選択するルールです。
詳細は、https://docs.fluentbit.io/manual/concepts/data-pipeline/router をご確認ください。

**Structured Message**

ソースイベントは構造を持つことができます。構造体はイベントメッセージ内の key と value のセットを定義し、データ変更に対する高速な操作を実装しています。すべてのイベントメッセージを構造化メッセージとして扱います。


## バッファリング

Fluent Bit がデータを処理するとき、レコード・ログが配信される前にシステムメモリ（ヒープ）を一時的な保管場所として使用します。 レコードはこのプライベートメモリ領域で処理されます。
バッファリングとは、レコードを保存し、以前のデータが処理され配信される間、受信データを保存し続ける機能です。メモリ内のバッファリングは最も高速なメカニズムですが、データの安全性に対処したり制約のある環境でサービスによるメモリ消費を抑えるために特別な戦略を必要とするシナリオもあります。
バッファーの戦略は、バックプレッシャーや配信失敗に関連する問題を解決するようにデザインされています。プライマリーはインメモリ、セカンダリーはファイルシステムというメカニズムになっています。ハイブリットソリューションは、どのユースケースのデータプロセスに対しても安全でハイパフォーマンスに行うことができる。
これらのメカニズムは相互に排他的なものではありません。データが処理されたり配信されたりする準備ができたら、それは常にメモリ上にあります。一方、キュー内の他のデータは処理されメモリに移動される準備ができるまでは、ファイルシステム内にあるかもしれません。
詳細は、 https://docs.fluentbit.io/manual/administration/buffering-and-storage ご確認ください。


以上、Fluent Bit の超概要でした。かなり浅くしか紹介できなかったので、より深い情報を求めている方が公式ドキュメントをご確認いただければと思います。
