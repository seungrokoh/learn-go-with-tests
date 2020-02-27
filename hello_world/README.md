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

***
### 함수 인자 추가하기
위에서 작성한 함수를 이름을 입력받아 출력하는 형식으로 바꿔보면서 TDD를 진행해보자

첫 번째로 `hello_test.go` 파일에서 Test함수를 변경해보자

:heavy_check_mark: hello_test.go 수정
```go
package main

import "testing"

func TestHello(t *testing.T) {
    got := Hello("Chris")
    want := "Hello, Chris"

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```
위와 같이 `hello_test.go` 파일을 수정하고 `go test`를 진행하면 `Error`가 뜬다. 이 상황을 **RED** 라고 한다. 이제 실제 함수를 변경해보자

:heavy_check_mark: hello.go 수정
```go
package main

import "fmt"

func Hello(name string) string {
    return "Hello, " + name
}

func main() {
    fmt.Println(Hello("Chris"))
}
```
실제 함수를 변경해서 해당 Test가 `PASS` 하도록 변경할 수 있다. 이 상황을 **GREEN** 이라고 한다. TDD 프로세스에 따르면 이제 `Refactor`를 진행하면 된다.

`리팩토링`은 `"Hello, "` 부분을 상수로 변경해보자.

:heavy_check_mark: hello.go 리팩토링
```go
package main

import "fmt"

const englishHelloPrefix = "Hello, "

func Hello(name string) string {
    return englishHelloPrefix + name
}

func main() {
    fmt.Println(Hello("Chris"))
}
```

***
### 테스트 케이스 추가하기
새로운 테스트 케이스로 `Hello()` 함수의 인자값으로 `empty_string`이 왔을 때 `Hello, `가 아닌 `Hello, World`를 출력하도록 변경해보자.

:heavy_check_mark: hello_test.go 수정

```go
package main

import "testing"

func TestHello(t *testing.T) {
    t.Run("saying hello to people", func (t *testing.T) {
        got := Hello("Chris")
        want := "Hello, Chris"

        if got != want {
            t.Errorf("got %q want %q", got, want)
        }
    })

    t.Run("say 'Hello, World' when an empty string in supplied", func (t *testing.T) {
        got := Hello("")
        want := "Hello, World"

        if got != want {
            t.Errorf("got %q want %q", got, want)
        }
    })
}
```
위와 같이 테스트 코드를 수정한 뒤 `go test`를 진행해보면 `Error`가 뜬다. **RED** 상황이 발생했으므로 동작할 수 있도록 `hello.go`파일을 수정해보자.

:heavy_check_mark: hello.go 수정
```go
package main

import "fmt"

const englishHelloPrefix = "Hello, "

func Hello(name string) string {
    if name == "" {
        name = "World"
    }
    return englishHelloPrefix + name
}
```
단순하게 `empty_string`이 들어왔을 경우 기대값인 `World`로 변경하여 출력하도록 수정하였다. 이로써 **GREEN** 상황을 만들었으므로 **REFACTOR** 를 진행해보자

위의 `hello_test.go`파일에서 **에러를 출력하는 부분의 코드가 중복** 되는걸 볼 수 있다. 중복되는 부분을 제거하면서 `Refactoring`을 진행해보자.

:heavy_check_mark: hello_test.go 수정

```go
package main

import "testing"

func TestHello(t *testing.T) {

    assertCorrectMessage := func(t *testing.T, got, want string) {
        t.Helper()
        if got != want {
            t.Errorf("got %q want %q", got, want)
        }
    }

    t.Run("saying hello to people", func (t *testing.T) {
        got := Hello("Chris")
        want := "Hello, Chris"

        assertCorrectMessage(t, got, want)
    })

    t.Run("say 'Hello, World' when an empty string in supplied", func (t *testing.T) {
        got := Hello("")
        want := "Hello, World"

        assertCorrectMessage(t, got, want)
    })
}
```
에러 출력하는 부분을 함수화 시켜서 중복되는 코드는 줄였다. 여기서 `t.Helper()`라는 함수를 호출해주는데 `t.Helper()`를 호출해주는 이유는 **해당 메소드가 Helper임을 나타내며 오류가 발생했을 때 해당 함수가 호출된 곳의 `line number`를 알려주도록 하기 위함이다.**

***
