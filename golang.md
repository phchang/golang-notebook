# Concurrency
Use mutual exclusion lock for thread safety
```go
mockServiceCall := func() int {
    return rand.Intn(100)
}

syncMutex := sync.Mutex{}
var results []int

waitGroup := sync.WaitGroup{}
for i := 0; i < 5; i++ {
    waitGroup.Add(1)
    go func() {
        defer waitGroup.Done()

        result := mockServiceCall()

        syncMutex.Lock()
        results = append(results, result)
        syncMutex.Unlock()
    }()
}
waitGroup.Wait()

fmt.Println(results)
```

# Modules

Use the `replace` directive to work on a dependent module locally:
```
replace example.com/original/import/path => /your/forked/import/path
```
Or use a relative path:
```
replace example.com/project/foo => ../foo
```

# Strings

Create a string from a byte array
```go
type Foo struct {
    Bar string
    Baz string
}

b, _ := xml.Marshal(Foo{Bar: "test", Baz: "struct"})
fooAsXML := string(b[:])
fmt.Println(fooAsXml)
```
