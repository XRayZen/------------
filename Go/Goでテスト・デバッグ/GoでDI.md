
>[GoのWebアプリをテストするノウハウ](https://zenn.dev/media_engine/articles/testing-go-applications)
>[Go言語でテスト駆動開発を学ぶ](https://andmorefine.gitbook.io/learn-go-with-tests/go-fundamentals/dependency-injection)

# GoでDI(Dependency Injection/依存性注入)

このパターンでは、外部と連携する責務をインターフェースで表現し、アプリコードではそのインターフェースを利用して機能を実装する
- この方法は、特定のメソッドや関数などのユニットテストなどを書く際に利用するとよい

具体的には、まず次のようなインターフェースを実装する

```go
package api

type DBRepository interface {
	GetUserInfo(userID string) (resUserInfo Data.UserInfo, err error)
	GetExploreCategories(userID string, country string) (resExp Data.ExploreCategories, err error)
}
```
次にこのインターフェースの機能を実現する構造体を実装する
- Goでは該当interfaceで宣言した同一メソッドを実装することで、自動的にそのinterfaceを実装（継承？）したとみなされます。
```go
// ... 省略 ...
type DBRepoImpl struct {
}

func (s DBRepoImpl) GetUserInfo(userID string) (resUserInfo Data.UserInfo, err error) {
	... 省略 ...
}

func (s DBRepoImpl) GetExploreCategories(userID string, country string) (resExp Data.ExploreCategories, err error) {
	... 省略 ...
}
```
そして、Mockを実装する
```go
// ... 省略 ...
type MockDBRepo struct {
}

func (s MockDBRepo) GetUserInfo(userID string) (resUserInfo Data.UserInfo, err error) {
    // モックが返す値を定義する
	return Data.UserInfo{}, nil
}

func (s MockDBRepo) GetExploreCategories(userID string, country string) (resExp Data.ExploreCategories, err error) {
	return Data.ExploreCategories{
		CategoryName:        "CategoryName",
	}, nil
}
```
## 利用する側
```go
...省略...
// 利用する側のメンバにインターフェイスを定義しておく
type Explore struct {
	DBrepo DBRepo.DBRepository
}
...省略...
// 利用する時に構造体の中にそれを実装した構造体を入れて依存性を注入（DI）する
// クリーンアーキテクチャではエントリポイントでDIする
str := Explore{
	DBrepo: diDBRepo,
}
res,err= str.GetExploreCategories(userID)
```
## テストコード
- テストではモックをDIする
- モックが返す値も定義するのがベストだが、モックの実装によっては不要な場合もある
```go
...省略...
func TestParseRequestType(t *testing.T) {
	// 正解のデータを用意する
	want_Ex, _ := json.Marshal(Data.ExploreCategories{
		CategoryName: "CategoryName",
	})
	want_ExStr := string(want_Ex)
	want_Rs, _ := json.Marshal(Data.Ranking{
		RankingName: "RankingName",
	})
	want_RsStr := string(want_Rs)
	type args struct {
		diDBRepo    DBRepo.DBRepository
		requestType string
		userID      string
	}
	tests := []struct {
		name    string
		args    args
		want    string
		wantErr bool
	}{
		{
			name: "正常系 ExploreCategories",
			args: args{
				// ここでDIしているがargsでした方が良いかも
				diDBRepo:    DBRepo.MockDBRepo{},
				requestType: "ExploreCategories",
				userID:      "userID",
			},
			want: want_ExStr,
		},
		{
			name: "正常系 Ranking",
			args: args{
				diDBRepo:    DBRepo.MockDBRepo{},
				requestType: "Ranking",
				userID:      "userID",
			},
			want: want_RsStr,
		},
		{
			name: "異常系 invalid request type",
			args: args{
				diDBRepo:    DBRepo.MockDBRepo{},
				requestType: "invalid request type",
				userID:      "userID",
			},
			want:    "",
			wantErr: true,
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got, err := ParseRequestType(tt.args.diDBRepo, tt.args.requestType, tt.args.userID)
			if (err != nil) != tt.wantErr {
				t.Errorf("ParseRequestType() error = %v, wantErr %v", err, tt.wantErr)
				return
			}
			if got != tt.want {
				t.Errorf("ParseRequestType() = %v, want %v", got, tt.want)
			}
		})
	}
}
```

