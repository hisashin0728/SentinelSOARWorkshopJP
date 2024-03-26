# 演習3. Sentinel インシデントトリガーからエンティティ属性を抽出して、Defender XDR のアドバンスドハンティングクエリーを実行する
> 演習2 に加えて、Sentinel のインシデントに含まれるエンティティ情報を利用して Advanced Hunting を実行してみましょう

Microsft Sentinel のインシデント検知をトリガーとして、インシデントに含まれるエンティティ情報から Defender XDR へクエリー分析を自動化するロジックアプリを実践してみましょう。
構成イメージは以下の通りです<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/9a855ed7-8337-4523-8a86-b5cb4426e5dc)


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
- For Each 処理は JSON アレイを入れることで、JSON フィールド毎に分解されます。先に設定した JSON アレイ変数を設定しましょう
  - 先に設定した情報から、「**ホスト**」を選択します<p>
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

## 3. For Each ループ内で Defender ATP コネクタを用いて Advanced Hunting クエリーを設定する
> 演習2と同じ内容で Advanced Hunting Query を設定しましょう
- 目的となる JSON 情報に分解出来たので、Defender ATP コネクタを接続して、Advanced Hunting Query を設定します
  - ロジックアプリのフロー Defender XDR コネクタを追加
  - 「**詳細な検索**」を選択
  - テナント接続が出てきますので、Entra ID 認証を用いて接続
- Hunting Query は演習2 と同様なものか、ホスト情報を条件に出来るクエリーを設定して下さい

### クエリ例1 - 脆弱性を Fix するセキュリティ更新プログラムを抽出する

```kql
DeviceTvmSoftwareVulnerabilities
| where DeviceName contains "(ホスト名)"
| where VulnerabilitySeverityLevel == @"High"
| where RecommendedSecurityUpdate != ""
| distinct RecommendedSecurityUpdate, RecommendedSecurityUpdateId, SoftwareVendor, SoftwareName, SoftwareVersion
```
### クエリ例2 - 対象サーバーのアセット情報からソフトウェアリストを抽出する

```kql
DeviceTvmSoftwareInventory
| where DeviceName contains "(ホスト名)"
| distinct SoftwareVendor,SoftwareName,SoftwareVersion
```
- ハンティングクエリーを設定する際に、**For Each 処理によって JSON アレイから分解された**JSONフィールドの情報をマッピングします
  - ロジックアプリは賢いので、For Each 処理前のプロセスである **エンティティ - ホストを取得「ホスト ホスト名」**を選択した場合でも、自動的に ``items('For_each')?['HostName']`` に変換してくれます！<p>

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/67ee66f9-d05d-4457-a879-4481700fbdf1)

- For Each 処理後のエンティティ抽出は式から JSON 起票で設定することも可能です。
  - JSON の設定イメージは以下図を参考にしてください
  - https://jpazinteg.github.io/blog/LogicApps/how-to-treat-json-in-logicApps/

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/c730b3dc-6d8d-4715-b802-b18590d3fbf1)

- クエリーが設定出来ましたら、過去のトリガーを再生してテストを行って下さい
  - エンティティのホスト情報が反映されて、Advanced Hunting Query 結果が返ってくれば成功です！

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/2109ab23-902a-40cb-ac83-1c7f3c6838b1)

## 4. [オプション] JSON2HTML 成型 / メール通知 / Teams 通知
> JSON -> HTML 変換、メール通知 / Teams 通知を実践してみましょう

- 演習2 と同様に、得られた結果を通知しましょう!!
  - [JSON 結果を HTML に変換する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work2.md#4-%E3%82%AF%E3%82%A8%E3%83%AA%E3%83%BC%E7%B5%90%E6%9E%9C%E3%82%92%E6%88%90%E5%9E%8B%E3%81%99%E3%82%8B)
  - [O365 コネクタを用いてメール通知](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work2.md#51-%E3%83%A1%E3%83%BC%E3%83%AB%E9%80%81%E4%BF%A1-office-365-%E3%82%B3%E3%83%8D%E3%82%AF%E3%82%BF)
  - [Microsoft Teams コネクタを用いてチャネルに投稿](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work2.md#52-microsoft-teams-%E9%80%81%E4%BF%A1-office-365-%E3%82%B3%E3%83%8D%E3%82%AF%E3%82%BF)

## 5. 振り返り
> お疲れ様でした！

- この演習では以下を実践していただきました
  - Microsoft Sentinel コネクタのインシデント情報から JSON アレイを抽出する
  - JSON アレイを処理するために For Each 処理を用いる
  - For Each 処理で分解された個々の JSON 情報を元に Defender ATP コネクタを用いてクエリーを実行する
  - [オプション] 得られた結果を成型して外部通知する
- まだまだあるよ！次の演習もやってみよう！
