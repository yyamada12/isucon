# さくらのクラウドでの環境構築

## クーポンコードの適用方法

https://manual.sakura.ad.jp/cloud/payment/coupon.html

1. [ [さくらのクラウド コントロールパネル](https://secure.sakura.ad.jp/cloud/) ]
2. [さくらのクラウド(IaaS)]をクリック
3. [オプション]をクリック
4. 左メニューの[クーポン]をクリック
5. 右上の[追加]をクリック
6. クーポンIDを入力し、[追加]をクリック

## 構築方法

https://knowledge.sakura.ad.jp/31520/
- サーバーの追加から追加したいスペックで追加する
- cloud-init設定のところで、cloud init の cfg の内容をコピペ
- ネットワークはデフォルトで100Mbpsとかになるので、スイッチを1つ作成し、アタッチする
  - まずサーバーをシャットダウンし、サーバーの詳細からNIC→追加
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgxODUzNjQwXX0=
-->