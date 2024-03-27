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
$RequiredScopes = @("Directory.AccessAsUser.All", "Directory.ReadWrite.All") Connect-MgGraph -Scopes $RequiredScopes
Connect-MgGraph -Scopes $RequiredScopes
```

<img width="534" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/e3ac3c5d-60da-4527-8b21-a08f6d053a76"><p>

### 3.2 Powershell スクリプト実行

```powershell:スクリプト実行
<作成したスクリプト名>.ps1
```

<img width="518" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/fc31b31f-80ab-4301-8ba0-a939f487ec61">


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

> 参考
> マネージド ID が削除されると、自動的に Entra ID のアプリケーションからも削除されます

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

