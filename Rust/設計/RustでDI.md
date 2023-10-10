>https://laysakura.github.io/2021/04/25/rust-mockall/
## 1. リポジトリトレイトをまとめるおまとめトレイト
```rust
/// UseCaseなどの各所で都度同じような型パラメータを定義しないで済むように、リポジトリtraitをこのtraitの関連型としてまとめる。
/// 例えば、 `ARepository` と `BRepository` を両方使う `XUseCase` があった場合、この trait がなければ
///  `XUseCase<A: ARepository, B: BRepository>` と2つの型パラメーターが必要なところ、
/// `XUseCase<R: Repositories>` の1つで済む。
pub trait RepoTrait {
    type DbRepo: DbRepoTrait;

    fn db_repo(&self) -> &Self::DbRepo;
}
```
## 2. ユースケースで実装
- モックする為にライフタイム修飾子が必要
```rust
pub struct Usecase<'a, T: RepoTrait> {
    db_repo: &'a T::DbRepo,
}
```
- usecaseの実装
```rust
impl<'a,T: RepoTrait> Usecase<'a,T> {
    pub fn new(repo:&'a T) -> Self {
        Self {
            db_repo: repo.db_repo(),
        }
    }
    // 以下アプリの機能を実装
}
```
## 3. インフラ層でリポジトリトレイトを実現する実装
```rust
// 依存するリポジトリをまとめるDI構造体
//リポジトリtraitの具体型を決定する、静的なDI (Dependency Injection) をする
pub struct ImplRepos {
    db_repo: ImplDbRepo,
}

impl RepoTrait for ImplRepos {
    type DbRepo = ImplDbRepo;

    fn db_repo(&self) -> Self::DbRepo {
        self.db_repo.clone()
    }
}

impl ImplRepos {
    pub fn new(db_repo: ImplDbRepo /*More Impl Repo...*/) -> Self {
        Self { db_repo }
    }
}
```
#### DIする使用例
```rust
let db_repo = ImplDbRepo::new("user".to_string());
let implrepos=ImplRepos::new(db_repo);
let usecase= Usecase::new(implrepos);
```
## 4. テスト用にモックを作成
>https://github.com/laysakura/mockall-example-rs/blob/master/app/tests/test_use_case_search_users.rs
- MockAllを使う
```rust
struct TestRepos {
    pub test_repo: MockDbRepoTrait,
}

impl RepoTrait for TestRepos {
    type DbRepo = MockDbRepoTrait;

    fn db_repo(&self) -> &Self::DbRepo {
        &self.test_repo
    }
}

impl TestRepos {
    pub fn new(test_repo: MockDbRepoTrait) -> Self {
        Self { test_repo }
    }
}
```
- テストコード
```rust
#[test]
fn test_usecase() {
    let mut mock = MockDbRepoTrait::new();
    mock.expect_get().returning(|_| Ok("test".to_string()));
    let repos = TestRepos::new(mock);
    let usecase = Usecase::new(repos);
    assert_eq!(usecase.get().unwrap(), "test".to_string());
}
```
## まとめ$$
- リポジトリトレイトをまとめるおまとめトレイトを作成する
- ユースケースで実装する
- インフラ層でリポジトリトレイトを実現する実装
- テスト

## 参考




