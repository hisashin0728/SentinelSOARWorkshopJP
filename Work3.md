# 演習3. Sentinel インシデントトリガーからエンティティ属性を抽出して、Defender XDR のアドバンスドハンティングクエリーを実行する
> 演習2 に加えて、Sentinel のインシデントに含まれるエンティティ情報を利用して Advanced Hunting を実行してみましょう

## 事前準備 Sentinel のインシデントトリガーのロジックアプリを作成する　
> インシデントトリガーのロジックアプリを作成する
- Sentinel のオートメーションルールから、「インシデントトリガーを使用したプレイブック」を作成します。
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/bf2a9e25-4554-4a2d-9168-3854cde38da8)
<img width="423" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/8604d868-6952-4340-9fe4-f3d0a77427d2">

## 1. Sentinel インシデントトリガーから得られるエンティティ情報を格納する
> JSON アレイを理解する

- Sentinel インシデントトリガーから得られるエンティティ情報は JSON アレイ型になっています。
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/00773eba-13f8-4ddd-948d-b66a856389e4)
- 例えば、MDE for Linux が EICAR ファイルを検知した際のエンティティ情報は以下のようになります
 - ``kind`` が ``Host`` 属性となるホスト情報（例：これを使いたい）
 - ``kind`` が ``File`` 属性となるファイル名情報
 - ``kind`` が ``FileHash`` 属性となるファイルハッシュ情報

```
[
  {
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-xxx-xxx/providers/Microsoft.OperationalInsights/workspaces/log-hubhnakada-JapanEast/providers/Microsoft.SecurityInsights/Entities/",
    "type": "Microsoft.SecurityInsights/Entities",
    "kind": "Host",
    "properties": {
      "dnsDomain": "rglnoiucbtnuzarcriubjs2azc.lx.internal.cloudapp.net",
      "hostName": "vmlinuxcustomlog",
      "osFamily": "Unknown",
      "osVersion": "7.9",
      "additionalData": {
        "MdatpDeviceId": "95ffd3240bde37ef347cf836db74b13f85cbf185",
        "DeviceDnsName": "vmlinuxcustomlog.rglnoiucbtnuzarcriubjs2azc.lx.internal.cloudapp.net",
        "RiskScore": "informational",
        "HealthStatus": "active",
        "FirstSeenDateTime": "2023-09-04T03:29:26.736412+00:00",
        "RbacGroupName": "Servers",
        "DefenderAvStatus": "notSupported",
        "OnboardingStatus": "onboarded"
      },
      "friendlyName": "vmlinuxcustomlog"
    }
  },
  {
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-xxx-xxx/providers/Microsoft.OperationalInsights/workspaces/log-hubhnakada-JapanEast/providers/Microsoft.SecurityInsights/Entities/",
    "type": "Microsoft.SecurityInsights/Entities",
    "kind": "File",
    "properties": {
      "directory": "/home/azureuser",
      "fileName": "eicar.com",
      "fileHashEntityIds": [
        "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-xxx-xxx/providers/Microsoft.OperationalInsights/workspaces/log-hubhnakada-JapanEast/providers/Microsoft.SecurityInsights/entities/",
        "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-xxx-xxx/providers/Microsoft.OperationalInsights/workspaces/log-hubhnakada-JapanEast/providers/Microsoft.SecurityInsights/entities/"
      ],
      "additionalData": {
        "MdatpDeviceId": "95ffd3240bde37ef347cf836db74b13f85cbf185",
        "DetectionStatus": "prevented"
      },
      "friendlyName": "eicar.com"
    }
  },
  {
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-xxx-xxx/providers/Microsoft.OperationalInsights/workspaces/log-hubhnakada-JapanEast/providers/Microsoft.SecurityInsights/Entities/",
    "type": "Microsoft.SecurityInsights/Entities",
    "kind": "FileHash",
    "properties": {
      "hashValue": "3395856ce81f2b7382dee72602f798b642f14140",
      "algorithm": "SHA1",
      "friendlyName": "3395856ce81f2b7382dee72602f798b642f14140(SHA1)"
    }
  }
]
```
- 後述のフローに用いるため、エンティティの JSON アレイ を変数として用います。
 - 「ビルトイン」-> 「変数」より、「変数を初期化する」を選択して、JSON アレイ型（種類：アレイ）を選んで、Sentinel インシデントトリガーのエンティティ情報を格納します
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/f6f39fd9-76f9-4049-95f9-67c0c0d332ae)<BR>
 - 設定例
<img width="519" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/50334553-bc6f-4eb3-b5c7-2742e11d15d5"><BR>
 - インシデントが発生する環境をお持ちの方は、保存してインシデントをトリガーさせると、JSON アレイに格納されることが確認出来ます
<img width="486" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/cc371ce8-da64-48b2-8bc6-31b233e1e30b"><BR>

## 2. JSON アレイの情報を For Each ループで分解して処理させる


