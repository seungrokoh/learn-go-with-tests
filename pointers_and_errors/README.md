# Pointers & errors
Go 언어를 이용해 간단한 `Bitcoin`을 관리하는 시스템을 만들어보자.

먼저 `Bitcoin`을 담을 수 있는 `Wallet`을 테스트 하는 코드를 작성해보자.

:heavy_check_mark: wallet_test.go
```go
package pointers_and_errors

import "testing"

func TestWallet(t *testing.T) {
	wallet := Wallet{}
	wallet.Deposit(10)

	got := wallet.Balance()
	want := 10

	if got != want {
		t.Errorf("got %d want %d", got, want)
	}
}
```
이제 테스트 코드를 통과할 수 있는 최소한의 코드를 작성해보자.

:heavy_check_mark: wallet.go
```go
package pointers_and_errors

type Wallet struct {
    balance int
}

func (w Wallet) Deposit(amount int) {
    w.balance += amount
}

func (w Wallet) Balance() int {
    return w.balance
}
```
위와 같이 작성하고 테스트 코드를 동작시키면 `PASS`가 나올 것 같지만 `FAIL`이 떨어진다. 그 이유에 대해 살펴보자.

Go 언어에서 함수나 메서드를 호출하면 **인자가 복사가 된다.** 테스트 코드와 `Deposit` 메서드에서 balance의 주소값을 출력해보면 이해할 수 있다. 따라서 이 문제를 해결하기 위해선 `Pointer`를 사용해야 한다. 즉, 주소 값을 넘겨주어야 한다.

`type`을 정의하는 방식은 아래와 같다.

```go
type MyName OriginalType
```

## Refactor
위에서 언급한 Pointer에 관하여 `Refactoring`을 진행한다. 또한 `Bitcoin`에 대해 작성하고 있지만 위의 코드는 `int`를 사용하고 있어 어떤 것을 구현하는지 쉽게 이해할 수 없다. `bitcoin`을 `struct`로 만들 수도 있지만 이보다 `type`을 이용해 Bitcoin을 만들어보자.

:heavy_check_mark: wallet.go 수정
```go
package pointers_and_errors

type Bitcoin int

type Wallet struct {
	balance Bitcoin
}

func (w *Wallet) Deposit(amount Bitcoin) {
	w.balance += amount
}

func (w *Wallet) Balance() Bitcoin {
	return w.balance
}
```

:heavy_check_mark: wallet_test.go 수정
```go
package pointers_and_errors

import (
	"testing"
)

func TestWallet(t *testing.T) {
	wallet := Wallet{}
	wallet.Deposit(Bitcoin(10))

	got := wallet.Balance()
	want := Bitcoin(10)

	if got != want {
		t.Errorf("got %d want %d", got, want)
	}
}
```

새로운 `type`을 이용해 새로운 유형을 만들고 **메서드를 작성할 수 있다.** 기존 유형에 도메인 별 기능을 추가하고 싶을 때 매우 유용하다.

이제 `Bitcoin`에 `Stringer interface`를 구현해보자. `Stringer interface`는 아래와 같이 구현되어 있다.

```go
type Stringer interface {
    String() string
}
```

:heavy_check_mark: implement Stringer on Bitcoin
```go
// in wallet.go

func (b Bitcoin) String() string {
    return fmt.Sprintf("%d BTC", b)
}
```
`Stringer` 인터페이스는 `fmt` 패키지에 정의되어 있으며 출력할 때 `%s`을 이용하여 출력하는 방식을 정의할 수 있다. 에러 출력부분을 수정해보고 출력해보자.

```go
if got != want {
    t.Errorf("got %s want %s", got, want)
}
```
