# Mocking

`3, 2, 1, Go!`를 출력하는 프로그램을 만들면서 `Mocking`에 대해서 살펴보자. 프로그램은 매우 간단한 것 같지만 `TDD` 프로세스를 따르면서 개발을 해야한다.  

:bulb: **중요한 점**  
요구 사항을 **최대한 작게 분할하여** 작동하는 소프트웨어를 가질 수 있게 하는 것은 매우 중요한 기술이다.

위 예제에서 작업을 나누고 반복하는 방법은 다음과 같다.

* 3 출력
* 3, 2, 1, Go! 출력
* 각 출력 사이에 1초간 딜레이 주기

먼저 테스트 코드를 작성해보자.

:heavy_check_mark: countdown_test.go
```go
package main

import (
	"bytes"
	"testing"
)

func TestCountdown(t *testing.T) {
	buffer := &bytes.Buffer{}

	Countdown(buffer)

	got := buffer.String()
	want := "3"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```
* `main` 함수에서 `os.Stdout`으로 보내 `Terminal`에서 데이터를 출력한다.
* `test code` 에서 `bytes.Buffer`로 보내 테스트에서 생성되는 데이터를 캡쳐할 수 있다.

이제 `Countdown` 함수를 만들어보자.

:heavy_check_mark: main.go
```go
package main

import (
	"bytes"
	"fmt"
	"os"
)

func Countdown(out *bytes.Buffer) {
	fmt.Fprint(out, "3")
}
```

## Refactor

`Countdown` 함수에 있는 `*bytes.Buffer`를 `Interface`를 이용해 받아올 수 있도록 변경해보자.

:heavy_check_mark: Countdown 함수 수정
```go
func Countdown(out io.Writer) {
    fmt.Fprint(out, "3")
}
```

이제 테스트가 `PASS` 하는 것을 확인했다면 `main` 함수에서 실제로 동작하는 소프트웨어를 작성해보자.

:heavy_check_mark: main.go
```go
package main

import (
    "fmt"
    "io"
    "os"
)

func Countdown(out io.Writer) {
    fmt.Fprint(out, "3")
}

func main() {
    Countdown(os.Stdout)
}
```
이렇게 작성하는 것이 사소한 것처럼 보이지만 **접근법은 모든 프로젝트에 권장되는 것이다.**  

* 기능별로 최대한 작게 분리하고, 테스트를 통해 `end-to-end`로 작동하도록 만들자

이제 2, 1, Go! 를 출력해보자.
