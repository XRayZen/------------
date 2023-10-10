
# モデルを宣言する

モデルは Go の基本型、（基本型の）ポインタ/エイリアス、 Scanner および Valuer インターフェイスを実装するカスタム型からなる通常の構造体です。

例：

```go
type User struct {
  ID           uint
  Name         string
  Email        *string
  Age          uint8
  Birthday     *time.Time
  MemberNumber sql.NullString
  ActivatedAt  sql.NullTime
  CreatedAt    time.Time
  UpdatedAt    time.Time
}
```

## 規約

GORM は設定よりも規約を好みます。
- デフォルトでは、GORM は主キーとして `ID` を使用し、構造体名をスネークケースに複数形に変換してテーブル名とし、スネークケースをカラム名として使用し、作成日時と更新日時を追跡するために `CreatedAt`、`UpdatedAt` を使用します。

GORM の採用する規約に従えば、ほとんどの設定/コードを書く必要はありません。
- もし規約が要件に合わない場合、GORM ではそれらを設定することができます。

## gorm.Model
GORM は `gorm.Model` 構造体を定義しています。これには `ID`、`CreatedAt`、`UpdatedAt`、`DeletedAt` のフィールドが含まれます。

```go
// gorm.Modelの定義
type Model struct {
  ID        uint           `gorm:"primaryKey"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt gorm.DeletedAt `gorm:"index"`
}
```

この構造体を埋め込むことで、これらのフィールドを自身の構造体に含めることができます。Embedded Struct も参照してください。

## 高度な機能

### フィールドレベルの権限

Exported フィールドは GORM を使用して CRUD 操作を行う際にすべての権限を持っていますが、GORM ではタグを使用してフィールドレベルの権限を変更することができます。
- そのため、フィールドを読み取り専用、書き込み専用、作成専用、更新専用、または無視するように設定することができます。

!!! warning 注意:
    GORM Migrator を使用してテーブルを作成した場合、除外されたフィールドは作成されません。

以下はいくつかの例です:

```go
type User struct {
  Name string `gorm:"<-:create"` // 読み取りと作成が可能
  Name string `gorm:"<-:update"` // 読み取りと更新が可能
  Name string `gorm:"<-"`       // 読み取りと書き込みが可能（create と update）
  Name string `gorm:"<-:false"`  // 読み取りのみ可能、書き込みは無効
  Name string `gorm:"->"`        // 読み取り専用（変更されない限り、書き込みは無効）
  Name string `gorm:"->;<-:create"` // 読み取りと作成が可能
  Name string `gorm:"->:false;<-:create"` // 作成専用（データベースからの読み取りは無効）
  Name string `gorm:"-"`            // このフィールドを構造体を使用した書き込みと読み込みの際に無視する
  Name string `gorm:"-:all"`        // このフィールドを構造体を使用した書き込み、読み取り、マイグレーションの際に無視する
  Name string `gorm:"-:migration"`  // マイグレーション時にこのフィールドを無視する
}
```
