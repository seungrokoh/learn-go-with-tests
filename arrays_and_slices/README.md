# Arrays and slices

주어진 배열을 가지고 Test를 진행해보자.  
배열에 저장된 원소들의 총 합을 구하는 `Sum()` 함수를 테스트하는 코드를 작성해보자.

:heavy_check_mark: sum_test.go
```go
package main

import "testing"

func TestSum(t *testing.T) {
    numbers := [5]int{1, 2, 3, 4, 5}

    got := Sum(numbers)
    want := 15

    if got != want {
        t.Errorf("got %d want %d given, %v", got, want, numbers)
    }
}
```

:heavy_check_mark: sum.go
```go
package main

func Sum(numbers [5]int) int {
    sum := 0
    for i := 0; i < 5; i++ {
        sum += numbers[i]
    }
    return sum
}
```
테스트를 통과할 수 있는 최소한의 코드를 작성한다. 이제 `REFACTOR`를 거치면서 코드를 다듬어보자.

첫 번째로, `for`문을 조금 변경해보자. `range`를 이용해 `slice`의 범위만큼 접근할 수 있다.

:heavy_check_mark: sum.go 수정
```go
package main

func Sum(numbers [5]int) int {
    sum := 0
    for _, number := range numbers {
        sum += number
    }
    return sum
}
```

`Sum()`함수가 받는 인자를 보면 `고정된 길이의 배열`을 인자로 받는걸 볼 수 있다. `Go` 언어에서 **크기가 다른 배열은 서로 다른 타입을 가진다.** 즉, `[5]int`와 `[4]int`는 타입이 다르기 때문에 위의 함수에 `[4]int`를 넘겨주는 것은 `string`을 넘겨주는 것과 같이 **완전 다른 타입을 넘겨주는 것** 이므로 컴파일이 되지 않는다. 따라서 다양한 배열의 `Sum()` 함수를 테스트 하기 위해서 `REFACTOR` 해보자.

:heavy_check_mark: sum_test.go
```go
package main

import "testing"

func TestSum(t *testing.T) {

    t.Run("collection of 5 numbers" , func(t *testing.T) {
        numbers := []int{1, 2, 3, 4, 5}
        got := Sum(numbers)
        want := 15

        if got != want {
            t.Errorf("got %d want %d given %v", got, want, numbers)
        }
    })

    t.Run("collection of any size", func (t *testing.T) {
        numbers := []int{1, 2, 3}
        got := Sum(numbers)
        want := 6

        if got != want {
            t.Errorf("got %d want %d given %v", got, want, numbers)
        }
    })
}
```
:heavy_check_mark: sum.go
```go
package main

func Sum(numbers []int) int {
    sum := 0
    for _, number := range numbers {
        sum += number
    }
    return sum
}
```
### Coverage Tool
테스트를 작성하면서 `테스트의 가치`에 대해 생각해보는 것은 중요하다. 테스트를 작성하는 것은 중요하지만 **필요 이상으로 너무 많은 테스트를 작성하면 실제 코드에 문제가 발생할 수 있으며, 유지 관리에 오버헤드 또한 크다.** 따라서, 너무 많은 테스트 코드를 작성하는 것에 중점이 아닌 **기존 코드에 대한 자신감을 가져야 한다.** ~~(너무 많은 테스트 코드를 작성하는 것은 독이 될 수 있다라는 뜻 같음)~~

Go 에서 제공하는 Toolkit 중 하나인 `coverage`에 대해서 알아보자. `cover`는 작성한 테스트코드가 다루지 않는 영역을 판별하는데 도움을 준다. (100%는 아님) 실제 동작하는 방법은 아래와 같다.

```go
go test -cover
```

### 새로운 함수 추가
여러 `슬라이스`를 인자로 받아 각 `슬라이스`의 총합을 슬라이스로 반환하는 `SumAll`함수를 만들어보자. 예를들어 `SumAll([]int{1, 1, 1}, []int{1, 9})`는 `[]int{3, 10}`을 반환한다.

이제 `sum_test.go` 파일에 새로운 함수인 `func TestSumAll`를 작성해보자.

:heavy_check_mark: sum_test.go 수정
```go
package main

import "testing"

func TestSum(t *testing.T) {
    // Something..
}

func TestSumAll(t *testing.T) {
    got := SumAll([]int{1, 2}, []int{0, 9})
    want := []int{3, 9}

    if got != want {
        t.Errorf("got %v want %v", got, want)
    }
}
```
위 `TestSumAll` 테스트가 통과하기 위한 최소한의 코드를 작성한다.

:heavy_check_mark: sum.go 수정
```go
package main

func Sum(numbers []int) int {
    // Something...
}

func SumAll(numbersToSum... []int) (sums []int) {
    return
}
```
위와 같이 테스트 코드 및 베이스 코드를 작성한 뒤 테스트를 돌리려고 하면 `Error`가 발생한다. 이유는 `비교 연산자`가 잘못되었기 때문이다.

`slice`는 `nil`과 비교해야 한다. 즉 `!=` 연산자는 `slice`가 `nil`인지 확인하는 용도로 사용된다. **만약 슬라이스의 원소들의 비교를** 하고 싶다면 `reflect.DeepEqual`을 사용해야 한다. 따라서 `TestSumAll` 함수를 아래와 같이 수정한다.

:heavy_check_mark: func TestSumAll 수정
```go
func TestSumAll(t *testing.T) {
    got := SumAll([]int{1, 2}, []int{0, 0})
    want := []int{3, 9}

    if !reflect.DeepEqual(got, want) {
        t.Errorf("got %v want %v", got, want)
    }
}
```

여기서 주의해야 할 점은 **`reflect.DeepEqual()`함수가 `type safe`하지 않다는 점이다.** 위의 예제에서 `want = "Bob"` 으로 변경(string 과 비교)한 뒤 실행해도 compile이 된다. 따라서 `reflect.DeepEqual`을 사용시에 `type`에 대해 주의해야 한다.

> **:bulb: reflect.DeepEqual() 은 Type Safe 하지 않다!**

이제 `TestSumAll` 테스트 함수가 통과할 수 있도록 `SumAll` 함수를 변경한다.

:heavy_check_mark: sum.go 수정
```go
package main

func Sum(numbers []int) int {
    // Something...
}

func SumAll(numbersToSum... []int) []int {
    lengthOfNumbers := len(numbersToSum)
    sums := make([]int, lengthOfNumbers)

    for i, numbers := range numbersToSum {
        sums[i] = Sum(numbers)
    }

    return sums
}
```
`numbersToSum`의 크기를 가져와 `make`를 이용해 `sums` 슬라이스의 크기를 미리 할당하였다. 이후 각 `slice`의 sum을 구해 `sums`에 할당해주면 된다.

이제 `SumAll` 함수를 `REFACOR` 해보자.


:heavy_check_mark: func SumAll() 수정
```go
package main

func Sum(numbers []int) int {
    // Something...
}

func SumAll(numbersToSum... []int) []int {
    var sums []int
    for _, numbers := range numbersToSum {
        sums := append(sums, Sum(numbers))
    }
    return sums
}
```
위와 같은 코드로 변경시 `capacity`는 고려하지 않아도 된다.

### 요구사항 추가
위에서 구현한 `SumAll()` 함수를 `SumAllTails()` 함수로 변경해보자.  
`SumAllTails()` 함수는 첫 번째 원소를 제외한 나머지 원소들의 합을 구하는 함수이다.

:heavy_check_mark: sum_test.go 수정
```go
package main

import "testing"

func TestSumAllTails(t *testing.T) {
    got := SumAllTails([]int{1, 2}, []int{0, 9})
    want := []int{2, 9}

    if got != want {
        t.Errorf("got %v want %v", got, want)
    }
}
```
이제 SumAllTails 테스트가 통과할 수 있도록 `SumAll()` 함수를 `SumAllTails()`로 번경해보자.

:heavy_check_mark: sum.go 수정
```go
package main

func SumAllTails(numbersToSum... []int) []int {
    var sums []int
    for _, numbers := range numbersToSum {
        tail := numbers[1:]
        sums = append(sums, Sum(tail))
    }
    return sums
}
```
입력받은 `Slice`에서 `Slicing`을 이용해 첫 번째 원소를 제외한 `Slice`를 만들고 `Sum` 함수를 이용해 전체 합을 구하도록 변경하였다.

**하지만,** 과연 이 코드에 문제가 없을지 잘 생각해보자.

### Refactoring
위의 코드는 전혀 문제 없는 것 처럼 보이지만 **문제점이 있다.** 만약 `빈 슬라이스`를 `SumAllTails()` 함수에 넣는다면 어떻게 되는지 확인해보자.

오류가 발생하는 것을 볼 수 있다. 여기서 주의해야 할 점은 `컴파일 오류`가 아닌 `런타임 오류`라는 것이다. `런타임 오류`는 사용자에게 직접적인 영향을 줄 수 있으므로 주의해야 한다. 이제 이 문제를 해결하기 위해 코드를 `REFACTOR` 해보자.

:heavy_check_mark: sum.go 수정
```go
package main

func SumAllTails(numbersToSum... []int) []int {
    var sums []int
    for _, numbers := range numbersToSum {
        if len(numbers) == 0 {
            sums = append(sums, 0)
        } else {
            tail := numbers[1:]
            sums = append(sums, Sum(tail))
        }
    }
    return sums
}
```
추가적으로 여러 테스트를 작성할 때 반복되는 `assertion code`를 제거하기 위해 `REFACTOR`를 진행해보자.

:heavy_check_mark: func TestSumAllTails() 수정
```go
func TestSumAllTails(t *testing.T) {
	checkSum := func(t *testing.T, got, want []int) {
		t.Helper()
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	}
	t.Run("make the sums of tails of", func(t *testing.T) {
		got := SumAllTails([]int{1, 2}, []int{0, 9})
		want := []int{2, 9}
		checkSum(t, got, want)
	})

	t.Run("safely sum empty slices", func(t *testing.T) {
		got := SumAllTails([]int{}, []int{0, 9})
		want := []int{0, 9}
		checkSum(t, got, want)
	})
}
```
위와 같이 `checkSum(t, got, want)`로 `assertion code`를 따로 분리하였다. 이렇게 분리함으로써 `코드의 중복`을 줄임과 동시에 `type safe`한 코드를 작성할 수 있다. `want := "dave"`로 수정한 뒤 테스트를 진행해보자.
