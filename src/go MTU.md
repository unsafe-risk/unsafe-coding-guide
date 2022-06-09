# Change MTU in go

```go
package main

import (
	"fmt"
	"net"
	"os"
	"os/exec"
)

func main() {
	netInterfaces, err := net.Interfaces()
	if err != nil {
		fmt.Println(err)
		return
	}
	var mtu string
	if len(os.Args) < 2 || os.Args[1] != "on" {
		mtu = "1500"
	} else {
		mtu = "400"
	}
	for _, v := range netInterfaces[1:] {
		if err := exec.Command("sudo", "ip", "link", "set", v.Name, "mtu", mtu).Run(); err != nil {
			fmt.Println(err)
		}
	}
}
```

#go #MTU