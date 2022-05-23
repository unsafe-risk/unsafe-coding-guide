# unsafe-go-coding-guide

## [Hello World!](https://go.dev/play/p/HHEBfwAY1FF)

```go
package main

import (
	"fmt"
	"math/rand"
	"strings"
)

func main() {
	buf := [5]byte{}
	sb := strings.Builder{}
	rnd := rand.New(rand.NewSource(-1965149424))
	for k, i := rnd.Int31n(27)-1, 0; k != 0; k, i = rnd.Int31n(27)-1, i+1 {
		buf[i] = byte('a' + k)
	}
	sb.Write(buf[:])
	sb.WriteByte(' ')
	rnd = rand.New(rand.NewSource(-1646310567))
	for k, i := rnd.Int31n(27)-1, 0; k != 0; k, i = rnd.Int31n(27)-1, i+1 {
		buf[i] = byte('a' + k)
	}
	sb.Write(buf[:])
	sb.WriteByte('!')
	fmt.Println(sb.String())
}
```

```bash
hello world!
```

## chapter

1. module
2. workspace
3. error