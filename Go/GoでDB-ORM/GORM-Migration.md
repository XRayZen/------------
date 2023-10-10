- [Migration](#migration)
  - [Auto Migration](#auto-migration)
- [Migrator Interface](#migrator-interface)
  - [例：](#例)


# Migration
## Auto Migration
スキーマ定義のマイグレーションを自動で行い、スキーマを最新の状態に保ちます。

!!! warning 注意!: AutoMigrate はテーブル、外部キー、制約、カラム、インデックスを作成します。 
    カラムのサイズ、精度、null可否などが変更された場合、既存のカラムの型を変更します。
    - しかし、データを守るために、使われなくなったカラムの削除は実行されません。
```go
db.AutoMigrate(&User{})

db.AutoMigrate(&User{}, &Product{}, &Order{})

// Add table suffix when creating tables
db.Set("gorm:table_options", "ENGINE=InnoDB").AutoMigrate(&User{})
```

!!! warning 注意 AutoMigrate はデータベースの外部キー制約を自動的に作成します。初期化時にこの機能を無効にすることができます。
```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  DisableForeignKeyConstraintWhenMigrating: true,
})
```
# Migrator Interface
GORMは、データベースに依存しないスキーママイグレーションを構築するために使用できる、各データベースのための統一されたAPIインタフェースを含むmigratorインタフェースを提供します。

## 例：
SQLiteはALTER COLUMN, DROP COLUMNをサポートしていませんが、GORMは変更しようとしているテーブルを新しいテーブルとして作成し、すべてのデータをコピーし、古いテーブルを削除し、新しいテーブルの名前を変更します。

MySQLは、いくつかのバージョンではカラム名変更、インデックス変更をサポートしていませんが、GORMは使用しているMySQLのバージョンに応じて異なるSQLを実行します。
```go
type Migrator interface {
  // AutoMigrate
  AutoMigrate(dst ...interface{}) error

  // Database
  CurrentDatabase() string
  FullDataTypeOf(*schema.Field) clause.Expr

  // Tables
  CreateTable(dst ...interface{}) error
  DropTable(dst ...interface{}) error
  HasTable(dst interface{}) bool
  RenameTable(oldName, newName interface{}) error
  GetTables() (tableList []string, err error)

  // Columns
  AddColumn(dst interface{}, field string) error
  DropColumn(dst interface{}, field string) error
  AlterColumn(dst interface{}, field string) error
  MigrateColumn(dst interface{}, field *schema.Field, columnType ColumnType) error
  HasColumn(dst interface{}, field string) bool
  RenameColumn(dst interface{}, oldName, field string) error
  ColumnTypes(dst interface{}) ([]ColumnType, error)

  // Views
  CreateView(name string, option ViewOption) error
  DropView(name string) error

  // Constraints
  CreateConstraint(dst interface{}, name string) error
  DropConstraint(dst interface{}, name string) error
  HasConstraint(dst interface{}, name string) bool

  // Indexes
  CreateIndex(dst interface{}, name string) error
  DropIndex(dst interface{}, name string) error
  HasIndex(dst interface{}, name string) bool
  RenameIndex(dst interface{}, oldName, newName string) error
}
```






