# Get Free Port in Go

```go
func GetFreePort() (string, error) {
	conn, err := net.Listen("tcp", "127.0.0.1:0")
	if err != nil {
		return "", err
	}
	defer conn.Close()
	sp := strings.Split(conn.Addr().String(), ":")
	if len(sp) < 2 {
		return "", errors.New("not exist port")
	}
	return sp[1], nil
}
```

#go #TCP #free_port