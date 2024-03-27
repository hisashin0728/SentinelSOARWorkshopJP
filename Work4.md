# 演習4. Sentinel インシデントトリガーからエンティティ属性を抽出して、Microsoft Graph API を叩いて Entra ID 情報を取得する
> RESTAPI の接続方法、Microsoft Graph の活用をちゃんレンジしてみましょう

Microsft Sentinel のインシデント検知をトリガーとして、インシデントに含まれるエンティティ情報から Microsoft Graph API を叩いて、Entra ID のユーザー情報を付与してみましょう。自動化するロジックアプリを実践してみましょう。  
構成イメージは以下の通りです<p>

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/03cf1312-856e-4fb6-a34d-0ff898600940)
