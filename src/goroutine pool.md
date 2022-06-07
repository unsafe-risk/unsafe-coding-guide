# goroutine pool

## gopool

`go get github.com/snowmerak/gopool/v2`

```go
package main

import (
	"fmt"
	"time"

	"github.com/snowmerak/gopool/v2"
)

func main() {
	gp := gopool.New[int](10)
	defer gp.Close()
	future, err := gp.Go(func() gopool.Result[int] {
		time.Sleep(time.Second)
		return gopool.Result[int]{
			Result: 1,
			Err:    nil,
		}
	})
	if err != nil {
		panic(err)
	}
	result := <-future
	if result.Err != nil {
		panic(result.Err)
	}
	fmt.Println(result.Result)
	gp.Wait()
}
```

#gopool #go #goroutine