# SHA

SHA 해시는 내장 패키지에 구현되어 있습니다.

SHA1 패키지는 기존 SHA1 알고리즘이 구현되어 있고, SHA256과 SHA512는 SHA2의 구현체입니다.

```go
package main

import (
	"crypto/sha1"
	"crypto/sha256"
	"crypto/sha512"
	"fmt"
)

func main() {
	data := []byte("snowmerak")

	// SHA1
	fmt.Printf("%x\n", sha1.Sum(data))

	// SHA224
	fmt.Printf("%x\n", sha256.Sum224(data))

	// SHA256
	fmt.Printf("%x\n", sha256.Sum256(data))

	// SHA384
	fmt.Printf("%x\n", sha512.Sum384(data))

	// SHA512
	fmt.Printf("%x\n", sha512.Sum512(data))

	// SHA512_224
	fmt.Printf("%x\n", sha512.Sum512_224(data))

	// SHA512_256
	fmt.Printf("%x\n", sha512.Sum512_256(data))
}
```

```bash
6d3038d074cc5d25a7603446cc5dac3d5e7da1f3
fccee7bbd43727287264283c4eb8ec8e770540de2b0fef6277fdd3b1
302e31fa633b5f2a7be6f6699dde17ab59ad55bb3ad780740ccdfedbe7a32406
d60b787270c9073ffcd0c3a52563c7c3ace789074c5a4129722db11e486154af852aa850a7e0cbf124ad38755bf88f22
dda2ebf3ecdb5e9f74867878273b6ff49af8571489b0cbd519eb436380b16d68eb078c766293a697db3e66bccf81485010799cda1438b8a7fa60c45873841572
c1130d2c531d2d71a225c7c716df45f0d0f94d8c8ec9ea7b966245c6
9abbbad30f037ff6e96994cdcaee49f8d84dd56aa27a0de8a4359c5c27240f00
```

## SHA3

SHA3 패키지는 현재 `x`가 붙은 시범 패키지로 구현은 되어 있지만 정식으로 포함되진 않았으니 사용에 주의하셔야합니다.

```go
package main

import (
	"fmt"

	"golang.org/x/crypto/sha3"
)

func main() {
	data := []byte("snowmerak")

	// SHA3-225
	fmt.Printf("%x\n", sha3.Sum224(data))

	// SHA3-256
	fmt.Printf("%x\n", sha3.Sum256(data))

	// SHA3-384
	fmt.Printf("%x\n", sha3.Sum384(data))

	// SHA3-512
	fmt.Printf("%x\n", sha3.Sum512(data))
}
```

```bash
beda42e4e24efcf0de034f627a18c0cf6cb7d3e76aa6ccb3719b92d9
be591aa6ffe605e01c1d72b64c0328accb1939f43da36c6382b6f5d5821e03e2
931df31efd780360758b35a95c2fd18962641d2eea547a384e9c49b992e62f57f1bd719b88a527a99d1fb111b6266443
1a4d2422348eb3057639c1cd732286fc711198c756658da50c21e52c68612ee1b2e28416c7fc558c61b44944bb0abc65d3c7885e60481b7ce2c41acdeb5a47db
```

#SHA #SHA1 #SHA2 #SHA3 #SHA256 #SHA512

# FNV-1, FNV-1a

FNV는 Glenn Fowler, Landon Curt Noll, 그리고 Kiem-Phong Vo가 만든 암호학과 관련없는 해시 알고리즘입니다.

마찬가지로 표준 라이브러리에 제공됩니다. 그리고 암호학과 관련 없기 때문에 해싱이 빠릅니다.

```go
package main

import (
	"fmt"
	"hash/fnv"
)

func main() {
	data := []byte("snowmerak")

	// 32 bit FNV-1, FNV-1a hash
	fmt.Printf("%x\n", fnv.New32().Sum(data))
	fmt.Printf("%x\n", fnv.New32a().Sum(data))

	// 64 bit FNV-1, FNV-1a hash
	fmt.Printf("%x\n", fnv.New64().Sum(data))
	fmt.Printf("%x\n", fnv.New64a().Sum(data))

	// 128 bit FNV-1, FNV-1a hash
	fmt.Printf("%x\n", fnv.New128().Sum(data))
	fmt.Printf("%x\n", fnv.New128a().Sum(data))
}
```

```bash
736e6f776d6572616b811c9dc5
736e6f776d6572616b811c9dc5
736e6f776d6572616bcbf29ce484222325
736e6f776d6572616bcbf29ce484222325
736e6f776d6572616b6c62272e07bb014262b821756295c58d
736e6f776d6572616b6c62272e07bb014262b821756295c58d
```

#FNV-1 #FNV-1a

# blake2

`blake2` 알고리즘은 빠른 속도와 높은 보안성을 가진 해시 알고리즘입니다.

이 중, `blake2b`는 64비트에 최적화 되어 있고, 그 아래 비트에선 `blake2s`가 유리합니다.

`SHA3`와 마찬가지로 `X` 패키지에 들어가 있으니 충분한 테스트 후 사용하시기를 권장합니다.

```go
package main

import (
	"fmt"

	"golang.org/x/crypto/blake2b"
	"golang.org/x/crypto/blake2s"
)

func main() {
	data := []byte("snowmerak")

	// blake2b 256
	fmt.Printf("%x\n", blake2b.Sum256(data))

	// blake2b 384
	fmt.Printf("%x\n", blake2b.Sum384(data))

	// blake2b 512
	fmt.Printf("%x\n", blake2b.Sum512(data))

	// blake2s 256
	fmt.Printf("%x\n", blake2s.Sum256(data))
}
```

#blake2 #blake2b #blake2s

#go #hash