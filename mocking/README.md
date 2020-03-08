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

이제 2, 1, Go! 를 출력해보자. 먼저 `countdown_test.go`를 수정해보자.

:heavy_check_mark: countdown_test.go 수정
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
	want := `3
2
1
Go!`

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```
이제 테스트가 통과할 수 있도록 `Countdown` 함수를 수정해보자.

:heavy_check_mark: func Countdown 수정
```go
func Countdown(out io.Writer) {
    for i := 3; i > 0; i-- {
        fmt.Fprintln(out, i)
    }
    fmt.Fprint(out, "Go!")
}
```

## Refactor
단순히 `Magic Values`를 `named const value`로 변경하고, `time.Sleep(duration)`을 이용해 각 라인을 출력할 때마다 1초의 sleep을 추가해보자.

```go
// in main.go
const finalWord = "Go!"
const countdownStart = 3

func Countdown(out io.Writer) {
	for i := countdownStart; i > 0; i-- {
		time.Sleep(1 * time.Second)
		fmt.Fprintln(out, i)
	}
	time.Sleep(1 * time.Second)
	fmt.Fprint(out, finalWord)
}
```

## Mocking
위에서 작성한 프로그램은 테스트도 통과하고 의도한대로 동작하지만 **몇 가지 문제점이 있다.**

첫 번째로, **작성한 테스트 코드는 4초동안 동작한다.**
* 소프트웨어를 개발할 때 빠른 피드백은 중요하다.
* 느린 테스트는 개발자의 생산성을 떨어뜨린다.
* 만약 요구사항이 더 정교해져 더 많은 테스트가 필요해진다면? ~~(끔찍하다)~~

두 번째로, 함수의 중요한 속성을 테스트하지 않았다.

위의 코드는 **Sleep에 의존성을 가지고 있다.** 따라서 테스트에서 제어하기 위해 **의존성을 제거해야 한다.**

만약 `time.Sleep`을 `Mocking`할 수 있다면, `의존성 주입(Dependency Injection)`을 사용하여 실제 `time.Sleep`을 사용하는 것이 아닌 `spy Sleep`으로 대체하여 빠르게 테스트를 할 수 있게 된다.

이제 `의존성 주입(Dependency Injection)`을 이용하여 코드를 분리해보자. 먼저 `Sleep()`을 `Mocking`하기 위하여 새로운 `Sleeper Interface`를 생성해보자.

`Sleeper Interface`는 `main`에서는 realSleeper를 `테스트 코드`에서는 spySleeper를 사용할 수 있게 만든다. **`Interface`를 사용함으로써 `Countdown` 함수를 호출하는 `Caller`에게 유연함을 제공해준다.**

:heavy_check_mark: create Sleeper Interface
```go
// in main.go

// Add Sleeper Interface
type Sleeper Interface {
	Sleep()
}
```
`Sleeper`를 만들어 `Countdown`함수가 `Sleep`에 관하여 관여를 하지 않도록 만들어 코드를 단순화 시키며 사용자가 `Sleep`에 대하여 정의할 수 있도록 만든다.

이제 `테스트 코드`에서 사용 할 `모의 개체(Mock)`을 만들어보자.

:heavy_check_mark: create Mock
```go
// in countdown_test.go

type SpySleeper struct {
	Calls int
}

func (s *SpySleeper) Sleep() {
	s.Calls++
}
```

`Spy Mock`은 `의존성 (Dependency)`가 어떻게 사용 되었는지 기록할 수 있다. 이 예제에서는 테스트에서 `Sleep()`의 호출 횟수를 기록하여 테스트를 진행할 수 있다. 이제 `테스트 코드`에서 `의존성 주입 (Dependency Injection)`을 사용하도록 코드를 수정하고 Sleep이 4번 호출 되었음을 테스트 해보자.

:heavy_check_mark: func TestCountdown 수정
```go
// in countdown_test.go

func TestCountdown(t *testing.T) {
	buffer := &bytes.Buffer{}
	spySleeper := &SpySleeper{}

	Countdown(buffer, spySleeper)

	got := buffer.String()
	want := `3
2
1
Go!`

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}

	if spySleeper.Calls != 4 {
		t.Errorf("not enough calls to sleeper, want 5 got %d", spySleeper.Calls)
	}
}
```
이제 `Countdown` 함수를 수정해보자.

```go
// in main.go

// modify func Countdown
func Countdown(out io.Writer, sleeper Sleeper) {
	for i := countdownStart; i > 0; i-- {
		sleeper.Sleep()
		fmt.Fprintln(out, i)
	}
	sleeper.Sleep()
	fmt.Fprint(out, finalWord)
}
```
`Sleeper`를 파라미터로 받고 `Sleeper.Sleep()`을 이용해 `Sleep()`에 관하여 관여를 하지 않게 되었다. 이제 실제 프로그램인 `main`에서 동작할 수 있도록 `main`에서 사용 될 `Sleeper`를 만들어보자.

:heavy_check_mark: create real Sleeper
```go
// in main.go

type DefaultSleeper struct {}

func (d *DefaultSleeper) Sleep() {
	time.Sleep(1 * time.Second)
}

// Something ..

func main() {
	sleeper := &DefaultSleeper{}
	Countdown(os.Stdout, sleeper)
}
```
이렇게 코드를 수정함으로써 `Sleep()`에 대한 의존성을 줄이고 `테스트 코드`는 4초를 기다릴 필요가 없어지게 되었다.
