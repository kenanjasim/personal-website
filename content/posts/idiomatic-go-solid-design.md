---
title: 'Does SOLID even apply to Go?'
date: "2025-12-08"
author: "Kenan Jasim"
tags: ["golang", "software engineering"]
readTime: true
toc: true
summary: "A look at which of the SOLID principles still hold up in a language with no classes and no inheritance."
description: "A look at which of the SOLID principles still hold up in a language with no classes and no inheritance."
---

Something I have wondered about for a while is whether SOLID actually means much in Go. The principles were written with class-based languages like Java and C++ in mind, and Go has neither classes nor inheritance, so it is a fair question whether they still apply or whether I am just repeating a Java-era idea out of habit.

The view I have ended up with is that most of SOLID does still apply, but in Go it mostly reduces to one feature, the interface. If you understand small, implicitly satisfied interfaces, you have understood most of what people mean when they talk about idiomatic Go design.

## The main rule: accept interfaces, return structs

If there is one thing to take from this, it is this one.

```go
// Save accepts an interface, so it writes to anything with a Write
// method: a file, a socket, or a bytes.Buffer in a test.
func Save(w io.Writer, data []byte) error {
    _, err := w.Write(data)
    return err
}

// NewStore returns a concrete *Store, not an interface. The caller
// decides what to depend on, not you.
func NewStore(path string) *Store {
    return &Store{path: path}
}
```

`Save` asks for the smallest thing it needs, an `io.Writer`, so it behaves the same in production and in a test without knowing the difference. `NewStore` returns the concrete type. If you return an interface here, you have decided for the caller what they are allowed to see, which is often not what they want. It is usually better to return the struct and let them narrow it down on their side if they need to.

## SOLID without the classes

Taken one at a time, with the class machinery removed:

- **Single Responsibility** is really about package cohesion. Name packages for what they do (`net/http`, `os/exec`) rather than `util` or `common`, which tend to turn into dumping grounds.
- **Open/Closed** happens through embedding rather than inheritance, which I will come to below.
- **Liskov Substitution** is basically interfaces. Two types are interchangeable if the caller cannot tell them apart, and in Go a type satisfies an interface implicitly by having the right methods, with no `implements` keyword.
- **Interface Segregation** means depending only on the behaviour you use. If you only need to read, accept an `io.Reader` rather than a larger `File` type with fifteen methods.
- **Dependency Inversion** means your logic depends on an interface, and the concrete implementation sits behind it. The aim is an import graph that is wide and flat rather than tall and narrow.

Four of the five come down to using a small interface. That is not really a coincidence. Go put the useful part of SOLID into the language and left out most of the ceremony around it.

## Composition instead of inheritance

The one that tends to catch people coming from Java is Open/Closed. There is no `extends`, so you embed instead:

```go
type Logger struct{ prefix string }

func (l Logger) Log(msg string) { fmt.Println(l.prefix, msg) }

type Server struct {
    Logger        // embedded
    addr   string
}
```

Because `Logger` is embedded, `Server` gets its `Log` method promoted, so `srv.Log("started")` works without writing a forwarding method or touching `Logger`'s code. You have extended the behaviour without changing the original, which is what Open/Closed is asking for, done through composition.

## Conclusion

When I am thinking about a design in Go, I keep coming back to a few things:

1. Accept interfaces and return structs.
2. Keep interfaces small, ideally a single method.
3. Name packages for what they do, and avoid `util`.
4. Compose with embedding rather than reaching for inheritance that is not there.

I would not say SOLID is wrong in Go so much as mostly redundant. The language already pushes you towards small interfaces, and small interfaces are most of the point.
