# Sqlxtx [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) [![CircleCI](https://circleci.com/gh/andresmijares/sqlxtx.svg?style=svg)](https://circleci.com/gh/andresmijares/sqlxtx) [![Coverage Status](https://coveralls.io/repos/github/andresmijares/sqlxtx/badge.svg?branch=main)](https://coveralls.io/github/andresmijares/sqlxtx?branch=main)

Changes behavior between `sqlx.DB` and `sqlx.Tx` in runtime.

## Installation 
```bash
go get -u github.com/andresmijares/sqlxtx
```

## Usage
```golang
// db.go
// Will manage each operation independently 
func init() {
    datasourceName := fmt.Sprintf("%s:%s@tcp(%s)/%s?charset=utf8",
		username,
		password,
		host,
		schema)
	var err error

	Client, err = sqlx.Open("mysql", datasourceName)
	if err != nil {
		panic(err)
	}

	WithTx = &sqlxtx.EnableSqlxTx{
		Client: Client,
		Config: sqlxtx.Config{
			Verbose: true,
		},
	}
}
// domain.go
type User struct {
	Client *sqlx.DB
}

func (u *User) Create(username, password string) error {
    _, err := u.Client.NamedExec("INSERT INTO users (first_name, last_name) VALUES (:first_name, :last_name);",
		map[string]interface{}{
			"first_name": "Jhon",
			"last_name":  "Doe",
		})
    if err != nil {
		return err
    }
    return nil
}

type Events struct {
	Client *sqlx.DB
}

func (u *Events) Create(name string) error {
    _, err := e.Client.NamedExec(`INSERT INTO events(name) VALUE (:name);`,
		map[string]interface{}{
			"name": name,
		})
    if err != nil {
		return err
    }
    return nil
}

// service.go
UserDao := &User{
	Client: Client,
}
EventsDao := &Events{
	Client: Client,
}

func CreateUser () {
    if err := UserDao.Create(); err != nil {
		return err
    }

    if err := EventsDao.Create("userCreated"); err != nil {
		return err
    }

    return nil
}

// Will run a transaction, not need to modify the underline implementation
func CreateUserWithTx () {
    if err := db.WithTx.Exec(func() error {
	if err := UserDao.Create(); err != nil {
	    return err
	}

	if err := EventsDao.Create("userCreated"); err != nil {
	    return err
	}

		return nil
    }); err != nil {
		return err
    }
    return nil
}
```

## Test
Sample how to test this can be found in the [sample folder](./sample/sample.go).

## Limitations
I've mocked only the most commons methods I use, for a more detailed list, please check [sqltxmocks](./sqlxtxmocks.go)

Nested transactions are not supported, yet.

## Motivation
Transactions should be implementation details, it should not force developers to re-write code to support between `Tx` and `DB`. I couldn't find a solid way to `decorate` operations in my services, so I created this one.

I found a lot of motivation in this articles.

 * [sqlx](https://github.com/jmoiron/sqlx)
 * [detect-and-commit-rollback](https://stackoverflow.com/questions/16184238/database-sql-tx-detecting-commit-or-rollback/23502629#23502629)
 * [db-transaction-in-golang](https://stackoverflow.com/questions/26593867/db-transaction-in-golang)
 * [golang transactions api design](https://stackoverflow.com/questions/51912841/golang-transactional-api-design)
 * [A clean way to implement database transaction in golang](https://dev.to/techschoolguru/a-clean-way-to-implement-database-transaction-in-golang-2ba)
 * [Go Microservice with clean architecture: transaction support](https://medium.com/@jfeng45/go-microservice-with-clean-architecture-transaction-support-61eb0f886a36)
 * [Isolation levels](https://github.com/launchbadge/sqlx/issues/481)

I only added the methods I use, please, feel free to submit PR's to support more methods.

## License
MIT
