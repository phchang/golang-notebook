# Comments
## Deprecation Warnings

```go
const (
    SomeObsoleteConstant = "Don't use me" // Deprecated: Use something else instead
)
```

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

# Testing

## Mocking http.Client
```go
type HttpClient interface {
    Do(*http.Request) (*http.Response, error)
}

type SomeService struct {
    Client HttpClient // interface used instead of actual http.Client
}
```
Production code will instantiate a new `SomeService` with an actual `http.Client` reference.
```go
svc := SomeService{
    Client: &http.Client{}, // http.Client fulfills the HttpClient interface
}
```
Test code will create a new mock.
```go
type mockClient struct {
    MockDo func(*http.Request) (*http.Response, error)
}

func (c mockClient) Do(request *http.Request) (*http.Response, error) {
    return c.MockDo(request)
}

func TestSomeService(t *testing.T) {
	client := mockClient{
		MockDo: func(request *http.Request) (*http.Response, error) {
			// generate a mock response and/or make assertions about request
			mockResponse := []byte("some canned response")
			return &http.Response{
				Body: ioutil.NopCloser(bytes.NewReader(mockResponse))
			}, nil
		},
	}

	svc := SomeService{Client: client}
    // test the client
}
```
