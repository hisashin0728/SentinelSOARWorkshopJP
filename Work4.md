# 演習4. Sentinel インシデントトリガーからエンティティ属性を抽出して、Microsoft Graph API を叩いて Entra ID 情報を取得する
> RESTAPI の接続方法、Microsoft Graph の活用をちゃんレンジしてみましょう

Microsft Sentinel のインシデント検知をトリガーとして、インシデントに含まれるエンティティ情報から Microsoft Graph API を叩いて、Entra ID のユーザー情報を付与してみましょう。自動化するロジックアプリを実践してみましょう。  
構成イメージは以下の通りです<p>

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/03cf1312-856e-4fb6-a34d-0ff898600940)

## 理解 Microsoft Graph を触ってみる
> はじめに Microsoft Graph Explorer を触ってみましょう

- Microsoft Graph については、[公式 Docs](https://learn.microsoft.com/ja-jp/graph/overview) を参照下さい
- [Graph エクスプローラー](https://learn.microsoft.com/ja-jp/graph/graph-explorer/graph-explorer-overview)の使い方も参照下さい

[https://learn.microsoft.com/ja-jp/graph/graph-explorer/graph-explorer-overview](https://developer.microsoft.com/en-us/graph/graph-explorer)https://developer.microsoft.com/en-us/graph/graph-explorer

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/1738f4b2-40cc-4e08-8c88-d48dfab535ad)

- 本演習では[ユーザーの取得](https://learn.microsoft.com/ja-jp/graph/api/user-get?view=graph-rest-1.0&tabs=http)クエリーを用います
  - Docs 情報から、規定値のクエリーだけでは限定的な情報のみ応答することが分かります
    - businessPhones
    - displayName
    - givenName
    - id
    - jobTitle
    - mail
    - mobilePhone
    - officeLocation
    - preferredLanguage
    - surname
    - userPrincipalName
  - 部署名フィールド(department) を付与するにはどうしたら良いでしょうか？
    - [$selectを使用してユーザーの特定のプロパティを取得する](https://learn.microsoft.com/ja-jp/graph/api/user-get?view=graph-rest-1.0&tabs=http#example-3-use-select-to-retrieve-specific-properties-of-a-user)

```
GET https://graph.microsoft.com/v1.0/users/87d349ed-44d7-43e1-9a83-5f2406dee5bd?$select=displayName,givenName,postalCode,identities
```
<img width="797" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/66a9ce25-3710-4a4e-a93b-56ed981a6da6">
