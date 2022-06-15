# QUIC echo sample code

## server

```go
package main

import (
	"context"
	"crypto/tls"
	"log"

	"github.com/lucas-clemente/quic-go"
)

func main() {
	certificate, err := tls.LoadX509KeyPair("localhost/cert.pem", "localhost/key.pem")
	if err != nil {
		log.Fatal(err)
	}
	lis, err := quic.ListenAddr("localhost:8080", &tls.Config{
		InsecureSkipVerify: true,
		NextProtos:         []string{"localhost"},
		ServerName:         "localhost",
		Certificates:       []tls.Certificate{certificate},
	}, nil)
	if err != nil {
		panic(err)
	}
	for {
		conn, err := lis.Accept(context.Background())
		if err != nil {
			panic(err)
		}
		go func() {
			stream, err := conn.AcceptStream(context.Background())
			if err != nil {
				panic(err)
			}
			defer stream.Close()
			buf := [4096]byte{}
			for {
				n, err := stream.Read(buf[:])
				if err != nil {
					log.Println(err)
					return
				}
				if _, err := stream.Write(buf[:n]); err != nil {
					log.Println(err)
					return
				}
			}
		}()
	}
}
```

## client

```go
package main

import (
	"context"
	"crypto/tls"
	"fmt"
	"time"

	"github.com/lucas-clemente/quic-go"
)

func main() {
	conn, err := quic.DialAddr("localhost:8080", &tls.Config{
		InsecureSkipVerify: true,
		NextProtos:         []string{"localhost"},
		ServerName:         "localhost",
	}, nil)
	if err != nil {
		panic(err)
	}
	stream, err := conn.OpenStreamSync(context.Background())
	if err != nil {
		panic(err)
	}
	defer stream.Close()
	buf := [4096]byte{}
	for {
		if err != nil {
			panic(err)
		}
		if _, err := stream.Write([]byte("hello, world!")); err != nil {
			panic(err)
		}
		n, err := stream.Read(buf[:])
		if err != nil {
			panic(err)
		}
		fmt.Println(buf[:n])
		time.Sleep(1 * time.Second)
	}
}
```

#go #QUIC #echo