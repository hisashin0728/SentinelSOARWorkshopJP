# 演習4. Sentinel インシデントトリガーからエンティティ属性を抽出して、Microsoft Graph API を叩いて Entra ID 情報を取得する
> RESTAPI の接続方法、Microsoft Graph の活用を Challenge してみましょう

Microsft Sentinel のインシデント検知をトリガーとして、インシデントに含まれるエンティティ情報から Microsoft Graph API を叩いて、Entra ID のユーザー情報を付与してみましょう。 
構成イメージは以下の通りです<p>

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/03cf1312-856e-4fb6-a34d-0ff898600940)

# [理解] Microsoft Graph を触ってみる
> はじめに Microsoft Graph Explorer を触ってみましょう

- Microsoft Graph については、[公式 Docs](https://learn.microsoft.com/ja-jp/graph/overview) を参照下さい
- [Graph エクスプローラー](https://learn.microsoft.com/ja-jp/graph/graph-explorer/graph-explorer-overview)の使い方も参照下さい

[https://learn.microsoft.com/ja-jp/graph/graph-explorer/graph-explorer-overview](https://developer.microsoft.com/en-us/graph/graph-explorer)https://developer.microsoft.com/en-us/graph/graph-explorer

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/1738f4b2-40cc-4e08-8c88-d48dfab535ad)

- 本演習では[ユーザーの取得](https://learn.microsoft.com/ja-jp/graph/api/user-get?view=graph-rest-1.0&tabs=http)クエリーを用います
  - ユーザ名を入れただけの規定値クエリーでは限定的な情報のみ応答することが分かります [Docs 情報](https://learn.microsoft.com/ja-jp/graph/api/user-get?view=graph-rest-1.0&tabs=http#example-1-standard-users-request)
    - ``businessPhones``
    - ``displayName``
    - ``givenName``
    - ``id``
    - ``jobTitle``
    - ``mail``
    - ``mobilePhone``
    - ``officeLocation``
    - ``preferredLanguage``
    - ``surname``
    - ``userPrincipalName``
  - 部署名フィールド(department) を付与するにはどうしたら良いでしょうか？
    - [$selectを使用してユーザーの特定のプロパティを取得する](https://learn.microsoft.com/ja-jp/graph/api/user-get?view=graph-rest-1.0&tabs=http#example-3-use-select-to-retrieve-specific-properties-of-a-user)

```
GET https://graph.microsoft.com/v1.0/users/87d349ed-44d7-43e1-9a83-5f2406dee5bd?$select=displayName,givenName,postalCode,identities
```
<img width="797" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/66a9ce25-3710-4a4e-a93b-56ed981a6da6">

- 出来ますね！
  - Microsoft Graph を使いこなすことで、必要となる情報を取得できることが分かりました

# 事前準備
## Sentinel のインシデントトリガーのロジックアプリを作成する　
> これまでの演習と同様に、インシデントトリガーのロジックアプリを作成しましょう
- Sentinel のオートメーションルールから、「インシデントトリガーを使用したプレイブック」を作成します<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/bf2a9e25-4554-4a2d-9168-3854cde38da8)
<img width="423" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/8604d868-6952-4340-9fe4-f3d0a77427d2">

## 作成されたロジックアプリに対して、マネージド ID を有効にする
> ロジックアプリが Microsoft Graph API に認証できるようにマネージド ID を有効にします
- ロジックアプリの ID より、システム割り当て済みマネージド ID を有効にして保存します<p>

<img width="744" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/223a5910-c627-46b3-ae8a-9c8c8713258f">

## マネージド ID に対して、Entra ID で「User.Read.All」権限を付与する
Microsoft Graph に接続するためには、Entra ID でマネージド ID に対して API のアクセス許可を与える必要があります。<p>
2024.3 現在、マネージド ID に対する権限の付与は、残念ながら Azure ポータル側からの設定に対応しておりません。<p>
Powershell を用いて権限を付与しましょう。

- 自PC環境で設定する方はこちら
  - 手順は[公式 Docs](https://learn.microsoft.com/ja-jp/powershell/microsoftgraph/installation?view=graph-powershell-1.0) に掲載されています
  - [PowerShell 5.1 以上のインストール](https://learn.microsoft.com/en-us/powershell/scripting/windows-powershell/install/installing-windows-powershell#upgrading-existing-windows-powershell)
  - [.NET Framework 4.7.2 以上のインストール](https://learn.microsoft.com/en-us/dotnet/framework/install/)
  - PowerShellGet の最新版のアップデート ``Install-Module PowerShellGet``
  - ``Micosoft.Graph`` [モジュールのインストール](https://learn.microsoft.com/ja-jp/powershell/microsoftgraph/installation?view=graph-powershell-1.0#installation)
- Azure Cloudshell の手順 (※お勧め)


# 1. Sentinel エンティティ情報を格納する
> アカウントタイプのエンティティのみに抽出する

- 後段の Microsoft.Graph に連携するため、ここでは ``Account`` 型のエンティティを抽出します
- 標準コネクタ「Microsoft Sentinel」->「エンティティ - アカウントを取得」を接続し、Sentinel インシデントトリガーの情報から「エンティティ」の一覧をセットします

<img width="876" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/8de88f80-8679-4425-8a9e-3b3066a2c653">

# 2. For Each 処理で JSON アレイの内容毎に処理する
> For Each 処理でエンティティのアレイを分解しましょう

 - [演習3 のプロセス](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work3.md#2-json-%E3%82%A2%E3%83%AC%E3%82%A4%E3%81%AE%E6%83%85%E5%A0%B1%E3%82%92-for-each-%E3%83%AB%E3%83%BC%E3%83%97%E3%81%A7%E5%88%86%E8%A7%A3%E3%81%97%E3%81%A6%E5%87%A6%E7%90%86%E3%81%95%E3%81%9B%E3%82%8B)同様に、エンティティで抽出した情報は JSON アレイになっているため（※複数のアカウントが検出する可能性がある）、For Each 処理を行います

<img width="808" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/02a2af7a-a867-4245-832a-5f8ee9a10a1a">

# 3. Microsoft Graph に RESTAPI を送る
> HTTP コネクタを用いて、Microsoft Graph に RESTAPI

- アカウント情報に対して、Microsoft Graph に HTTP コネクタを用いて RESTAPI を行います
  - 「ビルトイン」-> 「HTTP」より、HTTP コネクタを追加します
  - 「アクション」は「HTTP」を選択します
  -  HTTP コネクタのパラメータに Microsoft Graph 宛のパラメータを設定します

|  項目  | パラメータ設定 |
| ---- | ---- |
| 方法 | ``GET`` |
| URI |  Microsoft Graph に送るRESTAPI URI<BR> ``https://graph.microsoft.com/v1.0/users/@{items('For_each')?['Name']}?$select=displayName,userPrincipalName,mail,officeLocation,department,jobTitle``  |
| ヘッダー | Content-Type : ``application/json`` |
| 認証 | マネージド ID |
| マネージド ID | システムマネージド ID |
| 対象ユーザー | ``https://graph.microsoft.com/`` |

- 設定イメージ
<img width="737" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/ed08c34e-93de-47a8-8699-34c7fddc4b56">

