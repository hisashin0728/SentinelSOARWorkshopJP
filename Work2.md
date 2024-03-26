# 演習2. 標準コネクタを用いて、Defender XDR のアドバンスドハンティングクエリーを実行する
> 標準コネクタを用いて、ロジックアプリを作成してみましょう

Defender XDR のハンティングクエリーを実行するロジックアプリを実践してみましょう

## 事前準備 空のロジックアプリを作成する　
- 空のロジックアプリから作成します。Sentinel のオートメーション、もしくはロジックアプリから作成して下さい。
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/99089e58-8f74-4e19-a080-a2626e386c21)
- ロジックアプリが作成されましたら、「繰り返し」を選択します（crond）
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/3548efa0-2404-48dd-95e3-bc2f3e24edab)
- 「繰り返し」の設定のパラメータを変更して、1 時間おきに実行するようにします
<img width="1074" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/6ad559fc-0c9d-4994-85df-80cf786d629a">

## 1. ロジックアプリに「Defender ATP」コネクタを追加する
- ロジックアプリのフロー Defender XDR コネクタを追加します。<BR>
<img width="897" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/28d2e721-4f75-400a-a06a-f994886c8e33"><BR>
- **「詳細な検索」**を選択します
<img width="885" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/dd4b8824-8f0c-4736-bfaf-14d4d22969b1"><BR>
- テナント接続が出てきますので、Entra ID 認証を用いて接続します<BR>
<img width="513" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/75305c0c-f2a5-4485-8d13-a862a2d4dc9d"><BR>
- クエリー入力画面が出ればOKです<BR>
<img width="901" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/9f6f9a6e-56b4-4c84-8555-0da4a676c8b9"><BR>

## 2. Defender XDR Advanced Hunting Query を用意する
> インシデントをトリガーにハンティングクエリーを実行します
定期的に Defender XDR Advanced Hunting Query を実行して通知します。<BR>
以下はサンプルなので、他に実行したいクエリーアイデアをお持ちであれば、そちらをご活用下さい！<BR>

- クエリー例 1 - 脆弱性を Fix するセキュリティ更新プログラムを抽出する
 - MDTM 脆弱性情報から、高危険度の脆弱性があり、セキュリティ更新プログラムがある情報を抽出する
 - 「セキュリティ更新プログラム」があるのだから、パッチ適用しようよ！をアピール
 - CVE 情報ではなく、パッチ適用を推進するための情報を抽出するイメージ

```kql
DeviceTvmSoftwareVulnerabilities
| where DeviceName contains "(ホスト名)"
| where VulnerabilitySeverityLevel == @"High"
| where RecommendedSecurityUpdate != ""
| distinct RecommendedSecurityUpdate, RecommendedSecurityUpdateId, SoftwareVendor, SoftwareName, SoftwareVersion
```
<img width="746" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/c9c4b020-8406-4eec-a09e-c875810ad700">

- クエリー例 2 - 対象サーバーのアセット情報からソフトウェアリストを抽出する
 - MDTM 脆弱性情報から、ソフトウェアインベントリ情報を抽出する
 - サーバーやクライアントを検出した際に、この端末（サーバー）はどのようなものかを判定させる

```kql
DeviceTvmSoftwareInventory
| where DeviceName contains "(ホスト名)"
| distinct SoftwareVendor,SoftwareName,SoftwareVersion
```
<img width="415" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/a2cc2da4-eb3b-4105-845d-1170b0bb7ade">



## 3. ロジックアプリにクエリーを設定する
> ロジックアプリのフローから Defender XDR Advanced Hunting Query を設定しましょう

- ロジックアプリのコネクタにクエリーを設定してみましょう
 - インシデントトリガーでの実行を試す前に、まずは手動でクエリーを投入し、正しく動作するかどうかを確認することをお勧めします
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/bb98515f-12bc-4999-a92a-1e2a9f5c8041)

- 「繰り返し」の場合、ロジックアプリのテストは実行することでテストが容易に出来ます
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/051a56b4-56c7-4030-a6c3-2f48cb9ff371)

- 成功すると、ロジックアプリの結果から Defender XDR のクエリー結果を得られます
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/88d298d7-d22c-4bd6-8ef0-d43fd2715f58)


## 4. クエリー結果を成型する
> ロジックアプリの結果を成型して見やすくする

- RESTAPI の処理によって得られた結果は JSON Format による応答で得られます
 - クエリー結果を事後のアクションとしてメール通知、Teams 通知する場合、このままでは既読性が悪く運用に向きません
 - 通知し易いように HTML 変換しちゃいましょう!

### 4.1 ロジックアプリのフローに「HTML テーブルの作成」を追加する
- ロジックアプリの追加フローボタンを押して、「ビルトイン」-> 「データ操作」から、「**HTML テーブルの作成**」を選択します
<img width="798" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/c728cc74-8fe3-4557-afa0-e510a46b0960"><BR>
- 前のフローで実施したクエリー結果 **Results** を反映させて、保存します。
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/e02f8231-2642-4269-9f96-3763d75823cf)<BR>
- 一度、ロジックアプリの履歴から実行してテストして下さい。
 - クエリーは HTML テーブルに変換されましたか？
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/59cdd836-a193-4d6d-bf89-052101bc71f6)<BR>

### 4.2 HTML テーブルをカスタマイズする
> 美的センスを高めてカスタマイズしましょう！

- デフォルトの設定でも自動で HTML テーブルが作成されましたが、要らないフィールドが含まれています。項目名も日本語化しましょう
- ロジックアプリデザイナーに戻り、「HTML テーブルの作成」をカスタムに変更します
<img width="849" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/9586da4e-bb16-4033-aae1-5ab574917cb8"><BR>
- HTML テーブルの「カスタム」より、各表の項目名を書き換えて、必要な項目だけを抽出してみましょう。
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/5bdb53c5-c660-48d3-8c7a-815422cd197f)

- ロジックアプリの履歴から、イベントを再送信してテストして下さい。成型した表形式に書換られたら成功です！
<img width="851" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/9d61177e-2236-4451-bfe8-f97f60651c76">


## 5. メール通知 or Teams 通知
> 最後に通知してみましょう

- Defender XDR に Advanced Hunting した結果を通知します
 - メールコネクタ or Teams コネクタお好きな方法を選んでください。

### 5.1 メール送信 (Office 365 コネクタ)
- 標準コネクタから「Office 365 Outlook」を選択します
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/13e36620-986a-4c0f-b362-257756c315b6)
- 「メールの送信」を選択します
　- サインインはメール送信元ユーザーで Entra ID 認証を行って下さい 
<img width="870" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/99171718-cddc-4ce4-bd31-570e8e045f50">
- 「メールの送信」から、送信先メールアドレス、メールタイトルなどを設定します
 - メール本文に、前項目で設定した JSON2HTML の BODY を選択して下さい
<img width="1062" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/a471e79f-b14c-4d86-82da-13d4edd964f5">
- テストしてメール通知が行われることを確認して下さい
<img width="1097" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/4d7bf3e0-d802-439d-a0ee-004574070a72">

### 5.2 Microsoft Teams 送信 (Office 365 コネクタ)
-　標準コネクタから「Microsoft Teams」を選択します
<img width="888" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/79b30b8c-e71f-4e1d-a7ab-3a604e78a9e7">
- 「チャットやチャネルにメッセージを投稿する」を選択
　-　Teams に投稿するユーザーで Entra ID 認証を行います 
<img width="877" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/7fba0f8a-11bd-41dc-85d6-fcf89a5f37e3">
- 投稿先の Teams チャネル、メッセージ内容を編集します
 - 本文に、前項目で設定した JSON2HTML の BODY を選択して下さい
<img width="832" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/a8be1e34-7d29-4e25-9536-6a171451c4da">
- テストして Teams チャネル宛に通知が行われることを確認して下さい

## 6. 振り返り
> お疲れさまでした！
- この演習では以下を実践しました。
 - 標準コネクタを用いて、Defender XDR にクエリーを実施する
 - クエリーの結果から JSON を HTML に成型する
 - HTML をメール通知 / Team 通知する

次の章に移ってください！
