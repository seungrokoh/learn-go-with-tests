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

## WithDraw 기능 추가
이제 `Bitcoin`을 인출하는 `WithDraw` 함수를 추가로 작성해보자. 먼저 테스트 코드를 작성하고 통과할 수 있는 최소한의 코드를 작성해보자.

:heavy_check_mark: wallet_test.go
```go
package pointers_and_errors

import (
	"testing"
)

func TestWallet(t *testing.T) {

	t.Run("Deposit", func(t *testing.T) {
		wallet := Wallet{}
		wallet.Deposit(Bitcoin(10))

		got := wallet.Balance()
		want := Bitcoin(10)

		if got != want {
			t.Errorf("got %s want %s", got, want)
		}
	})

	t.Run("WithDraw", func(t *testing.T) {
		wallet := Wallet{balance: Bitcoin(20)}
		wallet.WithDraw(Bitcoin(10))

		got := wallet.Balance()
		want := Bitcoin(10)

		if got != want {
			t.Errorf("got %s want %s", got, want)
		}
	})
}
```

:heavy_check_mark: wallet.go
```go
// in wallet.go

// Something ...

// add WithDraw method
func (w *Wallet) WithDraw(amount Bitcoin) {
	w. balance -= amount
}
```

## Refactor
테스트 코드에서 중복되는 코드를 수정해보자.

:heavy_check_mark: wallet_test.go
```go
package pointers_and_errors

import (
	"testing"
)

func TestWallet(t *testing.T) {

	assertBalance := func(t *testing.T, wallet Wallet, want Bitcoin) {
		t.Helper()
		got := wallet.Balance()

		if got != want {
			t.Errorf("got %s want %s", got, want)
		}
	}

	t.Run("Deposit", func(t *testing.T) {
		wallet := Wallet{}
		wallet.Deposit(Bitcoin(10))
		assertBalance(t, wallet, Bitcoin(10))
	})

	t.Run("WithDraw", func(t *testing.T) {
		wallet := Wallet{balance: Bitcoin(20)}
		wallet.WithDraw(Bitcoin(10))
		assertBalance(t, wallet, Bitcoin(10))
	})
}
```
assertBalance 함수를 새로 생성하여 중복되는 코드를 제거할 수 있다. 여기서 문제점을 한 번 살펴보자. 만약 **`Wallet`에 남은 `Bitcoin`보다 더 많은 양의 `Bitcoin`을 인출하려 하면** 어떻게 될까?

예제에서 `초과 인출`은 없다고 가정하자. 초과 인출을 했을 때 알 수 있도록 코드를 수정해보자. Go 언어에서 오류를 표시하고 처리하  위해서 함수가 `err`을 반환하여 처리할 수 있도록 하는게 일반적인 방법이다. `wallet_test.go`에 새로운 테스트 코드를 작성하고 `WithDraw` 메서드를 수정해보자.

:heavy_check_mark: wallet_test.go 수정
```go
// in wallet_test.go

// Something ...

// add Test code
t.Run("Withdraw insufficient funds", func(t *testing.T) {
    startingBalance := Bitcoin(20)
    wallet := Wallet{startingBalance}
    err := wallet.WithDraw(Bitcoin(100))
    assertBalance(t, wallet, startingBalance)

    if err == nil {
        t.Error("wanted an error but didn't get one")
    }
})
```

:heavy_check_mark: wallet.go 수정
```go
// in wallet.go

// Something ...

// modify WithDraw
func (w *Wallet) WithDraw(amount Bitcoin) error {
	if w.balance < amount {
		return errors.New("oh no")
	}
	w.balance -= amount
	return nil
}
```
`errors.New(string)`은 메세지를 포함한 새로운 `error`를 생성한다.
