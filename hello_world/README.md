# Hello, World TDD

`Hello, World`를 출력하는 프로그램을 만들어보면서 TDD를 진행해보자

:bulb: TDD 프로세스

1. 실패하는 Test 작성
2. Test를 Pass할 수 있는 최소한의 코드 작성
3. 리팩토링

> RED -> GREEN -> REFACTOR

***
### 함수 분리하기
먼저 `Hello, World`를 출력하는 함수를 `테스트`하기 위해 함수로 따로 분리해서 작성해보자. 해당 함수를 작성하는 파일은 `hello.go`로 작성한다.

:heavy_check_mark: hello.go
```go
package main

import "fmt"

func Hello() string {
    return "Hello, World"
}

func main() {
    fmt.Println(Hello())
}
```

이제 `hello.go`파일을 테스트 하기 위해 테스트 파일을 생성한다. 테스트 파일을 생성하는 규칙은 `[파일명]_test.go`이다. 따라서 `hello_test.go`파일을 생성한다.

:heavy_check_mark: hello_test.go
```go
package main

import "testing"

func TestHello(t *testing.T) {
    got := Hello()
    want := "Hello, World"

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```
`got`은 Test할 함수가 반환하는 값이며, `want`는 기대값을 나타낸다. 이에 위 코드는 Test값과 기대값이 다를 때 `에러를 반환`하는 코드이다.

테스트를 진행하고 싶다면 `go test` 명령어를 입력하면 된다.
