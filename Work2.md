# 演習2. 標準コネクタを用いて、Defender XDR のアドバンスドハンティングクエリーを実行する
> 標準コネクタを用いて、ロジックアプリを作成してみましょう
インシデントをトリガーに、Defender XDR のハンティングクエリーを実行するロジックアプリを実践してみましょう

## 1. ロジックアプリに「Defender ATP」コネクタを追加する
演習 1 で作成したロジックアプリに Defender XDR コネクタを追加します。<BR>
<img width="1061" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/3cd0e8ff-afc1-45d2-a7af-c81b4ef8c30f"><BR>
**「詳細な検索」**を選択します
<img width="510" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/aa2dc48c-ee35-47a5-9fb9-d11b10481d68"><BR>
テナント接続が出てきますので、Entra ID 認証を用いて接続します<BR>
<img width="513" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/75305c0c-f2a5-4485-8d13-a862a2d4dc9d"><BR>
クエリー入力画面が出ればOKです。
<img width="857" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/013fb0b4-c419-4b2c-91bc-be757cc77322"><BR>

## 2. Defender XDR Advanced Hunting Query を用意する
> インシデントをトリガーにハンティングクエリーを実行します
MDE のインシデント検出時にアドバンスハンティングを利用する例です。以下はサンプルなので、他に実行したいクエリーアイデアをお持ちであれば、そちらをご活用下さい！

- MDTM 脆弱性情報から、高危険度の脆弱性があり、セキュリティ更新プログラムがある情報を抽出する
 - 「セキュリティ更新プログラム」があるのだから、パッチ適用しようよ！をアピール
 - CVE 情報ではなく、パッチ適用を推進するための情報を抽出するイメージ

```
DeviceTvmSoftwareVulnerabilities
| where DeviceName contains "(ホスト名)"
| where VulnerabilitySeverityLevel == @"High"
| where RecommendedSecurityUpdate != ""
| distinct RecommendedSecurityUpdate, RecommendedSecurityUpdateId, SoftwareVendor, SoftwareName, SoftwareVersion
```
<img width="746" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/c9c4b020-8406-4eec-a09e-c875810ad700">

## 3. ロジックアプリにクエリーを設定する
> ロジックアプリのフローから Defender XDR Advanced Hunting Query を設定しましょう
ロジックアプリのコネクタにクエリーを設定してみましょう。<BR>
インシデントトリガーでの実行を試す前に、まずは手動でクエリーを投入し、正しく動作するかどうかを確認することをお勧めします。
<img width="715" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/ada8e3dd-b638-4562-b66c-b07bce0a985a"><BR>
ロジックアプリのテストは「過去発生した履歴」を選択することで、再送信が出来ます。<BR>
演習 1 で実行したアラートなどを用いて発砲テストを行って下さい。
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/21870da7-46aa-435a-8652-4d2a21455483)

成功すると、ロジックアプリの結果から Defender XDR のクエリー結果を得られます
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/e26973f7-c4a9-4fac-b13d-945d12a7c8e1)

## 4. クエリー結果を成型する
> ロジックアプリの結果を成型して見やすくする

RESTAPI の処理によって得られた結果は JSON Format による応答で得られます。<BR>
クエリー結果を事後のアクションとしてメール通知、Teams 通知する場合、このままでは既読性が悪く運用に向きません。<BR>
通知し易いように HTML 変換しちゃいましょう!

### 4.1 ロジックアプリのフローに「HTML テーブルの作成」を追加する
ロジックアプリの追加フローボタンを押して、「ビルトイン」-> 「データ操作」から、**「HTML テーブルの作成」を選択します。
<img width="798" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/c728cc74-8fe3-4557-afa0-e510a46b0960">
前のフローで実施したクエリー結果 **Results** を反映させて、保存します。
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/e02f8231-2642-4269-9f96-3763d75823cf)
<BR>
一度、ロジックアプリの履歴から実行してテストして下さい。
クエリーは HTML テーブルに変換されましたか？<BR>
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/59cdd836-a193-4d6d-bf89-052101bc71f6)

### 4.2 HTML テーブルをカスタマイズする
> 美的センスを高めてカスタマイズしましょう！

デフォルトの設定でも自動で HTML テーブルが作成されましたが、要らないフィールドが含まれています。項目名も日本語化しましょう。<BR>
ロジックアプリデザイナーに戻り、「HTML テーブルの作成」をカスタムに変更します。
<img width="849" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/9586da4e-bb16-4033-aae1-5ab574917cb8">
デフォルトで生成した表がこちらになります。

<table>
<thead><tr><th>RecommendedSecurityUpdate</th><th>RecommendedSecurityUpdateId</th><th>SoftwareVendor</th><th>SoftwareName</th><th>SoftwareVersion</th></tr></thead><tbody><tr><td>November 2023 Security Updates</td><td>5032006</td><td>microsoft</td><td>.net_framework</td><td>4.8.1.0</td></tr><tr><td>January 2024 Security Updates</td><td>5033919</td><td>microsoft</td><td>.net_framework</td><td>4.8.1.0</td></tr><tr><td>December 2023 Security Updates</td><td>5033369</td><td>microsoft</td><td>windows_11</td><td>10.0.22000.2538</td></tr><tr><td>November 2023 Security Updates</td><td>5032192</td><td>microsoft</td><td>windows_11</td><td>10.0.22000.2538</td></tr><tr><td>October 2023 Security Updates (Last updated at November 2023)</td><td>5032192</td><td>microsoft</td><td>windows_11</td><td>10.0.22000.2538</td></tr><tr><td>January 2024 Security Updates</td><td>5034121</td><td>microsoft</td><td>windows_11</td><td>10.0.22000.2538</td></tr><tr><td>February 2024 Security Updates</td><td>5034766</td><td>microsoft</td><td>windows_11</td><td>10.0.22000.2538</td></tr><tr><td>March 2024 Security Updates</td><td>5035854</td><td>microsoft</td><td>windows_11</td><td>10.0.22000.2538</td></tr></tbody>
</table>

HTML テーブルの「カスタム」より、各表の項目名を書き換えて、必要な項目だけを抽出してみましょう。
![image](https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/5bdb53c5-c660-48d3-8c7a-815422cd197f)

## 5. メール通知 or Teams 通知
> 最後に通知してみましょう

Defender XDR に Advanced Hunting した結果を通知します。
メールコネクタ or Teams コネクタお好きな方法を選んでください。






