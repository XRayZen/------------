```go
package main

import (
    "fmt"
    "gorm.io/driver/sqlite"
    "gorm.io/gorm"
)

type Client struct {
    //  ApiKey Index
    ID int `json:"id"`
    UserName string `json:"userName"`
    //  Client login hashed password
    Password string `json:"password"`
    //  ApiKeys
    APIKeys []*APIKey `json:"apiKeys"`
}

type APIKey struct {
    //  ApiKey Index
    ID int `json:"id"`
    //  ApiKey Value
    Key string `json:"key"`
    //  ApiKey Client Relation
    ClientID int `json:"clientID"`
    //  ApiKey Client Info
    Client *Client `json:"client"`
}

func main() {

    db, err := gorm.Open(sqlite.Open("many2many.db"), &gorm.Config{})
    if err != nil {
        panic("failed to connect database")
    }

    // Migrate the schema
    err = db.AutoMigrate(&APIKey{}, &Client{})
    if err != nil {
        fmt.Print(err)
    }

    clientOne := Client{
        UserName:          "Client One",
    }
    db.Create(&clientOne)

    apiKeyOne := APIKey{
        Key:"one",
        Client: &clientOne,
    }
    apiKeyTwo := APIKey{
        Key:"two",
        Client: &clientOne,
    }

    db.Create(&apiKeyOne)
    db.Create(&apiKeyTwo)

    // Fetch from DB
    fetchedClient := Client{}

    db.Debug().Preload("APIKeys").Find(&fetchedClient, clientOne.ID)
    fmt.Println(fetchedClient)

    db.Delete(&clientOne)
    db.Delete(&apiKeyOne)
    db.Delete(&apiKeyTwo)
}
```