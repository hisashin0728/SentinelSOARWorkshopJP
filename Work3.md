# 演習3. Sentinel インシデントトリガーからエンティティ属性を抽出して、Defender XDR のアドバンスドハンティングクエリーを実行する
> 演習2 に加えて、Sentinel のインシデントに含まれるエンティティ情報を利用して Advanced Hunting を実行してみましょう

## 事前準備 Sentinel のインシデントトリガーのロジックアプリを作成する　
> インシデントトリガーのロジックアプリを作成する
- Sentinel のオートメーションルールから、「インシデントトリガーを使用したプレイブック」を作成します<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/bf2a9e25-4554-4a2d-9168-3854cde38da8)
<img width="423" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/8604d868-6952-4340-9fe4-f3d0a77427d2">

## 1. Sentinel インシデントトリガーから得られるエンティティ情報を格納する
> JSON アレイを理解する

- Sentinel インシデントトリガーから得られるエンティティ情報は JSON アレイ型になっています<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/00773eba-13f8-4ddd-948d-b66a856389e4)
- 例えば、MDE for Linux が EICAR ファイルを検知した際のエンティティ情報は以下のようになります
  - ``kind`` が ``Host`` 属性となるホスト情報
  - ``kind`` が ``File`` 属性となるファイル名情報
  - ``kind`` が ``FileHash`` 属性となるファイルハッシュ情報
- ここでは、後述の Advanced Hunting に使うデータとして、ホスト情報を抽出する想定としています

```json
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
- エンティティ情報からホスト情報を抽出するため、Sentinel 標準コネクタ「エンティティ - ホストを取得」を用います<p>
<img width="286" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/c8f55b78-c27a-4008-ba1b-8e0c29642c2e"><BR>
- コネクタが追加されましたら、トリガーとなった Sentinel の「エンティティ」リストを適用します<p>
<img width="675" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/4e7fb35f-f49b-4eb6-8338-8b9788659c18"><BR>
- このコネクタを用いることで、エンティティの JSON アレイに含まれる情報のうち、``kind`` = ``Host`` 条件のものだけを抽出出来たことが分かります<p>
<img width="491" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/3e05691d-ad16-418a-a084-3ab13fa1908e"><BR>

```json
[
  {
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
    "friendlyName": "vmlinuxcustomlog",
    "Type": "host"
  }
  {
    "dnsDomain": "(以後省略・・・）",
    },
    "friendlyName": "vmlinuxcustomlog",
    "Type": "host"
  }
]
```

## 2. JSON アレイの情報を For Each ループで分解して処理させる
> JSON アレイのエンティティ情報をループ処理させる
- 前プロセスで Host 条件のエンティティ情報だけ抽出が出来ましたが、エンティティには複数の Host 情報が含まれる可能性があるため JSON アレイ型になっています
- JSON アレイの情報をループ処理させるために For Each ループを用います
 - 「ビルトイン」-> 「制御」-> 「For Each」を選択して下さい。
- For Each 処理は JSON アレイを入れることで、JSON フィールド毎に分解されます。先に設定した JSON アレイ変数を設定しましょう<p>
<img width="559" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/30c9258f-38f9-498c-93a7-aac2979bced3"><p>
- For Each 処理をすることで、前述のJSONアレイは分解され、JSON フィールド毎に処理されます（アレイが外れます）

```json
  {
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
    "friendlyName": "vmlinuxcustomlog",
    "Type": "host"
  }
```

