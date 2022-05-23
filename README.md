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

# chapter

1. [module](README.md#module)
   1. [$GOPATH/bin](README.md#gopathbin-환경-변수에-대해)
2. [workspace](README.md#workspace)
   1. [mono-repo](README.md#mono-repo)
3. [type](README.md#type)
   1. [integer](README.md#integer)
   2. [float](README.md#float-and-double)
   3. [string and bytes](README.md#string-and-bytes)
   4. [slice](README.md#slice)
   5. [map](README.md#map)
4. [error]()

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

# workspace

고랭은 하나의 `GOPATH`를 여러 모듈로 분리했지만, 이후 새로운 문제에 직면 했습니다. 하나의 모듈에서 같은 로컬에 존재하는 다른 모듈의 패키지를 불러올 때, `replace`같은 부가 동작이 필요하고, 하나의 작업 환경이라는 결속력이 부족했습니다.

다른 여러 이유도 있겠지만, 고랭은 여러 모듈을 하나의 폴더에서 관리할 수 있게 `workpsace`라는 개념을 도입했습니다. 워크스페이스를 생성하기 위해서, 워크스페이스로 사용할 폴더에서 모듈 때와 같이 터미널에서 `go work init`을 입력합니다.

워크스페이스를 생성할 때는 별도의 이름을 입력할 필요는 없습니다. 실행하면 `go.work`라는 파일이 생성될 것입니다. 해당 파일에는 워크스페이스에서 어떤 고랭 버전을 사용할지, 어떤 폴더의 모듈을 포함할지 작성합니다.

```go
go 1.18
```

생성하자마자 파일을 열게 되면 위처럼 고랭 버전만 작성되어 있습니다.

만약 하위 폴더에 `prac`이란 폴더를 생성하고 모듈을 생성하면 워크스페이스에 `use` 키워드를 사용하여 작성해주어야 워크스페이스의 일원으로 들어가게 됩니다.

```go
go 1.18

use ./prac
```

`prac` 폴더 내부에 어떤 이름으로 모듈을 생성했는지 상관없이 경로만 적어주면, 같은 워크스페이스의 다른 모듈에서 참조할 수 있습니다. 이렇게 나누게 되면 고랭에서 훨씬 간편하게 모노 레포를 만들 수 있습니다.

## mono-repo

모놀리틱 구조에서는 하나의 프로젝트가 하나의 아키텍처를 가지고, 자연스럽게 하나의 프로젝트가 하나의 깃 레포지토리를 가졌습니다. 이후 여러 프로젝트가 하나의 아키텍처를 이루는 MSA 구조가 되어서는 프로젝트를 분산하면서 깃 레포지토리도 분산시켜서, 하나의 아키텍처에 여러 깃 레포지토리를 가지게 되었습니다.

하지만 깃 레포지토리가 분산되면서, 서로의 코드를 재활용하기 어렵고, 참조하는 다른 프로젝트의 코드의 히스토리를 파헤치는데 생각보다 많은 시간과 노력이 들게 되고, 빌드와 배포에 있어 분산되어 있기에 기존보다 많은 노력과 단계가 필요했습니다.

그래서 아키텍처에 여러 프로젝트를 쓰는 건 유지하면서, 깃 레포지토리는 하나만 쓰는 모노 레포라는 개념이 나오게 됩니다. 고랭의 워크스페이스는 하위에 여러 모듈을 둘 수 있고, 각 모듈 마다 각기 다른 재사용 가능한 코드가 들어간 프로젝트나 실제 돌아가는 프로젝트를 두어 모노 레포를 구성할 수 있습니다.

# type

## integer

## float and double

## string and bytes

## slice

## map