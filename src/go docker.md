# golang docker binding

```go
package main

import (
	"bufio"
	"context"
	"fmt"
	"io"
	"os"
	"os/signal"
	"syscall"

	"github.com/docker/docker/api/types"
	"github.com/docker/docker/api/types/container"
	"github.com/docker/docker/client"
	"github.com/docker/go-connections/nat"
)

func main() {
	ctx := context.Background()
	cli, err := client.NewClientWithOpts(client.FromEnv, client.WithAPIVersionNegotiation())
	if err != nil {
		panic(err)
	}

	reader, err := cli.ImagePull(ctx, "docker.io/yeasy/simple-web", types.ImagePullOptions{})
	if err != nil {
		panic(err)
	}
	defer reader.Close()
	io.Copy(os.Stdout, reader)

	hostConfig := &container.HostConfig{
		AutoRemove: true,
		PortBindings: nat.PortMap{
			"80/tcp": []nat.PortBinding{
				{
					HostIP:   "0.0.0.0",
					HostPort: "8080",
				},
			},
		},
	}

	resp, err := cli.ContainerCreate(ctx, &container.Config{
		Image: "yeasy/simple-web",
		ExposedPorts: nat.PortSet{
			"80/tcp": struct{}{},
		},
		Tty: false,
	}, hostConfig, nil, nil, "")
	if err != nil {
		panic(err)
	}

	if err := cli.ContainerStart(ctx, resp.ID, types.ContainerStartOptions{}); err != nil {
		panic(err)
	}

	out, err := cli.ContainerLogs(ctx, resp.ID, types.ContainerLogsOptions{ShowStdout: true})
	if err != nil {
		panic(err)
	}

	go func() {
		for {
			scanner := bufio.NewReader(out)
			line, _, err := scanner.ReadLine()
			if err != nil {
				if err == io.EOF {
					break
				}
				panic(err)
			}
			fmt.Printf("[%s-%s]: %s\n", "simple-web", resp.ID[:10], string(line[8:]))
		}
	}()

	terminalChan := make(chan os.Signal, 1)
	signal.Notify(terminalChan, os.Interrupt, os.Signal(syscall.SIGTERM))
	<-terminalChan

	if err := cli.ContainerStop(ctx, resp.ID, nil); err != nil {
		panic(err)
	}
}
```

#go #docker