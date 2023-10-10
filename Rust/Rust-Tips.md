
### asyn_walkDir DirEntry
```rust
let rrr = async_walkdir::WalkDir::new(&dir_path)
.into_stream()
.filter_map(|e|async move {e.ok()})
.collect::<Vec<_>>()
.await;
```
### ストリーミング並列イテレータ
```Rust
let example: Vec<_> = stream::iter(items)
    .map(|x| {
        tokio::spawn(async move {
            if example_tsak().await{
                return Ok(T)
            }
            return None;
        })
    })
    .buffer_unordered(N)
    .collect::<Vec<_>>()
    .await.into_par_iter().#ベクトルへの操作#.collect();
```
または、
```Rust
let mut tasks=vec![];
    for item in items {
        tasks.push(
            tokio::spawn(async move {
                #async_task
            });
        ); 
        
    }
let res = join_all(tasks).await.into_par_iter().flatten().collect::<Vec<_>>();
```
タスクを並列して実行させるにはtokio::spawnで囲う必要がある  
その後にfutures::future::join_allですべての結果をマージする  
どの方法でも大差ない

### Utc.timestampのopt
```Rust
Utc.timestamp_millis_opt(content.access_time).single().unwrap_or_default()
```

### iter()とinto_titer()の違い
1. iter()はVectorをmoveしない。
   - iter()はVectorから「参照のコレクションであるIterator」を作成し、
2. into_iter()はVectorをmoveする。
   - into_iter()はVectorから「値のコレクションであるIterator」を作成する。

### Vecに対する操作の高速化
1. rayonのpar_iter()のmap()で同期限定・非ブロッキングなし・リソースに制限無しなら使った方がいい

### vecの重複の削除
1. 完全に重複を無くしたい場合は、sort()してから、dedup()を使います。
```rust
let mut nums = vec![1, 2, 2, 2, 3, 3, 4, 5, 1, 5];
nums.sort();
nums.dedup();
//deduped [1, 2, 3, 4, 5]
```
>Rustの配列操作メモ:https://natsutan.hatenablog.com/entry/2018/11/19/182233