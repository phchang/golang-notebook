# Comments
## Deprecation Warnings

```go
const (
    SomeObsoleteConstant = "Don't use me" // Deprecated: Use something else instead
)
```

# Concurrency
## Fetching Pages of Data Concurrently
```go
var mockedPageResponses = make(map[int]serviceResponse)

func init() {
    for i := 1; i <= 90; i++ {
        mockedPageResponses[i] = serviceResponse{
            data:    []Data{{Contents: fmt.Sprintf("%v", i), Page: i}},
            err:     nil,
            hasMore: true,
        }
    }
}

type Data struct {
    Contents string
    Page     int
}

type DataService interface {
    GetAllData() []Data
}

type dataService struct { }

func NewDataService() DataService{
    return &dataService{}
}

type serviceResponse struct {
    data    []Data
    err     error
    hasMore bool
}

func (svc dataService) GetAllData() (data []Data) {

    responseChan := make(chan serviceResponse)
    done := make(chan struct{})

    var wg sync.WaitGroup
    wg.Add(1)

    go func() {
        wg.Wait() // wait until all calls finish and then signal to indicate that we're all done
        done <- struct{}{}
    }()

    const batchSize = 10 // arbitrarily set, see comment below

    go func() {
        defer wg.Done()
        /*
            Because in this instance, we do not know how many total pages there are,
            the code will call ahead in batches to try to find the last page. For
            instance, if the page size is set to 10, it will call for page 10, then
            calls for pages 1 through 9 will be made concurrently. If page 10 has
            data, then attempt to find by the next batch size (page 20), and repeat.

            The tradeoff is that potentially multiple calls could be made that actually
            have no data. If the batch size is small, there would potentially be a
            smaller number of unnecessary calls, but at the sacrifice of less concurrent
            page fetches.
         */
        for step := batchSize; ; step += batchSize {
            res := fetchPage(step)
            responseChan <- res

            for i := step - 1; i > step-batchSize; i-- {
                wg.Add(1)
                go func(page int) {
                    defer wg.Done()
                    responseChan <- fetchPage(page)
                }(i)
            }

            if !res.hasMore {
                return
            }
        }
    }()

    // block here and read from responseChan or wait for the signal that everything
    // is finished from the `done` channel.
    for {
        select {
        case <-done:
            return // we're done... exit and return the data
        case res := <-responseChan:
            data = append(data, res.data...)
        }
    }
}

func fetchPage(pageNumber int) (res serviceResponse) {
    // service would be called here

    res, ok := mockedPageResponses[pageNumber]
    if !ok {
        return serviceResponse{
            data:    []Data{},
            err:     nil,
            hasMore: false,
        }
    }
    return
}
```
Test
```go
func TestDataService_GetAllData(t *testing.T) {

    svc := NewDataService()

    data := svc.GetAllData()

    assert.Len(t, data, 90)
}
```

## Protecting Against Data Race Conditions
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
