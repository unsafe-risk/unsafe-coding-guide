# Linux Backup using Tar

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
)

func main() {
	if len(os.Args) < 3 {
		fmt.Println("too few args.")
		return
	}
	rootName := os.Args[1]
	targetName := os.Args[2]

	args := []string{"tar", "cvpzf", targetName}
	excludeFlag := "--exclude="
	excludeNames := []string{targetName, "/proc", "/lost+found", "/media", "/mnt", "/sys", "/run", "/dev"}
	excludeNames = append(excludeNames, os.Args[3:]...)

	for _, s := range excludeNames {
		args = append(args, fmt.Sprintf("%s%s", excludeFlag, s))
	}

	cmd := exec.Command("sudo", args...)
	cmd.Args = append(cmd.Args, rootName)
	cmd.Stdout = os.Stdout
	cmd.Stdin = os.Stdin
	cmd.Stderr = os.Stderr
	err := cmd.Run()
	if err != nil {
		fmt.Println(err)
		return
	}
}
```

#go #linux #backup