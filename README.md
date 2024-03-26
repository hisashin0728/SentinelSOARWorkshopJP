# はじめに
> Microsoft Sentinel / SOAR / ロジックアプリのワークショップ
このレポジトリは Microsoft Sentinel の SOAR 機能活用するために、ロジックアプリを勉強するためのコンテンツを提供しています。

# 前提条件
> 前提条件を以下に示します。演習によるリソース作成、利用分は有償になります。

- Microsoft Sentinel の環境を自テナントに有すること
- Microsoft Sentinel に対して、何らかの分析ルールによるアラート発砲が可能であること
  - 本演習では Microsoft Defender for Cloud とデータコネクタを接続し、Microsoft Defender for Cloud のサンプルアラートによる効果を想定しています。

# 演習コンテンツ
> ワークショップのコンテンツはこちらです。
> 
1. [演習1. Sentinel インシデントトリガー時にどのような情報が連携されるのかを理解する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work1.md)
2. [演習2. 標準コネクタを用いて、Defender XDR のアドバンスドハンティングクエリーを実行する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work2.md)
    - 定期的に Defender XDR (Microsoft 365 Defender) にアドバンスハンティングクエリーをかけてみよう
    - ハンティングした結果を成型して通知してみよう
3. [演習3. Sentinel インシデントトリガーからエンティティ属性を抽出して、Defender XDR のアドバンスドハンティングクエリーを実行する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work3.md)
    - Sentinel のインシデントをトリガーに、エンティティに含まれる Host 情報から Defender XDR (Microsoft 365 Defender) にアドバンスハンティングクエリーをかけてみよう
    - ハンティングした結果を成型して通知してみよう
5. RESTAPI を使ってみよう
    - Sentinel インシデントをトリガーとして、Microsoft Graph API を叩いてみよう
    - ハンティングした結果を成型して通知してみよう

# 免責事項
> 本レポジトリの演習は課金が発生します。

- 本レポジトリの演習によって発生するコストについては、利用するユーザーが責任を負います。
- 本レポジトリの演習によって作成される環境から出力される内容について、作成者は責任を負いません。
- 本レポジトリはオープンソースです。 
