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


# Strings

Create a string from a byte array
```go
type Foo struct {
    Bar string
    Baz string
}

b, _ := xml.Marshal(Foo{Bar: "test", Baz: "struct"})
fooAsXML := string(b[:]
fmt.Println(fooAsXml)
```
