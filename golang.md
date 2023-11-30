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


## テーブルのインメモリ化

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

#### User を インメモリ化した例
ISUCON11-prior
https://github.com/yyamada12/isucon11-p_re/commit/6e0f273b57aa7c60b2a5e7940da4087a4c7ab132

#### ISUCON11の例
https://github.com/yyamada12/isucon11q_re/commit/e835f878bedf8c3739993ec89c0f53628d8e7f7b
https://github.com/yyamada12/isucon11q_re/commit/852cda6998a2267b823c66414db8fc7cd0d14811


#### 汎用的な構造体を作った
ジェネリクスで、key(string) で 構造体をキャッシュする
```
type SyncMap[T any] struct {
  m map[string]*T
  mu sync.RWMutex
}

func NewSyncMap[T any]() *SyncMap[T] {
  return  &SyncMap[T]{m: map[string]*T{}}
}

func (sm *SyncMap[T]) Add(key string, value T) {
  sm.mu.Lock()
  defer sm.mu.Unlock()
  sm.m[key] = &value
}

func (sm *SyncMap[T]) Get(key string) *T {
  sm.mu.RLock()
  defer sm.mu.RUnlock()
  return sm.m[key]
}

func (sm *SyncMap[T]) Clear() {
  sm.mu.Lock()
  defer sm.mu.Unlock()
  sm.m =  map[string]*T{}
}
```

## 再起動対策
アプリケーション起動時にインメモリにデータを読み込むことをすると、再起動時にDBが起動していない場合にアプリケーションが立ち上がらないという事態になってしまう

sql.Open() はコネクションプールを初期化するだけで、実際に繋ぎに行くのはSQLを実行するタイミングとのこと
なので、初期実装では起動直後にSQLが走らないのでエラーにならないが、インメモリ戦略を取るとエラーで落ちてしまう
よって、以下のような修正を入れる

```
	// db.Open() が成功した直後にこれを入れる.
	for {
		err := db.Ping()
		if err == nil {
			break
		}
		log.Print(err)
		time.Sleep(time.Second * 2)
	}
	log.Print("DB ready!")
```
参考: https://zenn.dev/methane/articles/020f037513cd6b701aee


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

Nginx の 設定変更も合わせて。
https://github.com/yyamada12/private-isucon/commit/d22933069832b300fa7a387c370f6fdfe18cc82a


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

Goでは以下のSort.Interface を実装する必要がある

```
type Interface interface {
	Len() int
	Less(i, j int) bool
	Swap(i, j int)
}
```

Map の Value で sort する例
https://github.com/yyamada12/leetcode/blob/main/go/explore/october2021/day22/sort_characters_by_frequency.go

```
type Pair struct {
	key rune
	val int
}

type PairList []Pair

func (p PairList) Len() int           { return len(p) }
func (p PairList) Less(i, j int) bool { return p[i].val < p[j].val }
func (p PairList) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

func main() {
	pairs := []Pair{}
	for key, val := range m {
		pairs = append(pairs, Pair{key, val})
	}
	sort.Sort(sort.Reverse(PairList(pairs)))
}
```


## uuid
```
import (
  "github.com/google/uuid"
)

id := uuid.NewString()
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI3MzUyNzQ1MSwxMDk0NzY3MzYxLDE1Mj
UxMjA3MjUsLTEwNjkxNTY0OTcsLTIyMDA5ODg1MSw0OTIwMDQ4
MDQsLTY5ODY2Njg3NiwyMTMyODMyOTUsLTkyMTUxMjYxMSw1OT
c4NDY5MzAsMjEwMDAwNjQ0NCwxMTUzODExMjI4LDY4NDMwNDQ0
NCwtMTg5ODA5NzUxOCwtMjA0NzM4OTA3NiwtMTA5NTk1MDI4OC
wxNjg5NDMxMzk4LDE1NDE4MzMwNDAsLTkzODI5MTUxNSw1NDYy
NTUzNjVdfQ==
-->