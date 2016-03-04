# JSH-API

[![GoDoc](https://godoc.org/github.com/derekdowling/go-json-spec-handler?status.png)](https://godoc.org/github.com/derekdowling/jsh-api)
[![Build Status](https://travis-ci.org/derekdowling/jsh-api.svg?branch=master)](https://travis-ci.org/derekdowling/jsh-api)
[![Go Report Card](http://goreportcard.com/badge/manyminds/api2go)](http://goreportcard.com/report/derekdowling/jsh-api)

A [JSON API](http://jsonapi.org) specification micro-service builder created on top of
[jsh](http://github.com/derekdowling/go-json-spec-handler), [Goji](http://goji.io), and [context](https://godoc.org/golang.org/x/net/context) to handle the nitty gritty but predictable (un)wrapping, validating, preparing, and logging necessary for any JSON API written in Go. The rest (storage, and business logic) is up to you.

## Setup

The easiest way to get started is like so:

```go
import github.com/derekdowling/jsh-api

// implement jshapi/store.CRUD interface and add resource specific middleware via Goji
userStorage := &UserStorage{}
resource := jshapi.NewCRUDResource("user", userStorage)
resource.UseC(yourUserMiddleware)

// setup a logger, your shiny new API, and give it a resource
logger := log.New(os.Stderr, "<yourapi>: ", log.LstdFlags)
api := jshapi.Default("<prefix>", true, logger)
api.Add(resource)

// launch your api
http.ListenAndServe("localhost:8000", api)
```

For a completely custom setup:

```go
import github.com/derekdowling/jsh-api

// manually setup your API
api := jshapi.New("<prefix>")

// add a custom send handler
jshapi.SendHandler = func(c context.Context, w http.ResponseWriter, r *http.Request, sendable jsh.Sendable) {
    // do some custom logging, or manipulation
    jsh.Send(w, r, sendable)
}

// add top level Goji Middleware
api.UseC(yourTopLevelAPIMiddleware)

http.ListenAndServe("localhost:8000", api)
```

## Feature Overview

### Fast CRUD Implementation

Quickly build resource APIs for:

```
POST /resources
GET /resources
GET /resources/:id
DELETE /resources/:id
PATCH /resources/:id
```

### Relationships

Routing for relationships too:

```
GET /resources/:id/relationships/otherResource(s)
GET /resources/:id/otherResource(s)
```

### Mutations

```
GET /resources/:id/<action>
```

### Other

* Request, Response, and 5XX Auto-Logging

## Storage Driver Example

Here is a basic example of how you might implement a CRUD usable
[Storage Driver](https://godoc.org/github.com/derekdowling/jsh-api/store#CRUD)
for a given resource using [jsh](https://godoc.org/github.com/derekdowling/go-json-spec-handler)
for Save and Update. This should give you a pretty good idea of how easy it is to
implement the Storage driver with jsh.


```go
type User struct {
    ID string
    Name string `json:"name"`
}

func Save(ctx context.Context, object *jsh.Object) (*jsh.Object, jsh.ErrorType) {
    user := &User{}
    err := object.Unmarshal("user", user)
    if err != nil {
        return err
    }

    // generate your id, however you choose
    user.ID = "1234"

    err := object.Marshal(user)
    if err != nil {
        return nil, err
    }

    // do save logic
    return object, nil
}

func Update(ctx context.Context, object *jsh.Object) (*jsh.Object, jsh.ErrorType) {
    user := &User{}
    err := object.Unmarshal("user", user)
    if err != nil {
        return err
    }

    user.Name = "NewName"
    
    err := object.Marshal(user)
    if err != nil {
        return nil, err
    }

    // perform patch
    return object, nil
}
```
