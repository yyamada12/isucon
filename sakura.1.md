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
  - サーバーをシャットダウンし、サーバーの詳細からNIC→追加で作成したスイッチを選択
  -  次に、ubuntu側で設定する `ip link` コマンドで
```
network: ethernets: eth0: addresses: - 203.0.113.14/24 dhcp4: 'no' dhcp6: 'no' gateway4: 203.0.113.1 nameservers: addresses: - 203.0.113.10 - 203.0.113.11 search: - localdomain eth1: addresses: - 192.168.0.41/16 dhcp4: 'no' dhcp6: 'no' renderer: networkd version: 2
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYzMTI3NzAxNSwtMTg2NTU5ODkwMF19
-->