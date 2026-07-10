---
title: 'How should you structure a Go package?'
date: "2025-09-15"
author: "Kenan Jasim"
tags: ["golang", "software engineering"]
readTime: true
toc: true
summary: 'The package layout that has held up best for me, and why I avoid the "util" package.'
description: 'The package layout that has held up best for me, and why I avoid the "util" package.'
---

Every Go project I have worked on that grew past a few thousand lines eventually ran into the same question: where does this file actually belong? If you get it wrong early it tends to make the rest of the codebase harder to work in. The layout below is the one that has held up best for me, and it is based on Ben Johnson's standard package layout.

## Layouts that tend not to work

A few common approaches that I have seen cause problems:

- **One large package.** This avoids circular imports, but it becomes hard to navigate as it grows.
- **Grouping by function** (all the handlers in one place, all the controllers in another). You end up with repetitive names like `controller.UserController`, and the layers often start importing each other.
- **Grouping by module.** This tends to run into the same problems in a slightly different form.

## The root package holds the domain types

Put your domain types, meaning the plain data and the interfaces that describe how they are used, in the root package, and try to keep that package from depending on any of your other packages.

```go
// package myapp: the root. Pure domain, no dependencies on your own code.
package myapp

type User struct {
    ID   int
    Name string
}

type UserService interface {
    User(id int) (*User, error)
}
```

`User` here is just data. `UserService` describes what you can do with a user without saying anything about *how* it is done. There is no database or HTTP in here. The point is to keep the centre of the application free of that detail, so that everything else depends inward on it.

## Sub-packages grouped by their dependency

Each sub-package acts as an adapter between the clean domain and one external thing, and it implements the domain's interfaces:

```go
// package postgres: implements myapp.UserService against a real database.
package postgres

type UserService struct {
    db *sql.DB
}

func (s *UserService) User(id int) (*myapp.User, error) {
    // query the db, build and return a *myapp.User
}
```

All of the Postgres-specific code lives in one place. If you later want to swap it for SQLite, or an in-memory version for tests, you write another package that satisfies the same `myapp.UserService` interface, and the domain does not change. That separation is most of the reason to do this, and it is what makes the code straightforward to test.

## A shared mock package

Because everything communicates through the domain interfaces, you can keep a `mock` package of fake implementations. Tests use those instead of spinning up a real database, which keeps them fast and focused on the logic rather than the infrastructure.

## main wires everything together

The `main` package is the only place that knows which real adapters to use. It imports the concrete packages, connects them to the domain, and starts the application. It is the one place allowed to know about everything, and keeping that knowledge in a single place is part of why the rest stays clean.

## Conclusion

The short version:

1. Domain types go in the root package, and it depends on none of your other packages.
2. Group sub-packages by the dependency they wrap, with each one implementing a domain interface.
3. Keep a shared `mock` package for tests.
4. Let `main` be the only place that wires the real implementations together.

I would also avoid a `util` package. If it is hard to describe what a package is for in a short phrase, that is usually a sign it is doing too many things.
