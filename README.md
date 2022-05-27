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
      1. [type inference](README.md#type-inference)
   2. [float](README.md#float-and-double)
   3. [complex](README.md#complex)
   4. [string](README.md#string)
      1. [string and byte slice](README.md#string-and-byte-slice)
   5. [slice](README.md#slice)
      1. [slicing](README.md#slicing)
   6. [map](README.md#map)
      1. [set](README.md#set)
4. [error](README.md#error)

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

고랭은 인터페이스를 제외하면 명시적인 값 타입입니다. 파이썬이나 자바같이 생성된 변수가 암시적으로 간접 참조를 하는 경우는 없습니다. 만약 간접 참조를 하게 된다면 C나 C++에서 했던 것처럼 포인터 연산을 사용하여 명시적으로 하게 됩니다.

## integer

고랭에는 총 5개의 부호 있는 정수형과 총 5개의 부호 없는 정수형이 있습니다. 부호 있는 정수형은 `int8`, `int16`, `int32`, `int64`, `int`가 있고, 부호 없는 정수형은 각각의 앞에 `u`를 붙인 형태로 `uint8`, `uint16`, `uint32`, `uint64`, `uint`가 있습니다. 각각 8비트, 16비트, 32비트, 64비트의 크기를 가지는 부호 있는/부호 없는 정수형을 의미하고, 비트 수가 표기 되지 않은 `int`와 `uint`는 32비트 아키텍처에서는 32비트의 크기를, 64비트 아키텍처에서는 64비트의 크기를 가집니다.

### type inference

고랭에는 타입 추론이란게 있습니다. `var i = 32`라고 선언하면서 동시에 정의할 때, 고랭은 자동으로 이를 정수형이라고 판단합니다. 이때 정수형은 기본적으로 `int` 타입으로 타입이 추론됩니다. `int`는 위에서 작성했다시피 32비트 아키텍처와 64비트 아키텍처에서 각각 32비트, 64비트의 크기를 가지므로 대상 아키텍처에 주의해서 사용해야합니다.

고랭의 상수는 매우 큰 크기(`256bit`)를 가지고 있습니다. 비록 컴파일 타임에 모두 계산되어 사라지지만, 때에 따라서 자동으로 추론되는 타입에 의해 값이 유실되어 저장될 것입니다. 혹은 8비트의 공간만 있어도 되지만 32비트나 64비트의 공간을 사용하여 메모리를 낭비하게 될 수도 있습니다. 그럴 때는 상수에 형변환(`type conversion`)을 적용하여 타입 추론을 원하는 방향으로 적용할 수 있습니다. `var i8 = int8(32)`로 선언 및 정의하게 되면 `int8` 타입으로 추론됩니다.

## float and double

고랭의 실수형은 단 두 가지만 존재합니다. 32비트 크기를 가지는 단정밀도 부동소수점인 `float32`와 64비트 크기를 가지는 배정밀도 부동소수점인 `float64`입니다. `float64`는 다른 언어에서 `double`이나 `Double`로도 많이 쓰입니다. 부동소수점 타입은 타입 추론 시, `float64`가 기본으로 추론됩니다. 

```go
var f = 3.141592 // float64
```

## complex

고랭은 복소수 타입을 지원합니다. 32비트 부동소수점 2개를 각각 실수부, 허수부로 가지는 총 64비트 크기의 `complex64`와 64비트 부동소수점 2개를 실수부, 허수부로 가지는 128비트 크기의 `complex128`이 존재합니다. 정의는 `var c = 3 + 2i`같이 실수부 + 허수부i로 이루어집니다. 이때 타입 추론은 부동소수점이 64비트로 추론되었듯이 `complex128`로 추론됩니다.

## slice

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

고랭의 슬라이스는 위처럼 베이스가 되는 배열(`array`), 배열의 전체 크기(`cap`), 배열에 들어가 있는 원소의 수(`len`)로 이루어져 있습니다. 일반적인 배열과 유사하지만 `append()` 함수를 사용하여 정확한 슬라이스의 크기를 모르더라도 원소를 계속 추가할 수 있습니다.

```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3}
	a = append(a, 4, 5, 6)
	a = append(a, 7, 8, 9)
	fmt.Println(a)
	fmt.Println(len(a))
	fmt.Println(cap(a))
}
```

[위 코드](https://go.dev/play/p/_FQIWgOtyca)를 실행하게 되면 슬라이스에서는 총 9개의 원소(`[1 2 3 4 5 6 7 8 9]`가 들어가 있는 걸 확인할 수 있고, 배열에 속한 원소의 수가 9(`len(a)`), 배열의 전체 길이가 12(`cap(a)`)인 것을 확인할 수 있습니다. 이는 `append()` 함수가 입력받은 슬라이스의 길이가 추가할 원소들을 수용할 수 없게 되면, 자동으로 더 큰 길이의 슬라이스를 생성하여, 기존 배열의 원소들을 복사하고 추가할 원소들을 추가하기 때문입니다.

이 때 이전 슬라이스의 길이가 256 미만(`go 1.18 기준`)이라면 2배를 하게 되고, 그 이상이라면 1.25배를 하게 됩니다.

### slicing

슬라이스는 배열과 동일하게 인덱스를 통한 접근이 가능하며, 인덱스를 이용해 슬라이싱이란 작업도 가능합니다. 슬라이싱은 전체 슬라이스에서 일부분만 자르는 것으로, 기반이 된 슬라이스의 일부를 가리키는 새로운 슬라이스를 생성합니다.

```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	a = a[3:6]
	fmt.Println(a)
	fmt.Println(len(a))
	fmt.Println(cap(a))
}
```

[위 코드](https://go.dev/play/p/KLqGYiWAlzD)를 실행하게 되면 결과는 `[4 5 6]`이고, 원소 수는 3, 전체 배열의 용량은 7로 나오는 것을 확인할 수 있습니다. 이는 기반이 된 슬라이스의 기반 배열(`[1 2 3 4 5 6 7 8 9 10]`)을 그대로 가져온 후, 인덱스 3을 배열의 시작점으로 잡았을 뿐이기 때문입니다.

이렇게 슬라이싱된 슬라이스는 이전 슬라이스와 기반 배열을 공유하고 있기 때문에 `append()` 함수 등을 이용하여 작업을 하지 않는다면 데이터를 오염시킬 우려가 있습니다.

```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	b := a[3:6]
	b[0] = 100
	fmt.Println(a)
}
```

[위 코드](https://go.dev/play/p/WtX5_xffRlN)를 실행하면 수정한 건 `b` 슬라이스지만, `a` 슬라이스를 출력했을 때 4가 100으로 바뀐 결과(`[1 2 3 100 5 6 7 8 9 10]`)가 출력되는 걸 확인할 수 있습니다. 이 문제는 `append()` 함수를 사용할 때도 문제가 될 수 있습니다.

```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	b := a[3:6]
	b = append(b, 100)
	fmt.Println(a)
}
```

[위 코드](https://go.dev/play/p/N25psKOtRUP)를 실행하면 `b`에 `append()` 함수로 100을 추가했는데 `a` 슬라이스의 7번째 원소가 100으로 바뀐 것(`[1 2 3 4 5 6 100 8 9 10]`을 확인할 수 있습니다. 이 이유는 `b` 슬라이스의 용량이 7이기에 `append()` 함수가 여유 공간이 있으니 용량을 하나 땡겨서 100을 추가한 것입니다. 이 문제는 비교적 쉽게 해결할 수 있습니다. 

```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	b := a[3:6:6]
	b = append(b, 100)
	fmt.Println(a)
	fmt.Println(b)
	fmt.Println(len(b))
	fmt.Println(cap(b))
}
```

[위 코드](https://go.dev/play/p/gAzODvNIZX8)는 `a` 슬라이스를 슬라이싱할 때 전체 용량을 지정합니다. 기반 배열에서 인덱스 3부터 인덱스 6까지 잡으면서, 슬라이싱할 전체 용량은 인덱스 0부터 인덱스 6까지이니 6으로 지정해줍니다. 그러면 기반 배열의 뒤쪽 원소나 공간에 상관없이 전체 용량이 6인 슬라이스가 됩니다. 이후 `b` 슬라이스에 `append()` 함수를 호출하면, `b` 슬라이스의 용량은 가득찬 걸로 되어 `a` 슬라이스의 값(`[1 2 3 4 5 6 7 8 9 10]`)과 `b` 슬라이스의 결과 값(`[4 5 6 100]`)은 독립적이게 됩니다.

### copy

```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	b := make([]int, 3)
	copy(b, a[3:6])
	b = append(b, 100)
	fmt.Println(a)
	fmt.Println(b)
	fmt.Println(len(b))
	fmt.Println(cap(b))
}
```

위의 용량을 동반한 슬라이싱과 다르게 `copy(dst, src)` 함수를 사용하게 되면, 간편하게 새로운 슬라이스에 슬라이싱한 값들을 복사할 수 있습니다. `copy(dst, src)` 함수는 2개의 매개변수, 복사한 값이 저장될 슬라이스와 복사할 값이 저장된 슬라이스를 받습니다. 후자의 슬라이스의 값들을 전자의 슬라이스에 저장하게 되어 `b`는 `copy()`를 호출한 시점에 길이 3, 용량 3의 별도의 슬라이스가 되어, 기존 슬라이스에 영향을 끼칠 수 없게 됩니다.

`copy(dst, src)` 함수는 `src` 슬라이스의 원소 수가 `dst` 슬라이스의 길이를 초과할 경우, `dst` 슬라이스의 길이에 맞춰서 `src` 슬라이스의 원소를 복사합니다. 

## string

고랭의 문자열은 C 언어의 그것과는 다르게 동작합니다. C 언어의 문자열은 단순 `char` 배열로 이루어지며, 문자열 마지막에 널(`0`)이 들어가서 마지막임을 표시하는 정도였습니다.

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```

위 구조체는 실제 고랭(`go 1.18 기준`)에서 사용하는 문자열 구조체입니다. 슬라이스와 비슷한 형태를 하고 있지만, 배열의 전체 용량이 존재하지 않습니다. 그 이유는 고랭의 문자열은 다른 현대 언어가 으레 그렇듯 불변이기 때문입니다. `str` 멤버는 문자열이 저장된 `byte` 배열을 가리키고, `len` 멤버는 그 배열의 길이를 저장합니다. 

### string and byte slice

고랭은 문자열을 `byte` 배열로 저장하고 있기 때문에, `byte` 슬라이스와 `cap` 변수가 있냐 없냐의 차이만 있을 뿐 거의 동일함을 알 수 있습니다. 하지만 이 때문에 고랭에서 문자열을 바이트 슬라이스로 바꾸게 되면 문자열 내부 배열을 전부 복사하게 됩니다. 그래서 고퍼들은 문자열을 바이트 슬라이스로 만들 때, 복사가 발생하지 않도록 적지 않은 시도를 했고, 몇가지 코드들이 남았습니다.

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	a := "test string"
	b := unsafe.Slice((*byte)(unsafe.Pointer((*reflect.StringHeader)(unsafe.Pointer(&a)).Data)), len(a))
	fmt.Println(b)
}
```

[이 코드](https://go.dev/play/p/KAZ5iIfjHic)는 문자열을 로우 포인터로 만들어 강제로 문자열 헤더에 대입하고, 헤더의 데이터(위 문자열 헤더의 `str`에 해당하는 바이트 배열) 멤버를 문자열 길이 만큼의 슬라이스로 재구성 해주는 코드입니다. 

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	a := "test string"
	b := make([]byte, 0)
	(*reflect.SliceHeader)(unsafe.Pointer(&b)).Data = (*reflect.StringHeader)(unsafe.Pointer(&a)).Data
	(*reflect.SliceHeader)(unsafe.Pointer(&b)).Len = len(a)
	(*reflect.SliceHeader)(unsafe.Pointer(&b)).Cap = len(a)
	fmt.Println(b)
}
```

[다음 코드](https://go.dev/play/p/t-LwqO4yhkE)는 이전 코드와 비슷하게 문자열의 데이터 멤버를 가져옵니다. 다른 점은 미리 바이트 슬라이스를 선언하고 해당 바이트 슬라이스를 슬라이스 헤더로 캐스팅하여 데이터를 대입합니다. 각 길이와 용량에는 문자열의 길이와 용량을 대입합니다. 

어느 방법을 사용하더라도 문자열은 불변이므로 바이트 슬라이스에서 수정하려고 하면 런타임 패닉이 발생하니 주의하여야합니다. 

## map

```go
// A header for a Go map.
type hmap struct {
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	extra *mapextra // optional fields
}
```

고랭의 맵은 다른 언어에서 해시맵(혹은 unordred_map)이라고 부르는 자료구조입니다. 이름 그대로 키값 쌍을 저장하고, 키를 이용해 값에 접근하는 해시테이블 자료구조입니다. 위 구조체는 실제 고랭(`go 1.18 기준`)에서 사용 중인 해시맵 내부 구조체입니다. 이 중 `buckets` 멤버는 해시테이블을 저장하는 포인터입니다. 해시테이블은 리스트로 되어 있으며, 길이는 2^n을 유지하고, 하나의 버킷 당 8개의 키값 쌍을 저장합니다. 그리고 각 버킷에 평균적으로 6.5개 이상의 키값 쌍이 저장되면 자동으로 전체 크기를 2배 늘리고, 키값 쌍을 분산시킵니다.

```go
package main

import "fmt"

func main() {
	a := map[int]string{}
	a[1] = "abc"
	a[2] = "bbb"
	fmt.Println(a)
}
```

고랭의 맵은 `map[K]V`로 표현하고, `K`와 `V`는 각각 키와 값의 타입을 표현합니다. [위 코드](https://go.dev/play/p/V0fwx2eTv7C)는 `int` 타입을 키로 가지고 `string` 타입을 값으로 가지는 맵을 선언합니다. 선언 후에는 변수에 `[]` 연산을 사용하여 값을 추가, 설정 하거나 접근할 수 있습니다.

```go
package main

import "fmt"

func main() {
	a := map[int]string{}
	a[1] = "abc"
	a[2] = "bbb"

	delete(a, 1)
	fmt.Println(a)
}
```

맵에서 특정 원소를 삭제할 때는 `delete(map, key)` 함수를 사용합니다. 입력받은 맵 변수에서 입력받은 키를 찾아서 제거합니다. 슬라이스와 달리 맵은 이런 일련의 추가, 삭제 과정에서 내부 `buckets` 멤버가 배열의 포인터를 간접 참조하여 잡고 있으므로, 새로 맵 변수를 할당할 필요는 없습니다. 그리고 `delete()` 함수는 입력받은 맵 변수에 키가 존재하지 않는다면 동작을 하지 않고 무시할 것이기 때문에, 조건문으로 키 존재 유무를 확인할 필요가 없습니다.

```go
package main

import "fmt"

func main() {
	a := map[int]string{}
	a[1] = "abc"

	v, ok := a[1]
	fmt.Println(v, ok)

	v, ok = a[2]
	fmt.Println(v, ok)
}
```

[이 코드](https://go.dev/play/p/L3SiAhegj1P)는 맵에서 특정 키의 값을 받아오는 중, 키에 해당하는 값이 존재하는 지 확인하는 코드입니다. 맵의 키에 접근할 때 일반적으로는 무시하고 받지 않을 수 있지만, 의도적으로 두번째 반환값을 받게 되면, 해당 키값 쌍이 맵에 저장되어 있는 지 확인할 수 있습니다. 위 코드의 출력값으로 첫번째 `ok`는 `true`를, 두번째 `ok`는 `false`를 출력합니다. 그리고 존재하지 않는 키값 쌍의 값 반환은 해당 타입의 제로 값이 됩니다.

### set

고랭에는 맵 자료구조는 있지만 셋 자료구조는 없습니다. 하지만 당연하게도 B tree 맵도 값만 저장하지 않으면 B tree 셋으로 쓸 수 있는 것과 같은 느낌으로, 고랭의 맵에서 값을 `bool`로 저장하거나 저장하지 않는 방법으로 셋으로 사용할 수 있습니다.

```go
package main

import "fmt"

func main() {
	a := map[int]bool{}
	a[1] = true
	fmt.Println(a[1])
	fmt.Println(a[2])
}
```

[이 코드](https://go.dev/play/p/WhXRvTY758m)는 `int` 타입을 키로, `bool` 타입을 값으로 가지는 맵을 선언하고 사용하는 코드입니다. `1`은 미리 `true`를 값으로 가지는 키값 쌍을 입력하고 호출하기에, 당연히 `true`가 나옵니다. `2`는 존재하지 않기에 `bool`의 제로값인 `false`가 나옵니다. 단순 조건식으로 사용하면 `if a[1] {}`으로 사용합니다. 이 방식은 키를 넣을 때, 값으로 `true`를 넣지 않으면 의도한 대로 동작하지 않을 수 있다는 단점이 있습니다.

```go
package main

import "fmt"

func main() {
	a := map[int]struct{}{}
	a[1] = struct{}{}
	_, ok := a[1]
	fmt.Println(ok)
	_, ok = a[2]
	fmt.Println(ok)
}
```

[이 코드](https://go.dev/play/p/hkpjgnW07DK)는 `struct{}`를 활용합니다. 이 구조체는 빈 구조체라고 부르며, 그 어떤 멤버도 가지고 있지 않기 때문에 메모리에서 차지하는 크기가 `0`입니다. 이렇게 만든 맵은 키의 크기 만큼만 메모리를 차지하게 되어 `bool`보다 8 비트씩 공간을 덜 차지합니다. 대신 `bool`을 값의 타입으로 가질 때와 달리 두번째 반환값인 `ok`를 활용해야해서 코드가 아주 조금 늘어난다는 단점이 있습니다.

# error

