# 正常系・異常系テストケース

```go
func TestApiRequestLimit(t *testing.T) {
	// 最初にユーザーを登録する
	db_repo := DBRepo.DBRepoImpl{}
	// DBにモックモードで接続する（SQLite：メモリ上にDBを作成する）
	db_repo.ConnectDB(true)
	db_repo.AutoMigrate()
	db_repo.RegisterUser(Data.UserConfig{
		UserName:     "test",
		UserUniqueID: "test",
		AccountType:  "Free",
		Country:      "JP",
	})
	// テスト用オブジェクトを用意する
	input_add_api_config_json, _ := json.Marshal(Data.ApiConfig{
		AccountType:            "Free",
		RefreshArticleInterval: 10,
	})
	input_update_api_config_json, _ := json.Marshal(Data.ApiConfig{
		AccountType:            "Free",
		RefreshArticleInterval: 20,
	})
	// テストケース
	type args struct {
		requestType    string
		userId         string
		argumentJson_1 string
		argumentJson_2 string
	}
	tests := []struct {
		name    string
		args    args
		want    string
		is_want_err bool // エラーが発生するかどうか
	}{
		// AddApiRequestLimit
		{
			name: "AddApiRequestLimit",
			args: args{
				requestType:    "ModifyAPIRequestLimit",
				userId:         "test",
				argumentJson_1: "Add",
				argumentJson_2: string(input_add_api_config_json),
			},
			want:    "Success ModifyAPIRequestLimit",
			is_want_err: false,
		},
		// GetApiRequestLimit
		{
			name: "GetApiRequestLimit",
			args: args{
				requestType:    "GetAPIRequestLimit",
				userId:         "test",
				argumentJson_1: "",
				argumentJson_2: "",
			},
			want:    string(input_add_api_config_json),
			is_want_err: false,
		},
		// UpdateApiRequestLimit
		{
			name: "UpdateApiRequestLimit",
			args: args{
				requestType:    "ModifyAPIRequestLimit",
				userId:         "test",
				argumentJson_1: "Update",
				argumentJson_2: string(input_update_api_config_json),
			},
			want:    "Success ModifyAPIRequestLimit",
			is_want_err: false,
		},
		// DeleteApiRequestLimit
		{
			name: "DeleteApiRequestLimit",
			args: args{
				requestType:    "ModifyAPIRequestLimit",
				userId:         "test",
				argumentJson_1: "UnscopedDelete",
				argumentJson_2: string(input_add_api_config_json),
			},
			want:    "Success ModifyAPIRequestLimit",
			is_want_err: false,
		},
		// 異常系で削除されているのに読み込もうとする
		{
			name: "Failed GetApiRequestLimit",
			args: args{
				requestType:    "GetAPIRequestLimit",
				userId:         "test",
				argumentJson_1: "",
				argumentJson_2: "",
			},
			want:    "record not found",
			is_want_err: true,
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got, err := ParseRequestType("", db_repo,
				tt.args.requestType, tt.args.userId, tt.args.argumentJson_1, tt.args.argumentJson_2)
			if err != nil && tt.is_want_err {
				if err.Error() == tt.want {
					return // 期待したエラーが発生しているのでテスト成功
				}
				// 期待したエラーが発生していないのでテスト失敗
				t.Errorf("RequestHandler.HandleRequest() errorType: %v error = %v, wantErr %v", tt.args.requestType, err, tt.is_want_err)
			} else if err != nil && !tt.is_want_err {
				t.Errorf("RequestHandler.HandleRequest() errorType: %v error = %v, wantErr %v", tt.args.requestType, err, tt.is_want_err)
			}
			if got != tt.want {
				t.Errorf(err.Error())
				t.Errorf("RequestHandler.HandleRequest() = %v, want %v", got, tt.want)
			}
		})
	}
}

```



