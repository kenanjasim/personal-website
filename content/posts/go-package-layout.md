---
title: 'How should you structure a Go package?'
date: "2025-09-15"
author: "Kenan Jasim"
tags: ["golang", "software engineering"]
readTime: true
toc: true
summary: 'A layout that dodges the "util" dumping ground and the circular-import mess.'
description: 'A layout that dodges the "util" dumping ground and the circular-import mess.'
---

Every Go project I've grown past a few thousand lines has hit the same wall: where does this file actually go? Get it wrong early and the codebase rots fast. Here's the layout that's held up best for me, based on Ben Johnson's standard package layout.

## Layouts to avoid

A few common approaches that turn sour:

- **One giant package.** Handy for dodging circular imports, miserable to navigate once it grows.
- **Group by function** (all handlers here, all controllers there). You get stuttering names like `controller.UserController`, and circular dependencies as the layers reach across each other.
- **Group by module.** Same problems in a different coat.

## Root package: domain types only

Put your domain types, the plain data and the interfaces describing how they interact, in the root package. And let it depend on none of your other packages.

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

`User` is just data. `UserService` describes what you can do with it but says nothing about *how*. No database, no HTTP, nothing messy. That's the point: the middle of your app stays clean, and everything else points inward at it.

## Sub-packages grouped by dependency

Each sub-package is an adapter between your clean domain and one messy outside thing, and it implements the domain's interfaces:

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

All the Postgres-specific mess lives in exactly one place. Want to swap it for SQLite, or an in-memory version for tests? Write another package that satisfies the same `myapp.UserService` interface, and nothing in your domain changes. That isolation is the whole payoff, and it's what makes the app testable.

## A shared mock package

Because everything talks through domain interfaces, you can keep a `mock` package of fake implementations. Your tests wire those in instead of spinning up a real database, so they stay fast and focused on logic rather than infrastructure.

## main wires it together

Your `main` package is the only place that knows which real adapters to plug in. It imports the concrete packages, connects them to the domain, and starts things up. It's the one spot allowed to touch everything, and keeping that knowledge in a single place is exactly why the rest stays clean.

## Conclusion

The rules, short version:

1. Domain types go in the root package, depending on nothing of yours.
2. Group sub-packages by the dependency they wrap, each implementing a domain interface.
3. Keep a shared `mock` package for tests.
4. Let `main` be the only place that wires the real things together.

And never make a `util` package. If you can't say what a package is for in a short phrase, that's the smell.
