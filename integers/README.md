# Integers

두 정수를 받아 합을 출력하는 함수를 만들고 테스트 해보자. 테스트를 위해 `adder_test.go.` 파일을 생성해보자.

:bulb: Go 소스 파일은 디렉토리 당 **하나의 패키지 만 가질 수 있다.**

:heavy_check_mark: adder_test.go
```go
package integers

import "testing"

func TestAdder(t *testing.T) {
    sum := Add(2, 2)
    expected := 4

    if sum != expected {
        t.Errorf("expected '%d' but got '%d'", expected, sum)
    }
}
```
> 문자열이 아닌 정수를 출력하기 위해 %d를 사용한다.

위와 같이 Test 함수를 만들고 `go test`를 진행해보면 `undefined:Add` Error가 발생한다. 이제 `Add(int, int)` 함수를 만들기 위해 `adder.go` 파일을 생성하자.

:heavy_check_mark: adder.go
```go
package integers

func Add(x, y int) int {
    return x + y
}
```
Test를 통과할 수 있는 최소한의 코드를 작성하여 Test가 **PASS** 할 수 있도록 만든다.

이제 `Add()` 함수를 따로 테스트 하기 위해서 `ExampleAdd()` 함수를 작성해보자.

:heavy_check_mark: adder_test.go 수정
```go
package integers

import (
    "fmt"
    "testing"
    )

func TestAdder(t *testing.T) {
    sum := Add(2, 2)
    expected := 4

    if sum != expected {
        t.Errorf("expected '%d' but got '%d'", expected, sum)
    }
}

func ExampleAdd() {
    sum := Add(1, 5)
    fmt.Println(sum)
    // Output: 6
}
```
추가적인 테스트 함수는 `ExampleXXX()` 형식으로 작성하면 된다. `fmt`를 통해 `Add()` 함수를 출력하고 그 밑에 주석으로 기대값인 `Output`을 적어주면 된다.

이제 준비가 됐다면 `go test -v`로 실행 시켜보면 테스트 결과가 화면에 표시된다.

:bulb: **// Output:** 형식을 무조건 지켜야 한다! 띄어쓰기 하나라도 틀릴 시 동작하지 않는다.
