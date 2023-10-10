# Goでのenum
>[goのenumは文字列でいいのではないかという話](https://zenn.dev/kyoh86/articles/qiita-18b8bfc6ffe045aaf380)
- Goではenumは存在しない
- 代わりに、constを利用する
  - かなりややこしい


```go
type Zodiac string

const (
	Rat    = Zodiac("Rat")
	Ox     = Zodiac("Ox")
	Tiger  = Zodiac("Tiger")
	Rabbit = Zodiac("Rabbit")
	Dragon = Zodiac("Dragon")
	Snake  = Zodiac("Snake")
	Horse  = Zodiac("Horse")
	Sheep  = Zodiac("Sheep")
	Monkey = Zodiac("Monkey")
	Cock   = Zodiac("Cock")
	Dog    = Zodiac("Dog")
	Boar   = Zodiac("Boar")
)
```

