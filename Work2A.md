# 演習2A. 標準コネクタを用いて、Azure Resource Graph (ARG) に対して定期的に監視クエリーを実行する
> 標準コネクタを用いて、ロジックアプリを作成してみましょう

Azure Resource Graph の情報を監視するため、Azure Monitor 標準コネクタを用いてクエリーを実行するロジックアプリを実践してみましょう
構成イメージは以下の通りです。<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/bad8216d-70a7-4878-b1a3-fcb99f1157e0)

# 事前準備
## Log Analytics ワークスペースを用意する
- Azure Resource Graph へのアクセスは幾つかの方法が有りますが、本演習では最も簡単な Log Analytics ワークスペースから
 - [参考情報 リソース変更を検知するアラートを作成する (Japan Azure Monitoring Support Blog)](https://jpazmon-integ.github.io/blog/ame/HowToResourceChangeAlert/)
 - [クイックスタート: Azure Resource Graph と Log Analytics を使用してアラートを作成する](https://learn.microsoft.com/ja-jp/azure/governance/resource-graph/alerts-query-quickstart?tabs=azure-resource-graph)

## 空のロジックアプリを作成する　
- 空のロジックアプリから作成します。Sentinel のオートメーション、もしくはロジックアプリから作成して下さい。
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/99089e58-8f74-4e19-a080-a2626e386c21)
- ロジックアプリが作成されましたら、「繰り返し」を選択します（crond）
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/3548efa0-2404-48dd-95e3-bc2f3e24edab)
- 「繰り返し」の設定のパラメータを変更して、1 時間おきに実行するようにします
<img width="1074" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/6ad559fc-0c9d-4994-85df-80cf786d629a">

# ロジックアプリの編集
## 1. ロジックアプリに「Azure Monitor ログ」コネクタを追加する
- ロジックアプリのフローに「Azure Monitor ログ」コネクタを追加します。<BR>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/9bd3c3d4-0176-4ef6-a13f-4b24b4f35e17)<BR>

- 「**クエリーを実行して結果を一覧表示する**」を選択します
> V2 (プレビュー) でも同じ結果が得られます
> 「クエリーを実行して結果を視覚化する」を選択すると、``summarize`` 機能を用いてグラフ生成が出来ます

<img width="885" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/dd4b8824-8f0c-4736-bfaf-14d4d22969b1"><BR>

- テナント接続が出てきますので、Entra ID 認証を用いて接続します<BR>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/45376cb2-d871-4b65-9bcb-bb6e86e1ee28)<BR>

- クエリー入力画面が出ればOKです<BR>
> クエリー欄に実際に KQL を入れて結果を得ることが出来るようになりました！

![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/598e7511-c529-4c48-ab7a-b1f11ded4cdc)<BR>

## 2. Azure Resource Graph に対して実行するクエリーを用意する
> ARG に対してクエリーを実行します
定期的に Azure Resource Graph に対してクエリーを実行して通知することを検討します。<BR>
以下はサンプルなので、他に実行したいクエリーアイデアをお持ちであれば、そちらをご活用下さい！<BR>

### クエリー例 1 - Azure VM のサイズ毎の台数をレポートする
 - Azure VM の設定サイズと台数をクエリーでレポートする

```kql
arg("").resources
| where type =~ "Microsoft.Compute/VirtualMachines"
| summarize count() by tostring(properties.hardwareProfile.vmSize)
| sort by count_ desc 
```
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/ec979e4f-f91c-40eb-88fb-08b3b4b35baf)


### クエリー例 2 - Azure VM の OS 種別毎レポート
 - Azure VM の設定サイズと台数をクエリーでレポートする

```kql
arg("").resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| summarize count() by tostring(properties.storageProfile.osDisk.osType)
```
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/2725096b-22bf-45dc-a9b2-26d8a81082d2)

### クエリー例 3 - 未使用のまま放置されている Public IP アドレス (PIP)
 - 課金かかっているけど放置されている PIP の洗い出し

```kql
arg("").resources
| where type =~ "Microsoft.Network/publicipaddresses"
| where isnull(properties.ipConfiguration)
```
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/13a20a0b-e35b-45ab-88a5-4b2e2c85ebda)

## 3. ロジックアプリにクエリーを設定する
> ロジックアプリのフローから Log Analytics ワークスペースに対してクエリーを設定しましょう

- ロジックアプリのコネクタにクエリーを設定してみましょう
- Azure Monitor クエリーのコネクタに対してクエリーを設定します。<BR>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/c0c3a51d-5514-40f5-92e7-e51e2a30f0a1)<BR>

- 「繰り返し」の場合、ロジックアプリのテストは実行することでテストが容易に出来ます<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/653369ec-4cd7-4ae6-97be-f923bb9a6630)<BR>

- 成功すると、ロジックアプリの結果から クエリー結果を得られます<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/ab2b96fd-a03c-467a-ba15-11ff1e945518)<BR>


## 4. クエリー結果を成型する
> ロジックアプリの結果を成型して見やすくする

- RESTAPI の処理によって得られた結果は JSON フォーマットで結果を得ることが出来ます
 - クエリー結果を事後のアクションとしてメール通知、Teams 通知する場合、このままでは既読性が悪く運用に向きません
 - 通知し易いように HTML 変換しちゃいましょう!

### 4.1 ロジックアプリのフローに「HTML テーブルの作成」を追加する
- ロジックアプリの追加フローボタンを押して、「ビルトイン」-> 「データ操作」から、「**HTML テーブルの作成**」を選択します
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/26c9e994-e7b0-411e-9eee-9b7c50fcd48e)<BR>

- 前のフローで実施したクエリー結果 **value** を反映させて、保存します。
> value - 項目 ではないので注意して下さい！
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/916c1d45-994b-4560-bd16-313d38e1b647)<BR>
- 一度、ロジックアプリの履歴から実行してテストして下さい。
 - クエリーは HTML テーブルに変換されましたか？
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/57d8eedc-f91b-495a-bee0-6168aac33395)<BR>

### 4.2 HTML テーブルをカスタマイズする
> 美的センスを高めてカスタマイズしましょう！

- デフォルトの設定でも自動で HTML テーブルが作成されましたが、要らないフィールドが含まれていませんか？
 - 表の項目名、せっかくなので日本語化してみましょう
- ロジックアプリデザイナーに戻り、「HTML テーブルの作成」をカスタムに変更します<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/ff862d62-4f3f-4320-951b-28f5af02e9d1)<BR>
- HTML テーブルの「カスタム」より、各表の項目名を書き換えて、必要な項目だけを抽出してみましょう<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/e63b5e60-f5b4-4193-b8ef-51ba50e4b4d9)<BR>

- ロジックアプリの履歴から、イベントを再送信してテストして下さい。成型した表形式に書換られたら成功です！<p>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/0ee357cf-4687-4ef3-91e8-985f7e0f604a)

## 5. メール通知 or Teams 通知
> 最後に通知してみましょう

- Log Analytics ワークスペースから Azure Resource Graph にクエリー結果を通知します
 - メールコネクタ or Teams コネクタお好きな方法を選んでください。

### 5.1 メール送信 (Office 365 コネクタ)
- 標準コネクタから「Office 365 Outlook」を選択します
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/13e36620-986a-4c0f-b362-257756c315b6)
- 「メールの送信」を選択します
　- サインインはメール送信元ユーザーで Entra ID 認証を行って下さい 
<img width="870" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/99171718-cddc-4ce4-bd31-570e8e045f50"><p>
- 「メールの送信」から、送信先メールアドレス、メールタイトルなどを設定します
 - メール本文に、前項目で設定した JSON2HTML の BODY を選択して下さい
<img width="1062" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/a471e79f-b14c-4d86-82da-13d4edd964f5"><p>
- テストしてメール通知が行われることを確認して下さい
<img width="1097" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/4d7bf3e0-d802-439d-a0ee-004574070a72"><p>

### 5.2 Microsoft Teams 送信 (Office 365 コネクタ)
-　標準コネクタから「Microsoft Teams」を選択します
<img width="888" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/79b30b8c-e71f-4e1d-a7ab-3a604e78a9e7"><p>
- 「チャットやチャネルにメッセージを投稿する」を選択
　-　Teams に投稿するユーザーで Entra ID 認証を行います 
<img width="877" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/7fba0f8a-11bd-41dc-85d6-fcf89a5f37e3"><p>
- 投稿先の Teams チャネル、メッセージ内容を編集します
 - 本文に、前項目で設定した JSON2HTML の BODY を選択して下さい
<img width="832" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/a8be1e34-7d29-4e25-9536-6a171451c4da"><p>
- テストして Teams チャネル宛に通知が行われることを確認して下さい

# 振り返り
> お疲れさまでした！
- この演習では以下を実践しました。
  - 標準コネクタを用いて、Defender XDR にクエリーを実施する
  - クエリーの結果から JSON を HTML に成型する
  - HTML をメール通知 / Team 通知する

次の[演習 Sentinel インシデントトリガーからエンティティ属性を抽出して、Defender XDR のアドバンスドハンティングクエリーを実行する](https://github.com/hisashin0728/SentinelSOARWorkshopJP/blob/main/Work3.md)もやってみましょう！
