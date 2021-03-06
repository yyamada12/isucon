# Golang

## *sql.DB のコネプ設定


### 推奨する sql.DB の設定 
[Re: Configuring sql.DB for Better Performance]([http://dsas.blog.klab.org/archives/2018-02/configure-sql-db.html](http://dsas.blog.klab.org/archives/2018-02/configure-sql-db.html)) より

-   SetMaxOpenConns() は必ず設定する。負荷が高くなってDBの応答が遅くなったとき、新規接続してさらにクエリを投げないようにするため。できれば負荷試験をして最大のスループットを発揮する最低限のコネクション数を設定するのが良いが、負荷試験をできない場合も  `max_connection`  やコア数からある程度妥当な値を判断するべき。
-   SetMaxIdleConns() は SetMaxOpenConns() 以上に設定する。アイドルな接続の解放は SetConnMaxLifetime に任せる。
-   SetConnMaxLifetime() は最大接続数 × 1秒 程度に設定する。多くの環境で1秒に1回接続する程度の負荷は問題にならない。1時間以上に設定したい場合はインフラ／ネットワークエンジニアによく相談すること。


### ISUCON 9 予選1位のnilさんの設定
[https://github.com/takonomura/isucon9-qualify/commit/adca81f6ae380ec01847900edbffd84357f3aa05#diff-1657355b63a2aa9f3a7cf547c0c5133aR330](https://github.com/takonomura/isucon9-qualify/commit/adca81f6ae380ec01847900edbffd84357f3aa05#diff-1657355b63a2aa9f3a7cf547c0c5133aR330)

```
db.SetConnMaxLifetime(10 * time.Second)
db.SetMaxIdleConns(512)
db.SetMaxOpenConns(512)
```

### wait_timeout
SetConnMaxLifetime は mysql の wait_timeout よりも小さくしておく必要がある
mysql の以下のコマンドで設定値を確認できる
(デフォルトは8時間らしいのでおそらく問題ない)
```
show global variables like '%wait%';
```

## N + 1 の解消
毎回恒例の基本問題。
IN句を使ったクエリで1回で取ってくるのが基本。

```
qs, params, err := sqlx.In(`SELECT id, name FROM user WHERE id IN (?)`, userIDs)
if err != nil {
    log.Fatal(err)
}

users := []User{}
if err := db.Select(&users, qs, params...); err != nil {
    log.Fatal(err)
}

res := make(map[int64]User)
for _, user := range users {
    res[user.ID] = user
}
return res
```


[https://github.com/yyamada12/isucon7_re3/commit/405f0770410613486b1e2f312bec4f4f82fca9b4](https://github.com/yyamada12/isucon7_re3/commit/405f0770410613486b1e2f312bec4f4f82fca9b4)


## テーブルのオンメモリ化

https://github.com/yyamada12/isucon10_re2/commit/99dbcd579758dcfbf1bf0494f0858032c866fd22

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


## prepared statement
mysql で prepare admin が多い場合は、go側でprepareさせてあげられるっぽい
[http://dsas.blog.klab.org/archives/52191467.html](http://dsas.blog.klab.org/archives/52191467.html)

以下のように `interpolateParams=true` をつけてあげるだけ
```
sql.Open("mysql",  "root:password@tcp(localhost:3306)/test?interpolateParams=true&collation=utf8mb4_bin")
```

## Logging
ログ出力の処理が残っているとかなり重い
### echoの場合
https://echo.labstack.com/guide/customization/
```
e := echo.New()
e.Debug = true
```
となっていると debug modeになっているので削除する

ログを完全に止めるには以下のように `Logger.SetLevel(log.OFF)` を使う
```
e := echo.New()
e.Logger.SetLevel(log.OFF)
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU0MTgzMzA0MCwtOTM4MjkxNTE1LDU0Nj
I1NTM2NSwtOTc3MTkyNjM2LC03NTk3NjI4NjUsLTg5NzQ4ODUx
LC0xMTA2ODA3Mjk1XX0=
-->