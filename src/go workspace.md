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

#go #workspace #monorepo