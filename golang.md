# Strings

Create a string from a byte array
```go
type Foo struct {
    Bar string
    Baz string
}

b, _ := xml.Marshal(Foo{Bar: "test", Baz: "struct"})

fmt.Println(string(b[:])
```
