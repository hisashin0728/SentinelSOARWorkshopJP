# はじめに
> Microsoft Sentinel / SOAR / ロジックアプリのためのワークショップ

このレポジトリは Microsoft Sentinel の SOAR 機能活用するために、ロジックアプリを勉強するためのコンテンツを提供しています。

# 前提条件
> 前提条件を以下に示します。演習によるリソース作成、利用分は有償になります。

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/69bbd9df-73e2-4b8c-b9e2-44b3e7918578)

- Microsoft Sentinel の環境を自テナントに有すること
- Microsoft Sentinel に対して、何らかの分析ルールによるアラート発砲が可能であること
  - [コンテンツハブより、「Defender XDR」コネクタをインストールして Defender XDR と接続が完了していること](https://learn.microsoft.com/ja-jp/azure/sentinel/connect-microsoft-365-defender?tabs=MDE)
  - [Microsoft Defender for Cloud アラートを Microsoft Sentinel に取り込む](https://learn.microsoft.com/ja-jp/azure/sentinel/connect-defender-for-cloud)
- (**もしくは**) 既に Microsoft Sentinel のテスト環境を構築済みで、以下のようなインシデント情報を残していること
    - MDE で検知したアラート（アラート内容は何でもOK）で、ホスト情報が付与されたインシデントが作られていること
    - Microsoft Entra ID Protection (旧 AAD IDP) をテスト実装されており、ユーザー名のエンティティ情報が付与されたインシデントが作られていること
- [参考] サンプルアラート生成方法 (本演習は Sentinel インシデントから ``Host`` や ``Account`` 種別のエンティティを元にロジックアプリを起動させます)
    - [MDE for Windows サンプルアラート生成](https://learn.microsoft.com/ja-jp/microsoft-365/security/defender-endpoint/run-detection-test?view=o365-worldwide)
    - [MDE for Linux サンプルアラート生成](https://learn.microsoft.com/ja-jp/microsoft-365/security/defender-endpoint/linux-install-manually?view=o365-worldwide#client-configuration)
    - [Entra ID Protection サンプルアラート生成](https://learn.microsoft.com/ja-jp/entra/id-protection/howto-identity-protection-simulate-risk)

# 演習コンテンツ
> ワークショップのコンテンツはこちらです。

1. [演習1. Sentinel インシデントトリガー時にどのような情報が連携されるのかを理解する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work1.md)
2. どちらかを選択
    - [演習2M. 標準コネクタを用いて、Defender XDR のアドバンスドハンティングクエリーを実行する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work2M.md)
      - 定期的に Defender XDR (Microsoft 365 Defender) にアドバンスハンティングクエリーをかけてみよう
      - ハンティングした結果を成型して通知してみよう
    - [演習2A. 標準コネクタを用いて、Azure Resource Graph (ARG) に対して定期的に監視クエリーを実行する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work2A.md)
      - 定期的に Log Analytics ワークスペースから Azure Resourec Graph (ARG)にクエリーをかけてみよう
      - ハンティングした結果を成型して通知してみよう
3. [演習3. Sentinel インシデントトリガーからエンティティ属性を抽出して、Defender XDR のアドバンスドハンティングクエリーを実行する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work3.md)
    - Sentinel のインシデントをトリガーに、エンティティに含まれる Host 情報から Defender XDR (Microsoft 365 Defender) にアドバンスハンティングクエリーをかけてみよう
    - ハンティングした結果を成型して通知してみよう
5. [演習4. Sentinel インシデントトリガーからエンティティ属性を抽出して、Microsoft Graph API を叩いて Entra ID 情報を取得する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work4.md)
    - Sentinel インシデントをトリガーとして、Microsoft Graph API を叩いてみよう
    - ハンティングした結果を成型して通知してみよう

# アンケートのお願い
> 最後にアンケートにご協力ください

- 本ワークショップをより良いものにするため、[アンケート](https://forms.office.com/r/3Ed17BkkG1)にご記入下さい！

# 免責事項
> 本レポジトリの演習は課金が発生します。

- 本レポジトリの演習によって発生するコストについては、利用するユーザーが責任を負います。
- 本レポジトリの演習によって作成される環境から出力される内容について、作成者は責任を負いません。
- 本レポジトリはオープンソースです。 
