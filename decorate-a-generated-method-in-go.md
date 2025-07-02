---
date: 2024-05-26
tags: golang
---
# Decorate a generated method in Go

I'm using `github.com/tinylib/msgp` to add marshal/unmarshal code to my structs. Some of my structs have pointer alias. So I have to ignore the alias via `msg:"-"` and assign it to its source after unmarshaling.

But assigning aliases after each unmarshal call is repetitive and prone to errors. After hours of frustration, I devised a simple and so obvious trick to eliminate assigning alias at unmarshaling call site: renaming the struct and embedding it in a wrapper whose name is the struct's original name, then override and decorate its unmarshal method.

For an example, my struct:

```
type MyStruct struct {
    name       string
    age        uint64
    ref        *Foo
    anotherRef *Foo `msg:"-"`
}

// generated method
func (z *MyStruct) UnmarshalMsg(b []byte) error {}
```

becomes

```
type MyStruct struct {
    myStructInner
}

// "decorated" method
func (z *MyStruct) UnmarshalMsg(b []byte) error {
    if err := z.myStructInner.UnmarshalMsg(b); err != nil {
        return err
    }
    z.anotherRef = z.ref // assigning pointer alias
    return nil
}

type myStructInner struct {
    name       string
    age        uint64
    ref        *Foo
    anotherRef *Foo `msg:"-"`
}

// generated method
func (z *myStructInner) UnmarshalMsg(b []byte) error {}
```

The trick is so obvious that I wish I had came up with it sooner.