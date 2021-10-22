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

#### thread safe な counter 実装の例
ISUCON11-prior
https://github.com/yyamada12/isucon11-p_re/commit/5380ce2be09d3c0923acdf9468daa1dbbc0f1d5c 

#### User を オンメモリ化した例
ISUCON11-prior
https://github.com/yyamada12/isucon11-p_re/commit/6e0f273b57aa7c60b2a5e7940da4087a4c7ab132

#### ISUCON11の例
https://github.com/yyamada12/isucon11q_re/commit/e835f878bedf8c3739993ec89c0f53628d8e7f7b
https://github.com/yyamada12/isucon11q_re/commit/852cda6998a2267b823c66414db8fc7cd0d14811


## responseのcache
user 関係なしで返す結果が決まっている場合、レスポンスはキャッシュを返して、定期的にキャッシュを更新させれば良い

sync.mutexでキャッシュデータにロックをかけつつ、
go func() で 無限forループで sleepかけながら更新処理を実行する

### ISUCON11の例
https://github.com/yyamada12/isucon11q_re/commit/9df5ea0bc472ba9fe327a0196a6312b08bb595fe

### ISUCON10の例
https://github.com/yyamada12/isucon10_re3/commit/c77c3f29712fcfc95c05dc642d72d18237349eea

### responseはjson.Marshalしてからcacheしておくことで、go側の負荷も減らせる
https://github.com/yyamada12/isucon10_re3/commit/aad2f272a49922ed7bb9b84433a0fd68a89695c6

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

## API call の並列化

複数のAPI call を直列で呼び出していると遅い
goroutine を使って並列で投げた後、ch で受け取って結果を使用すれば良い

https://github.com/yyamada12/isucon9_re2/commit/9bca99a6a74801960274bad671d684fcb09a59c8


## DB 内の画像を保存する処理
dataを []byte で保持できていれば ioutil.WriteFile()で書き出せる
```
err := ioutil.WriteFile("/home/isucon/isubata/webapp/public/icons/"+name, data, 0666)
```

https://github.com/yyamada12/isucon7_re3/commit/0b575b47f11c9baf45b9f8d096aca5a3847938d3#

## SQL周り

それぞれのメソッドに対して xxxContext() メソッドが存在する。
引数の一つ目でcontextを受け取って、キャンセルされた際にクエリをキャンセルするメソッド。

### database/sql
- INSERT, UPDATE, DELETE: db.Exec()
```
func (db *DB) Exec(query string, args ...interface{}) (Result, error)
```
→ LastInsertedID, affectedRowsが取れる
```
type Result interface {
	LastInsertId() (int64, error)
	RowsAffected() (int64, error)
}
```

- SELECT: db.QueryRow (1行), db.Query (複数行)
row を Scan して使う
```
    age := 27
	rows, err := db.QueryContext(ctx, "SELECT name FROM users WHERE age=?", age)
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()

	names := make([]string, 0)
	for rows.Next() {
		var name string
		if err := rows.Scan(&name); err != nil {
			log.Fatal(err)
		}
		names = append(names, name)
	}
	// Check for errors from iterating over rows.
	if err := rows.Err(); err != nil {
		log.Fatal(err)
	}
	log.Printf("%s are %d years old", strings.Join(names, ", "), age)
```


### github.com/jmoiron/sqlx
- SELECT: dbx.Get(1行), dbx.Select(複数行)

```
    people := []Person{}
    err := db.Select(&people, "SELECT * FROM person ORDER BY first_name ASC")

    jason := Person{}
    err = db.Get(&jason, "SELECT * FROM person WHERE first_name=$1", "Jason")
```

### IN句へのバインド

https://github.com/Nagarei/isucon11-qualify-test/commit/a553313dea8d43abb241bf5c570ea96c723ba9c9

### bulk insert
https://github.com/yyamada12/isucon10_re2/commit/586d084ea40519c13a9d02ee8a6d3118672f8955

https://github.com/yyamada12/isucon11q_re/commit/c4a8d63c4ae75ddd9694b154c9e6fb9acd451825


## Sort周り

GoではSort.Interface を実装する必要がある

```
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int

	// Less reports whether the element with index i
	// must sort before the element with index j.
	//
	// If both Less(i, j) and Less(j, i) are false,
	// then the elements at index i and j are considered equal.
	// Sort may place equal elements in any order in the final result,
	// while Stable preserves the original input order of equal elements.
	//
	// Less must describe a transitive ordering:
	//  - if both Less(i, j) and Less(j, k) are true, then Less(i, k) must be true as well.
	//  - if both Less(i, j) and Less(j, k) are false, then Less(i, k) must be false as well.
	//
	// Note that floating-point comparison (the < operator on float32 or float64 values)
	// is not a transitive ordering when not-a-number (NaN) values are involved.
	// See Float64Slice.Less for a correct implementation for floating-point values.
	Less(i, j int) bool

	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgyNjY2NTU0MCwtOTIxNTEyNjExLDU5Nz
g0NjkzMCwyMTAwMDA2NDQ0LDExNTM4MTEyMjgsNjg0MzA0NDQ0
LC0xODk4MDk3NTE4LC0yMDQ3Mzg5MDc2LC0xMDk1OTUwMjg4LD
E2ODk0MzEzOTgsMTU0MTgzMzA0MCwtOTM4MjkxNTE1LDU0NjI1
NTM2NSwtOTc3MTkyNjM2LC03NTk3NjI4NjUsLTg5NzQ4ODUxLC
0xMTA2ODA3Mjk1XX0=
-->