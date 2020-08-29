# Golang

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


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkyMzU4OTE3MF19
-->