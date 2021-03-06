# module

고랭은 예전에 하나의 사용자 당 하나의 `GOPATH`라는 작업 폴더를 주었고, 그 내부에서 모든 코드를 작성해야 했습니다. 그러나 고랭은 이후 특정 폴더마다 모듈을 생성하여 기존의 `GOPATH`를 대체하고 있습니다.

모듈을 생성할 폴더를 하나 임의로 만든 후, 터미널에서 해당 폴더로 이동하여 `go mod init practice`을 입력합니다. 그러면 폴더에 `go.mod` 파일이 생성되고, 그 파일에 모듈이 사용하는 고랭의 버전이나 의부 의존성을 기록합니다.

```go
module practice

go 1.18
```

`go.mod` 파일은 생성하자마자 열어봤을 경우, 위와같이 작성되어 있을 것입니다.

모듈을 사용하는 여러가지 명령어가 있습니다.
1. `go get`: 모듈에 새로운 의존성을 추가합니다. 이때 경로는 어떤 모듈의 경로가 됩니다. 예를 들어 `unsafe-risk`의 `go-safe` 레포를 받고 싶다면, `go get github.com/unsafe-risk/go-safe`을 하면 해당 경로에서 `go.mod`를 인식하여 현재 모듈에 추가합니다.
2. `go mod tidy`: 모듈에서 쓰지 않는 의존성을 제거합니다. 과거에 추가한 의존성이지만, 현재는 쓰이지 않는 의존성을 찾아내서 모듈에서 제거해줍니다.
3. `go install`: 모듈에서 쓰고 있지만 로컬 환경에 설치되지 않은 의존성을 다운로드합니다. 이 과정은 해당 모듈에 속한 프로젝트를 빌드(`go build`)하거나 실행(`go run`)하면 같이 실행됩니다.
4. `go install`: `go install`에는 다른 사용 방법도 있습니다. [`minica`](https://github.com/jsha/minica)라는 레포는 고랭으로 작성된 실행 프로그램으로 테스트 용 TLS 인증서를 만들어주는 모듈입니다. 원래라면 컴파일된 파일을 받아서 환경 변수에 등록해주어야 하지만, `go install github.com/jsha/minica@latest`을 실행함으로 고랭에서 공용으로 쓰는 `$GOPATH/bin`에 로컬에서 컴파일하여 실행 파일을 만들어 줍니다.  

## `$GOPATH/bin` 환경 변수에 대해

윈도우에서는 해당 경로가 환경 변수에 이미 등록되어 있지만, 리눅스와 맥에서는 `.bashrc`나 `.zshrc`같은 파일에 기록해주어야합니다. 이 경로는 앞으로 `protobuf`나 `grpc` 등에서도 쓰이니 고랭 개발을 한다면 등록해 두는 것이 좋습니다.

#go #module