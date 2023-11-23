# さくらのクラウドでの環境構築


## クーポンコードの登録

https://manual.sakura.ad.jp/cloud/payment/coupon.html

[さくらのクラウド コントロールパネル](https://secure.sakura.ad.jp/cloud/) ]
→ [さくらのクラウド(IaaS)]をクリック
→ [オプション]をクリック
→ 左メニューの[クーポン]をクリックします。  
右上の[追加]をクリックします。

## ブリッジ作成

1. ブリッジ作成
2. サーバーをシャットダウン
3. サーバの詳細画面からNICを追加
4. 追加したNICにスイッチをアタッチ
5. ip addr コマンドで、追加sれたNICを確認 (ubuntu 22.04だとens4になるはず)
6. sudo vim /etc/netplan/01-isucon.yaml を作成し、以下の内容を記載 (IPは割り当てたいものを記載)
```
network:
  ethernets:
    ens4:
      addresses:
        - 192.168.0.1/16
      dhcp4: 'no'
      dhcp6: 'no'
```
7. sudo netplan apply
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQyMDkyNzMxMCwxODk0NDI5NjE2XX0=
-->