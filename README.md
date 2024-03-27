# はじめに
> Microsoft Sentinel / SOAR / ロジックアプリのワークショップ
このレポジトリは Microsoft Sentinel の SOAR 機能活用するために、ロジックアプリを勉強するためのコンテンツを提供しています。

# 前提条件
> 前提条件を以下に示します。演習によるリソース作成、利用分は有償になります。

- Microsoft Sentinel の環境を自テナントに有すること
- Microsoft Sentinel に対して、何らかの分析ルールによるアラート発砲が可能であること
  - [Microsoft Defender for Cloud とデータコネクタを接続し、Microsoft Defender for Cloud のサンプルアラートを発砲](https://learn.microsoft.com/ja-jp/azure/defender-for-cloud/alert-validation)
  - MDE (Microsoft Defender for Endpoint) が導入された Windows / Linux より、EICAR や Test Alert を発砲
    - [Windows OS 向け MDE テスト方法](https://learn.microsoft.com/ja-jp/microsoft-365/security/defender-endpoint/run-detection-test?view=o365-worldwide#verify-microsoft-defender-for-endpoint-onboarding-of-a-device-using-a-powershell-detection-test)
    - [Linux OS 向け MDE テスト方法](https://learn.microsoft.com/ja-jp/microsoft-365/security/defender-endpoint/linux-exclusions?view=o365-worldwide#validate-exclusions-lists-with-the-eicar-test-file)
  - [Microsoft Entra ID Protection による Test Alert を発砲](https://learn.microsoft.com/ja-jp/entra/id-protection/howto-identity-protection-simulate-risk)できること
- (**もしくは**) 既に Microsoft Sentinel のテスト環境を構築済みで、以下のようなインシデント情報を残していること
    - MDE で検知したアラート（アラート内容は何でもOK）で、ホスト情報が付与されたインシデントが作られていること
    - Microsoft Entra ID Protection (旧 AAD IDP) をテスト実装されており、ユーザー名のエンティティ情報が付与されたインシデントが作られていること

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
5. [演習4. Sentinel インシデントトリガーからエンティティ属性を抽出して、Microsoft Graph API を叩いて Entra ID 情報を取得する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work4.md)
    - Sentinel インシデントをトリガーとして、Microsoft Graph API を叩いてみよう
    - ハンティングした結果を成型して通知してみよう

# 免責事項
> 本レポジトリの演習は課金が発生します。

- 本レポジトリの演習によって発生するコストについては、利用するユーザーが責任を負います。
- 本レポジトリの演習によって作成される環境から出力される内容について、作成者は責任を負いません。
- 本レポジトリはオープンソースです。 
