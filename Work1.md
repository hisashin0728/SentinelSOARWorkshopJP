# 演習1. Sentinel インシデントトリガー時にどのような情報が連携されるのかを理解する
> Sentinel インシデントトリガーのロジックアプリを作りましょう
まずは Sentinel インシデント発生時にどのような情報が連携されるのかを理解しましょう

## 1. Sentinel オートメーションからプレイブックを作成する
Microsoft Sentinel 画面から「オートメーション」を選択し、「インシデント トリガー」を使用したプレイブックを作成します.
<img width="552" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/12803197-8668-4e7b-b6a8-731551420149">

> インシデントトリガー/アラートトリガー/エンティティトリガーの三種類はそれぞれの活用方法がありますが、まずはインシデントトリガーのプレイブックを選択してください。

サブスクリプション/リソースグループ/プレイブック名を記入し、作成を行います。
> 「リソースグループ」は Sentinel ワークスペースとは分けて、個別のリソースグループを作成することをお勧めします（リソースグループ毎消すことが多くなります）
> プレイブック名はロジックアプリの名前になります。ロジックアプリは数が増えるので、あらかじめ命名規則を定義しておいてください。

参考: 命名規則例
logic-[目的]-[リージョン]<BR>
``logic-Sentinel-Workshop1-JE``

<img width="339" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/3b254c61-12f0-4291-b554-a698649689dc">

無事作成出来ると、「Microsoft Sentinel インシデント」をトリガーにしたロジックアプリが作成されます。
<img width="667" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/7f1c07b3-be07-4c69-afb2-8f65d6808600">

## 2. オートメーションルールを作成し、ロジックアプリを関連付ける
> Sentinel インシデントから作成したロジックアプリを紐づける
Sentinel 画面に戻り、「オートメーション」から「オートメーションルール」を作成します。
<img width="416" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/6683df6b-4487-4535-91d9-f76921b8bf89">
<BR>
「新しいオートメーション ルールの作成」より、``Microsoft Defender for Cloud`` のデータソースからのインシデント作成時に検知するように設定します。
> 条件として、どのような場合にロジックアプリに紐づけが出来るのかをチェックして下さい
> インシデントプロパイダー、製品名、タグ、タイトルなど、どのような種類がありますか？
<img width="417" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/de35d6af-bd85-41b8-8b87-fcff677b2035">

作成したロジックアプリを選ぼうとすると、グレーになっていて選択が出来ませんか？<BR>
<img width="595" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/0e794fcb-08a7-47ba-986b-bc2568753d5c">
<br>
その場合は、下に表示されている「プレイブックのアクセス許可を管理」するリンクから、作成したリソースグループを Sentinel でアクセス出来るように権限を付与してください。反映に少し時間がかかる点に注意して下さい。
<img width="961" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/e1129fb3-20de-4764-92a1-75f587110ade">
既に多数のプレイブックを作成している場合は、ルール順番に注意して下さい。
<img width="598" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/4dd91b5a-8f8a-4b37-98cb-6f4d352a32ff">

## 3. Microsoft Defender for Cloud (MDC) からサンプルアラートを発砲してみる
> MDC のサンプルアラートより、ロジックアプリが起動されるかを確認する
MDC のサンプルアラートから、サンプルアラートを作成します。<BR>
MDC のアラートを大量に発生させないように、``Resource Manager`` のみに絞るなど注意して下さい。
<img width="982" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/4de9401d-0ce8-42ea-bdc8-da81321886d3">

## 4. ロジックアプリを確認する
> トリガーで発砲したロジックアプリを確認しましょう
ロジックアプリの実行履歴から、実行履歴を確認します。
<img width="1068" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/81bf8c11-8235-49e4-8f41-5dd778ea63fb">
ロジックアプリの実行履歴から、入力と出力を確認してみましょう。
<img width="1102" alt="image" src="https://github.com/hisashin0728/SentinelSOARWorkshopJP/assets/55295601/e6b9720d-2663-4943-bcd1-a3319a829d3b">

## 5. 確認ポイント
> Sentinel のインシデントからどのような情報が出力されましたか？

- インシデント名はどのフィールドに含まれていますか？
- インシデントの詳細は？
- 重要度は？
- 他にどのような情報が含まれていますか？
