# Structs, Methods and Interfaces

사각형의 `width`및 `height`가 주어졌을 때 둘레의 길이를 구하는 테스트 코드를 작성하고 함수를 만들어보자.

Perimeter 함수를 검사하는 `테스트 코드`를 작성하고 테스트를 통과할 수 있는 최소한의 코드를 작성해보자.

:heavy_check_mark: shapes_test.go
```go
package structs_methods_and_interfaces

import "testing"

func TestPerimeter(t *testing.T) {
    got := Perimeter(10.0, 10.0)
    want := 40.0

    if got != want {
        t.Errorf("got %.2f want %.2f", got, want)
    }
}
```

:heavy_check_mark: shapes.go
```go
package structs_methods_and_interfaces

func Perimeter(width, height float64) float64 {
    return 2 * (width + height)
}
```
이제 TDD에 조금 익숙해졌기 때문에 여기까지는 좀 쉽게 작성할 수 있다. 이제 사각형의 넓이를 구하는 `Area(width, height float64)` 함수를 작성해보자. `shapes_test.go` 파일에 `Area`의 테스트 코드를 작성하고 `shapes.go` 파일에 `Area` 함수를 작성하면 된다.

:heavy_check_mark: shapes_test.go
```go
package structs_methods_and_interfaces

// ...

func TestArea(t *testing.T) {
    got := Area(12.0, 6.0)
    want := 72.0

    if got != want {
        t.Errorf("got %.2f want %.2f", got, want)
    }
}
```

:heavy_check_mark: shapes.go
```go
package structs_methods_and_interfaces

// ...

func Area(width, height float64) float64 {
    return width * height
}
```

## **Refactor**
위의 예제에서 `perimeter`나 `area`는 특정 도형을 지칭하고 있지 않는다. 만약 `삼각형`의 둘레 및 넓이를 구하는데 해당 함수를 사용한다면 잘못된 결과값을 반환 할 것이다.  

이를 해결하기 위해 `RectangleArea` 같이 함수 이름을 특정 이름으로 나타내는 방법이 있지만 **더 좋은 방법은 이 개념을 캡슐화 하는 `Rectangle` 이라는 유형을 정의하는 것이다.**

`Rectangle` struct를 정의하고 테스트 코드를 수정해보자.

:heavy_check_mark: Rectangle struct
```go
// in shapes.go

type Rectangle struct {
    Width float
    Height float
}
```

:heavy_check_mark: perimeter_test.go 수정
```go
package structs_methods_and_interfaces

import "testing"

func TestPerimeter(t *testing.T) {
	rectangle := Rectangle{10.0, 10.0}
	got := Perimeter(rectangle)
	want := 40.0

	if got != want {
		t.Errorf("got %.2f want %.2f", got, want)
	}
}

func TestArea(t *testing.T) {
	rectangle := Rectangle{12.0, 6.0}
	got := Area(rectangle)
	want := 72.0

	if got != want {
		t.Errorf("got %.2f want %.2f", got, want)
	}
}
```
struct를 이용해 `테스트 코드`를 수정한 뒤 `shapes.go` 파일의 `Perimeter`, `Area` 함수를 수정한다.

:heavy_check_mark: shapes.go 수정
```go
func Perimeter(rectangle Rectangle) float64 {
    return 2 * (rectangle.Width + rectangle.Height)
}

func Area(rectangle Rectangle) float64 {
    return rectangle.Width * rectangle.Height
}
```
