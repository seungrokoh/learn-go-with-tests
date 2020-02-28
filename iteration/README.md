# Iteration

반복문을 사용하여 반복 작업을 진행해 보자. Go 언어는 `while`, `do`, `until`이 없다.  
반복작업을 위해서 오로지 `for`만을 사용해야 한다.

반복문을 이용해 `a` 5개를 이어붙인 `repeat_test.go` 함수를 만들고 테스트 해보자.

:heavy_check_mark: repeat_test.go
```go
package iteration

import "testing"

func TestRepeat(t *testing.T) {
    repeated := Repeat("a")
    expected := "aaaaa"

    if repeated != expected {
        t.Errorf("expected %q but got %q", expected, repeated)
    }
}
```
테스트 함수를 만들고 `go test`를 진행해보면 `Repeat()` 함수가 없어 `FAIL`이 난다. **(RED)**  
테스트를 통과하기 위해 `repeat.go` 파일을 만들고 `Repeat()` 함수를 작성하자.

:bulb: 본문에서는 먼저 테스트가 `FAIL`나기 위한 최소한의 함수를 작성 (아무 동작하지 않는 함수)를 만든 뒤 `GREEN` 상태로 넘어가기 위한 최소한의 코드를 작성하지만, 그 단계를 뛰어넘고 바로 `GREEN` 상태로 가기 위한 최소한의 코드를 작성해보자.

:heavy_check_mark: repeat.go
```go
package iteration

func Repeat(character string) string {
    var repeated string
    for i := 0; i < 5; i++ {
        repeated = repeated + character
    }
    return repeated
}
```
`Repeat()` 함수를 작성하고 `go test`를 진행하면 `PASS`가 뜬다. **(GREEN)** 이제 함수를 `REFACTOR` 해보자.

:heavy_check_mark: refactoring repeat.go
```go
package iteration

const repeatCount = 5

func Repeat(character string) string {
    var repeated string
    for i := 0; i < repeatCount; i++ {
        repeated += character
    }
    return repeated
}
```
