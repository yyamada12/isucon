# Golang

## 

### 推奨する sql.DB の設定 
[Re: Configuring sql.DB for Better Performance]([http://dsas.blog.klab.org/archives/2018-02/configure-sql-db.html](http://dsas.blog.klab.org/archives/2018-02/configure-sql-db.html))

-   SetMaxOpenConns() は必ず設定する。負荷が高くなってDBの応答が遅くなったとき、新規接続してさらにクエリを投げないようにするため。できれば負荷試験をして最大のスループットを発揮する最低限のコネクション数を設定するのが良いが、負荷試験をできない場合も  `max_connection`  やコア数からある程度妥当な値を判断するべき。
-   SetMaxIdleConns() は SetMaxOpenConns() 以上に設定する。アイドルな接続の解放は SetConnMaxLifetime に任せる。
-   SetConnMaxLifetime() は最大接続数 × 1秒 程度に設定する。多くの環境で1秒に1回接続する程度の負荷は問題にならない。1時間以上に設定したい場合はインフラ／ネットワークエンジニアによく相談すること。

## N + 1 の解消


## テーブルのオンメモリ化

### 使える条件
- テーブルサイズが小さいこと (必須) 
- マスタの様な修正が入らないテーブル
- UPDATEやINSERT, DELETEがあるテーブルの場合、複数台のサーバーから参照されないこと
→ 複数のサーバーから参照する場合 Redisなどを使う


### UPDATEやINSERT, DELETEがあるテーブルの注意点

更新する際にLockをかけないといけない

#### データを単純な map で持てる場合

sync.Map を使う
[go1.9から追加されたsync.Mapを使う](https://qiita.com/meta_closure/items/dd228e49aef8b67e872e)

#### map の map などで持つ場合
sync.RWMutexを使う
sync.Mutexより性能がいいらしい

↓ でsync.Mutexを使った書き方も紹介されている
[go1.9から追加されたsync.Mapを使う](https://qiita.com/meta_closure/items/dd228e49aef8b67e872e)

ISUCON5予選の例
[https://github.com/yyamada12/isucon5/commit/fac50cdd60a19bde0077d21aaf34cf6ff90a444f](https://github.com/yyamada12/isucon5/commit/fac50cdd60a19bde0077d21aaf34cf6ff90a444f)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ2NjMwMzM5MSwtMTEwNjgwNzI5NV19
-->