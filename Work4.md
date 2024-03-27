# 演習4. Sentinel インシデントトリガーからエンティティ属性を抽出して、Microsoft Graph API を叩いて Entra ID 情報を取得する
> RESTAPI の接続方法、Microsoft Graph の活用を Challenge してみましょう

Microsft Sentinel のインシデント検知をトリガーとして、インシデントに含まれるエンティティ情報から Microsoft Graph API を叩いて、Entra ID のユーザー情報を付与してみましょう。 
構成イメージは以下の通りです<p>

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/03cf1312-856e-4fb6-a34d-0ff898600940)

# Microsoft Graph を触ってみる
> はじめに Microsoft Graph Explorer を触ってみましょう

- Microsoft Graph については、[公式 Docs](https://learn.microsoft.com/ja-jp/graph/overview) を参照下さい
- [Graph エクスプローラー](https://learn.microsoft.com/ja-jp/graph/graph-explorer/graph-explorer-overview)の使い方も参照下さい

[https://learn.microsoft.com/ja-jp/graph/graph-explorer/graph-explorer-overview](https://developer.microsoft.com/en-us/graph/graph-explorer)https://developer.microsoft.com/en-us/graph/graph-explorer

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/1738f4b2-40cc-4e08-8c88-d48dfab535ad)

- 本演習では[ユーザーの取得](https://learn.microsoft.com/ja-jp/graph/api/user-get?view=graph-rest-1.0&tabs=http)クエリーを用います
  - ``id`` もしくは ``UserPrincipalName`` でユーザー情報を取得できます
  - UPN 名を入れただけの規定値クエリーでは限定的な情報のみ応答することが分かります [Docs 情報](https://learn.microsoft.com/ja-jp/graph/api/user-get?view=graph-rest-1.0&tabs=http#example-1-standard-users-request)
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
  - 部署名フィールド(``department``) を付与するにはどうしたら良いでしょうか？
    - [$selectを使用してユーザーの特定のプロパティを取得する](https://learn.microsoft.com/ja-jp/graph/api/user-get?view=graph-rest-1.0&tabs=http#example-3-use-select-to-retrieve-specific-properties-of-a-user)

```
GET https://graph.microsoft.com/v1.0/users/87d349ed-44d7-43e1-9a83-5f2406dee5bd?$select=displayName,givenName,postalCode,identities
```
<img width="797" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/66a9ce25-3710-4a4e-a93b-56ed981a6da6">

- 出来ますね！
  - Microsoft Graph を使いこなすことで、必要となる情報を取得できることが分かりました

# 事前準備
## 1. Sentinel のインシデントトリガーのロジックアプリを作成する　
> これまでの演習と同様に、インシデントトリガーのロジックアプリを作成しましょう
- Sentinel のオートメーションルールから、「インシデントトリガーを使用したプレイブック」を作成します<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/bf2a9e25-4554-4a2d-9168-3854cde38da8)
<img width="423" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/8604d868-6952-4340-9fe4-f3d0a77427d2">

## 2. 作成されたロジックアプリに対して、マネージド ID を有効にする
> ロジックアプリが Microsoft Graph API に認証できるようにマネージド ID を有効にします
- ロジックアプリの ID より、システム割り当て済みマネージド ID を有効にして保存します<p>

<img width="744" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/223a5910-c627-46b3-ae8a-9c8c8713258f">

## 3. マネージド ID に対して、Entra ID で「User.Read.All」権限を付与する
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
  - Azure ポータルから Cloudshell で設定するのがおススメです
  - Azure から Cloudshell を Powershell モードで起動します<p>
<img width="975" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/3ce7badf-46ea-4b0a-8035-19740576c560"><p>
<img width="384" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/d2140532-ca09-4539-8554-13d3f04bed22"><p>

- 実行用の Powershell スクリプトを作ります
> Web 版 VisualStudio Code はマウス右クリックで設定出来ます (例: ファイル保存は 「Ctrl + S」で保存)

```powershell
PS /home/hisashi> code <スクリプト名>.ps1
```    
- 自環境に合わせて以下 Powershell スクリプトを作成して保存します

```powershell
# テナント ID と先ほどメモしたオブジェクト ID を設定
$TenantID="<自環境の Entra ID　Tenant ID>"
$spID="<ロジックアプリの Managed ID>"

# MS Graph の許可を指定、1 つのみ指定
# 複数必要の場合は「マネージド ID にアクセス許可を設定」のコマンドを繰り返し実施
# 今回与える権限は Entra ID のユーザー情報 lookup なので、User.Read.All を設定
$PermissionName = "User.Read.All"

# 事前に MS Graph PowerShell にログイン
Connect-MgGraph -TenantId $TenantID -Scopes Application.Read.All,AppRoleAssignment.ReadWrite.All

# Microsoft Graph のサービスプリンシパルを取得
$GraphServicePrincipal = Get-MgServicePrincipal -Filter "DisplayName eq 'Microsoft Graph'" | Select-Object -first 1

# マネージド ID にアクセス許可を設定
$AppRole = $GraphServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$params = @{
    PrincipalId = $spID
    ResourceId = $GraphServicePrincipal.Id
    AppRoleId = $AppRole.Id
}
New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $spID -BodyParameter $params
```

- 以下手順で作成したスクリプトを実行します

### 3.1 Microsoft Graph への接続

```powershell:MicrosoftGraph接続
$RequiredScopes = @("Directory.AccessAsUser.All", "Directory.ReadWrite.All")
Connect-MgGraph -Scopes $RequiredScopes
```

<img width="969" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/cfea7136-07ad-4de7-bbd9-8b38d9a79956"><p>

> Web でコード認証 / ユーザー認証が行われます

### 3.2 Powershell スクリプト実行

```powershell:スクリプト実行
<作成したスクリプト名>.ps1
```

<img width="1105" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/7eb9e737-8552-4883-b6cc-d3700f93bbce">

### 3.3 Microsoft Graph から切断
```powershell:MgGraph切断
Disconnect-MgGraph
```

<img width="515" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/eed4b9cf-354a-4a15-97a1-a451078e8cb1">

### 3.4 Entra ID からエンタープライズアプリで確認する
- 無事スクリプトが成功すれば、Entra ID から確認が出来ます
- 「Entra ID」 -> 「エンタープライズアプリケーション」より、対象のマネージド ID を確認しましょう
- 「アクセス許可」から、Microsoft Graph に対して、``User.Read.All`` が付与されているかどうかを確認します

<img width="316" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/006b183e-4545-4a4b-a707-6f3439c73841"><p>
<img width="1102" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/27cf501f-ad00-41cf-9b93-182fd67d43b4">

> 参考
> マネージド ID が削除されると、自動的に Entra ID のアプリケーションからも削除されます

# ロジックアプリの編集
## 1. Sentinel エンティティ情報を格納する
> アカウントタイプのエンティティのみに抽出する

- 後段の Microsoft.Graph に連携するため、ここでは ``Account`` 型のエンティティを抽出します
- 標準コネクタ「Microsoft Sentinel」->「エンティティ - アカウントを取得」を接続し、Sentinel インシデントトリガーの情報から「エンティティ」の一覧をセットします

<img width="876" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/8de88f80-8679-4425-8a9e-3b3066a2c653">

## 2. For Each 処理で JSON アレイの内容毎に処理する
> For Each 処理でエンティティのアレイを分解しましょう

 - [演習3 のプロセス](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work3.md#2-json-%E3%82%A2%E3%83%AC%E3%82%A4%E3%81%AE%E6%83%85%E5%A0%B1%E3%82%92-for-each-%E3%83%AB%E3%83%BC%E3%83%97%E3%81%A7%E5%88%86%E8%A7%A3%E3%81%97%E3%81%A6%E5%87%A6%E7%90%86%E3%81%95%E3%81%9B%E3%82%8B)同様に、エンティティで抽出した情報は JSON アレイになっているため（※複数のアカウントが検出する可能性がある）、For Each 処理を行います

<img width="808" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/02a2af7a-a867-4245-832a-5f8ee9a10a1a">

- エンティティ情報の JSON アレイ分解後の JSON 情報は以下のようになります
  - ADDS より連携されたユーザー情報例

```json
  {
    "accountName": "中田 尚志",
    "ntDomain": "ad.azurecsa",
    "upnSuffix": "ad.azurecsa.net",
    "sid": "S-1-5-21-3013741847-3473879986-3602106788-1108",
    "aadTenantId": "63001692-0de3-4bc0-9815-4e9a04aea825",
    "aadUserId": "e47aff12-1da8-45cb-95df-fc5c3db05bee",
    "isDomainJoined": true,
    "displayName": "中田 尚志",
    "dnsDomain": "ad.azurecsa.net",
    "additionalData": {
      "Sources": "[\"AzureActiveDirectory\"]",
      "GivenName": "尚志",
      "IsDeleted": "False",
      "IsEnabled": "True",
      "Surname": "中田",
      "TransitiveDirectoryRoles": "[\"Application Administrator\",\"Cloud Application Administrator\"]",
      "UserType": "Member",
      "UpnName": "hnakada@ad.azurecsa.net",
      "SyncFromAad": "True",
      "AliasNames": "[\"hisas\",\"hnakada\",\"中田 尚志\"]",
      "UserPrincipalName": "hnakada@ad.azurecsa.net",
      "MailAddress": "hnakada@ad.azurecsa.net",
      "OnPremisesDistinguishedName": "CN=中田 尚志,CN=Users,DC=ad,DC=azurecsa,DC=net",
      "OnPremisesSamAccountName": "hnakada",
      "AccountName": "hnakada",
      "DomainName": "ad.azurecsa"
    },
    "friendlyName": "中田 尚志",
    "Type": "account",
    "Name": "中田 尚志"
  }
```

## 3. Microsoft Graph に RESTAPI を送る
> HTTP コネクタを用いて、Microsoft Graph に RESTAPI を送ります

- アカウント情報に対して、Microsoft Graph に HTTP コネクタを用いて RESTAPI を行います
  - 「ビルトイン」-> 「HTTP」より、HTTP コネクタを追加します
  - 「アクション」は「HTTP」を選択します
  -  HTTP コネクタのパラメータに Microsoft Graph 宛のパラメータを設定します
-  前述で For Each で分解された情報から ``UserPrincipalName`` を JSON で設定します
  - 上記の JSON 構造から JSON Path を考えてみましょう
  - ``@{items('For_each')?['additionalData']?['UserPrincipalName']}`` になります

|  項目  | パラメータ設定 |
| ---- | ---- |
| 方法 | ``GET`` |
| URI |  Microsoft Graph に送るRESTAPI URI<BR> ``https://graph.microsoft.com/v1.0/users/@{items('For_each')?['additionalData']?['UserPrincipalName']}?$select=displayName,userPrincipalName,mail,officeLocation,department,jobTitle``  |
| ヘッダー | Content-Type : ``application/json`` |
| 認証 | マネージド ID |
| マネージド ID | システムマネージド ID |
| 対象ユーザー | ``https://graph.microsoft.com/`` |

- JSON Path は「動的なコンテンツ」から設定することが出来ないため、「ロジックアプリコードビュー」から直接編集します
> 以下画面を参考に、コードビューから HTTP コネクタの URI を直接編集して下さい

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/0151a763-6fd5-4c56-b897-c41c57fee0de)

- 設定イメージ
> デザイナーから編集は出来ませんが、コードビューで直編集した　JSON パスが反映されます
<img width="800" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/aa841a12-e6f0-4b5f-9cb1-db2df2419453">

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/5056c104-accf-45de-ab9f-5861114ced2e)

- テストしてみましょう
> アカウント情報が出力されているインシデントがあれば、一度試してみましょう
- ``UserPrincipalName`` は正しく拾えていますか？
- マネージド ID 経由で Graph API を叩いて、レスポンスが返ってきますか？
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/8ba20a6a-485a-4ee2-a1b9-62d9da6e3e4a)

## 4. 後処理で使いやすいように JSON 正規化 (Parse JSON) を行う
> あと少し！頑張って！
- Graph API にクエリーした応答結果を、後工程で処理しやすいように JSON 正規化します
  - JSON 正規化すると、「動的なコンテンツ」から選択するだけで反映出来るようになります
- アクションの追加より、「ビルトイン」-> 「データ操作」 -> 「JSON の解析」を選択します
  -　コンテンツには、前ステップで得られた「本文(BODY)」を選択します
- スキーマの欄には、前ステップで得られたペイロード情報をサンプルとして貼り付けてスキーマ生成することが出来ますが、今回は以下を張り付けて下さい

```json
{
    "properties": {
        "@@odata.context": {
            "type": "string"
        },
        "department": {
            "type": "string"
        },
        "displayName": {
            "type": "string"
        },
        "jobTitle": {
            "type": "string"
        },
        "mail": {
            "type": "string"
        },
        "mobilePhone": {
            "type": [
                "string",
                "null"
            ]
        },
        "officeLocation": {
            "type": "string"
        },
        "userPrincipalName": {
            "type": "string"
        }
    },
    "type": "object"
}
```
> JSON の正規化モジュールでは、"type" : "string" などの固定値で設定したフィールドだけ、後段のプロセスで動的なコンテンツで読み出すことが出来ます。
> 上記例での "mobilePhone" のように応答値が NULL になる可能性がある場合、type 指定を複数設定しないとエラーになりますが、後段のプロセスでは読み出すことが出来なくなります。

- この処理が行われると、以下のように処理されます

<img width="876" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/805e5679-2d08-4967-8287-84820fcb9dbc">

## 5. 通知を行う
> あとは煮るなり焼くなり、何でもどうぞ！
- Microsoft Graph に lookup しましたので、得られた情報を通知しましょう
  - メール通知
<img width="700" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/a847f1b2-08fb-4b84-93d8-9333e07f4837">
  - Teams 通知
  - Microsoft Sentinel のインシデント情報にコメントで付与する 
<img width="608" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/c5bc72c5-bb1a-47b8-888a-df39578cfa79">

# 振り返り
> お疲れさまでした！
- 本演習では以下を実践しました
  - Graph API を理解する
  - Graph API への接続方法を理解する (マネージド ID 経由)
  - ロジックアプリで Graph API の叩き方、お作法を理解する
