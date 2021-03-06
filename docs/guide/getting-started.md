# Getting started

## Quick start

First you need to create a `sql.DB`, for example, using the SQLite3 [driver](drivers.md):

```go
import _ "github.com/mattn/go-sqlite3"

sqldb, err := sql.Open("sqlite3", ":memory:?cache=shared")
if err != nil {
	panic(err)
}
```

And then create a `bun.DB` on top of it using the corresponding SQLite [dialect](drivers.md) that
comes with Bun:

```go
import (
	"github.com/uptrace/bun"
	"github.com/uptrace/bun/dialect/sqlitedialect"
)

db := bun.NewDB(sqldb, sqlitedialect.New())
```

Now you are ready to issue some queries:

```go
type User struct {
	ID	 int64
	Name string
}

user := new(User)
err := db.NewSelect().
	Model(user).
	Where("name != ?", "").
	OrderExpr("id ASC").
	Limit(1).
	Scan(ctx)
```

The code above is equivalent to:

```go
query := "SELECT id, name FROM users AS user WHERE name != '' ORDER BY id ASC LIMIT 1"

rows, err := sqldb.QueryContext(ctx, query)
if err != nil {
	panic(err)
}

if !rows.Next() {
    panic(sql.ErrNoRows)
}

user := new(User)
if err := db.ScanRow(ctx, rows, user); err != nil {
	panic(err)
}

if err := rows.Err(); err != nil {
    panic(err)
}
```

## Basic example

First we need to load some data for our [basic example](example](/example/basic/). To provide
initial data we are going to use Bun [fixtures](https://bun.uptrace.dev/guide/fixtures.html):

```go
import "github.com/uptrace/bun/dbfixture"

// Register models for the fixture.
db.RegisterModel((*User)(nil), (*Story)(nil))

// WithRecreateTables tells Bun to drop existing tables and create new ones.
fixture := dbfixture.New(db, dbfixture.WithRecreateTables())

// Load fixture.yaml which contains data for User and Story models.
if err := fixture.Load(ctx, os.DirFS("."), "fixture.yaml"); err != nil {
	panic(err)
}
```

The `fixture.yaml` looks like this:

```yaml
- model: User
  rows:
    - _id: admin
      name: admin
      emails: ['admin1@admin', 'admin2@admin']
    - _id: root
      name: root
      emails: ['root1@root', 'root2@root']

- model: Story
  rows:
    - title: Cool story
      author_id: '{{ $.User.admin.ID }}'
```

To select all users:

```go
users := make([]User, 0)
if err := db.NewSelect().Model(&users).OrderExpr("id ASC").Scan(ctx); err != nil {
	panic(err)
}
```

To select a single user by id:

```go
user1 := new(User)
if err := db.NewSelect().Model(user1).Where("id = ?", 1).Scan(ctx); err != nil {
	panic(err)
}
```

To select a story and the associated author in a single query:

```go
story := new(Story)
if err := db.NewSelect().
	Model(story).
	Relation("Author").
	Limit(1).
	Scan(ctx); err != nil {
	panic(err)
}
```

To select a user into a map:

```go
m := make(map[string]interface{})
if err := db.NewSelect().
	Model((*User)(nil)).
	Limit(1).
	Scan(ctx, &m); err != nil {
	panic(err)
}
```

To select all users scanning each column into a separate slice:

```go
var ids []int64
var names []string
if err := db.NewSelect().
	ColumnExpr("id, name").
	Model((*User)(nil)).
	OrderExpr("id ASC").
	Scan(ctx, &ids, &names); err != nil {
	panic(err)
}
```

## What's next

By now, you should have a basic understanding of Bun API. Next, learn how to
[write queries](queries.md) or jump straight to [starter kit](starter-kit.md).
