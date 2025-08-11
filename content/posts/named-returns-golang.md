+++
title = 'When to use named returns in Go?'
date = "2025-01-11"
author = "Kenan Jasim"
categories = ["golang"]
tags = ["discussion", "golang", "programming", "software engineering"]
+++

Recently while reviewing a PR addressing some linting issues, a colleague of mine pointed out that we had enabled the `nonamedreturns` linter rule in our project. This rule is part of the `golangci-lint` tool and it checks for named return values in functions.

This prompted a discussion about when, if ever, it is appropriate to use named return values in Go.

## What are named returns?

In Golang, when defining a function, you can "name" the return values. This means that you can specify the names of the return variables in the function signature. For example:

```go
func add(a, b int) (sum int) {
    sum = a + b
    return // This is a "naked return" - it automatically returns 'sum'
}
```

In this example, `sum` is a named return value. The function `add` returns the sum of `a` and `b`, and the return value is named `sum`.

You may also notice that the `return` statement does not specify a value. This is because the named return value `sum` is automatically returned when the function exits.

There are plenty of examples in the Go standard library where named returns are used, for example the 'context' package [`WithCancel`](https://pkg.go.dev/context#WithCancel) function uses named returns:

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

However, even within the same package, you will find functions that do not use named returns, such as the [`WithDeadline`](https://pkg.go.dev/context#WithDeadline) function:

```go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)
```

Ultimately, I came to this conclusion when reviewing the code. 

## Prefer small functions with clear return types

The ideal function should be small and have a clear purpose. If the function is small enough, the return types should be self-explanatory, and named returns may not be necessary.

For example, consider this function which does something and returns an error:

```go

func doSomething() error {
    // Do something
    return nil // or an error if something went wrong
}
```

This function is straightforward, and the return type is clear. There is no need for named returns here.

If the function is responsible for doing some action, and returns a single and an error, the function name or some function docstring should indicate what it is returning:

```go
// getSomething retrieves something and returns it along with an error if it fails.
func getSomething() (string, error) {
    // Get something and return it
    return "something", nil // or an error if something went wrong
}
```

## Where multiple returns are needed, use structs

When a function has multiple return values, especially if they are not self-explanatory, it is often better to use a struct to encapsulate the return values. This makes the code more readable and maintainable.

For example, instead of returning multiple values like this:

```go
func getSomeItems() (a string, b string, err error) {
    // Get some items and return them
    return "item1", "item2", nil // or an error if something went wrong
}
```

You can define a struct to encapsulate the return values:

```go
type Something struct {
    A string
    B string
}

func getSomething() (Something, error) {
    // Get something and return it
    return Something{A: "item1", B: "item2"}, nil // or an error if something went wrong
}
```

This approach makes it clear what each return value represents, and it allows for easier expansion in the future if more fields need to be added. 

## When to use named returns

Where structs are not appropriate, due to adiditonal complexity or where the function is small enough, named returns can be used. However, it is important to avoid "naked returns" (using `return` without specifying the return values).

For example, if you have a function that retrieves a value and a boolean indicating whether it exists, you might write:

```go
// getAndExists retrieves a value and indicates whether it exists.
func getAndExists() (val string, ok bool) {
    // Retrieve the value and set ok to true if it exists
    val = "some value"
    ok = true

    // Dont do naked returns, explicitly return what you want
    return val, ok
}
```

This way, the named return values are clear, and the function is still small and easy to understand.

## Conclusion

In conclusion, when deciding when I should use named returns in Go, I consider the following:

* If the function is small and the return types are self-explanatory, I prefer not to use named returns.
* If the function has multiple return values that are not self-explanatory, I prefer to use a struct to encapsulate the return values.
* If named returns are used, I avoid naked returns and explicitly return the named values.

Generally, we as software engineers should try to make sure that our code is simple, easy to read and understand, and maintainable. Named returns can be a useful tool in achieving this, however naked returns can often make the code less clear and harder to maintain. 

Therefore, I prefer to use named returns sparingly and only when they add clarity to the code.

