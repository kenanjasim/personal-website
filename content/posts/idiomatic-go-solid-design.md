---
title: 'Does SOLID even apply to Go?'
date: "2025-12-08"
author: "Kenan Jasim"
tags: ["golang", "software engineering"]
readTime: true
toc: true
summary: "A look at which of the SOLID principles survive in a language with no classes and no inheritance."
description: "A look at which of the SOLID principles survive in a language with no classes and no inheritance."
---

I've been chewing on something for a while. Does SOLID actually mean anything in Go? The principles were written for class-based languages like Java and C++, and Go has neither classes nor inheritance. So do they still apply, or am I just nodding along to a Java-era idea out of habit?

Where I landed: most of SOLID does apply, but it collapses into one Go feature. The interface. If you understand small, implicitly satisfied interfaces, you've understood most of what people mean by "idiomatic Go design."

## The one rule: accept interfaces, return structs

If you remember one thing, remember this one.

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

`Save` asks for the smallest thing it needs, an `io.Writer`, so it works the same in production and in a test without knowing the difference. `NewStore` hands back the real type. Return an interface here and you've decided for the caller what they're allowed to see, usually wrongly. Give them the struct and let them narrow it on their own end if they want to.

## SOLID, minus the classes

Each principle, with the class machinery stripped out:

- **Single Responsibility** is really about package cohesion. Name packages for what they do (`net/http`, `os/exec`), not `util` or `common`, which quietly turn into dumping grounds.
- **Open/Closed** happens through embedding, not inheritance (more below).
- **Liskov Substitution** is just interfaces. Two types are swappable if the caller can't tell them apart, and in Go a type satisfies an interface implicitly by having the right methods. No `implements` keyword needed.
- **Interface Segregation** means depend only on the behaviour you use. If you only read, accept an `io.Reader`, not a fat `File` with fifteen methods.
- **Dependency Inversion** means your logic depends on an interface and the messy concrete thing implements it. Aim for an import graph that's wide and flat, not tall and narrow.

Four of the five are "use a small interface" in different hats. That's not an accident. Go baked the useful part of SOLID into the language and skipped the ceremony.

## Composition instead of inheritance

The one that trips up people coming from Java is Open/Closed. There's no `extends`, so you embed:

```go
type Logger struct{ prefix string }

func (l Logger) Log(msg string) { fmt.Println(l.prefix, msg) }

type Server struct {
    Logger        // embedded
    addr   string
}
```

Because `Logger` is embedded, `Server` gets its `Log` method promoted for free. So `srv.Log("started")` just works, and you never wrote a forwarding method or touched `Logger`'s code. Extended behaviour, original left alone. That's Open/Closed, done with composition.

## Conclusion

When I'm weighing up a design in Go, I keep coming back to a few rules:

1. Accept interfaces, return structs.
2. Keep interfaces small, ideally one method. Require no more, promise no less.
3. Name packages for what they do, and never create a `util`.
4. Compose with embedding instead of reaching for inheritance you don't have.

SOLID isn't wrong in Go. It's mostly redundant. The language already pushes you toward small interfaces, and small interfaces are most of the game.
