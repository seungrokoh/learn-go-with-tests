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
