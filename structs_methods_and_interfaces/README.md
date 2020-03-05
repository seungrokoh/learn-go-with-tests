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

## **Add Circle Area**
새로운 요구사항을 추가해보자. 기존에 있던 `Area()` 함수에 `Circle`의 넓이를 구하기를 추가하는 것이다. `shapes_test.go` 파일에 `Circle`의 넓이를 구하는 테스트를 추가해보자.

:heavy_check_mark: shapes_test.go 수정
```go
package structs_methods_and_interfaces

import "testing"

func TestArea(t *testing.T) {
	t.Run("rectangles", func(t *testing.T) {
		rectangle := Rectangle{12.0, 6.0}
		got := Area(rectangle)
		want := 72.0

		if got != want {
			t.Errorf("got %.2f want %.2f", got, want)
		}
	})

	t.Run("circles", func(t *testing.T) {
		circle := Circle{10}
		got := Area(circle)
		want := 314.1592653589793

		if got != want {
			t.Errorf("got %.2f want %.2f", got, want)
		}
	})
}
```

:heavy_check_mark: shape.go 수정
```go
type Circle struct {
    radius float64
}
```
이제 `Circle` struct에 대해서 `Area()` 함수를 생성하기 위해 `func Area(circle Circle) float64`를 선언해보자. 에러가 발생 할 것이다. **Go 언어에서는 다른 언어에서 제공하는 오버로딩 (Overloading)을 제공하지 않는다.** 이 문제를 해결하기 위해 2가지 방법이 존재한다.

1. 새로운 패키지를 만들고 그 패키지 안에 `Area(Circle)` 함수를 생성한다.
2. `메서드(methods)`를 정의한다.

`메서드(methods)`는 `함수(Functions)`와 매우 비슷하지만 `메서드`는 특정 유형의 인스턴스에서 호출됩니다.

각 도형 `Rectangle`, `Circle`에 대한 `Area()` 메서드를 정의하고 테스트 코드를 수정해보자.

:heavy_check_mark: 메서드 정의
```go
type Rectangle struct {
    Width float64
    Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}
```
`메서드`를 작성하는 방법은 함수를 작성하는 것과 비슷하지만 **func 키워드와 함수명 사이에 Receiver가 들어가는 것이 다르다.** 작성하는 형식은 아래와 같다.

```go
func (receiverName receiverType) MethodName(args)
```
다른 프로그래밍 언어에서는 리시버에 접근할 때 `this` 키워드를 사용한다. **Go 에서 ReceiverName을 정하는 `Convention`은 Type의 첫 번째 letter로 되도록 하는 규칙이다.**

이제 테스트 코드를 수정해보자.

:heavy_check_mark: shapes_test.go 수정
```go
func TestArea(t *testing.T) {
	t.Run("rectangles", func(t *testing.T) {
		rectangle := Rectangle{12.0, 6.0}
		got := rectangle.Area()
		want := 72.0

		if got != want {
			t.Errorf("got %.2f want %.2f", got, want)
		}
	})

	t.Run("circles", func(t *testing.T) {
		circle := Circle{10}
		got := circle.Area()
		want := 314.1592653589793

		if got != want {
			t.Errorf("got %.2f want %.2f", got, want)
		}
	})
}
```

## Refactor
위에서 작성한 함수에서 중복되는 부분을 `Refactor` 해보자.

도형을 생성한 다음 `Area()`를 호출하고 `Error`를 확인하는 부분의 `코드 중복`을 제거해보자. `checkArea()` 함수를 만들고 해당 함수에 도형을 넘겨주면 `checkArea()` 함수 안에서 각 도형에 대한 `Area()`를 호출하면 된다. **하지만 `Rectangle`과 `Circle`은 서로 다른 타입이기 때문에 `checkArea()` 함수에 같이 넘길 수 없다.** 이러한 문제점을 해결할 수 있는 방법이 `interfaces`를 사용하는 것이다.

:bulb: Interfaces  

`Interfaces` are a very powerful concept in statically typed languages like Go because they allow you to make functions that can be used with different types and create highly-decoupled code whilst still maintaining type-safety.

이제 `shapes_test.go` 파일을 `Refactor` 해보자.

:heavy_check_mark: refactoring shapes_test.go
```go
func TestArea(t *testing.T) {
	checkArea := func(t *testing.T, shape Shape, want float64) {
		t.Helper()
		got := shape.Area()
		if got != want {
			t.Errorf("got %.2f want %.2f", got, want)
		}
	}
	t.Run("rectangles", func(t *testing.T) {
		rectangle := Rectangle{12.0, 6.0}
		checkArea(t, rectangle, 72.0)
	})

	t.Run("circles", func(t *testing.T) {
		circle := Circle{10}
		checkArea(t, circle, 314.1592653589793)
	})
}
```
이제 도형의 `Area()` 메서드를 가진 `interface`를 선언한다.

:heavy_check_mark: interface 선언 in shape.go
```go
// Something....

type Shape interface {
    Area() float64
}
```
위와 같이 코드를 `Refactoring` 하면 테스트 코드는 `PASS` 할 것이다. 여기서 궁금한 점은 **어떻게 Shape Interface로 정의가 되는가?** 인데. 다른 언어에서 제공하는 `Interface`와는 조금 다르다.

다른 언어에서 제공하는 `Interface`는 `명시적`으로 구현한다. 예를 들어 아래와 같다.


- My type Foo implement interface Bar


하지만 **Go 언어에서 `Interface`는 `암시적`으로 구현한다.**

- `Rectangle`은 `float64`를 반환하는 `Area()` 메서드를 가지고 있어 `Shape` Interface를 만족한다.
- `Circle`은 `float64`를 반환하는 `Area()` 메서드를 가지고 있어 `Shape` Interface를 만족한다.
- `string`은 `float64`를 반환하는 `Area()` 메서드를 가지고 있지 않아 `Shape` Interface를 만족하지 않는다.

:bulb: **Decoupling**  
인터페이스를 선언함으로써 헬퍼는 구체적인 유형에서 분리(사각형, 삼각형, 원형인지 알 필요가 없음)되고 작업을 수행하는 데 필요한 방법만 알면 된다.

### Table Driven Test

[Table Driven Test](https://github.com/golang/go/wiki/TableDrivenTests)를 진행해보자. `Table driven test`는 동일한 방식으로 테스트할 수 있는 테스트 목록을 테스트할 때 유용하다.

`Table driven test`를 진행하기 위해 `익명 구조체 슬라이스`를 생성해보자. 해당 구조체는 도형을 나타내는 `Shape`와 테스트를 통과하는 기준인 `want` Field를 가지고 있다.

:heavy_check_mark: func TestArea() 수정
```go
func TestArea(t *testing.T) {

    areaTests := []struct {
        shape Shape
        want float64
    } {
        {Rectangle{12, 6}, 72.0},
        {Circle{10}, 314.1592653589793},
    }

    for _, tt := range areaTests {
        got := tt.shape.Area()
        if got != want {
            t.Errorf("got %.2f want %.2f", got, tt.want)
        }
    }
}
```
이제 새로운 도형인 `Triangle`을 작성하고 테스트 케이스에 추가해보자.

:heavy_check_mark: shape_test.go 수정
```go
func TestArea(t *testing.T) {

    areaTests := []struct {
        shape Shape
        want float64
    } {
        {Rectangle{12, 6}, 72.0},
        {Circle{10}, 314.1592653589793},
        {Triangle{12, 6}, 36.0}
    }

    for _, tt := range areaTests {
        got := tt.shape.Area()
        if got != want {
            t.Errorf("got %.2f want %.2f", got, tt.want)
        }
    }
}
```
새로운 테스트 케이스를 추가하는게 엄청 쉽다. 이제 테스트 코드가 동작하 수 있도록 `Triangle` 구조체를 선언하자.

:heavy_check_mark: Triangle 구조체 선언
```go
type Triangle struct {
    Base float64
    Height float64
}
```
하지만 `Triangle` 구조체를 선언하는 것으로 끝나는 것이 아니다. 테스트 코드를 통과하기 위해선 `Triangle`이 `Shape`를 만족해야 하기 때문이다. 따라서 `Triangle`에 `Area()` 함수를 추가해준다.

:heavy_check_mark: Area() 추가
```go
func (t Triangle) Area() float64 {
    return (t.Base * t.Height) * 0.5
}
```

## Refactor
위 코드는 테스트를 통과하고 `Table driven test`를 진행하는데 전혀 무리가 없다. 하지만 가독성을 위해서 아래와 같은 코드를 살펴보자.

`areaTests []struct`를 선언하는 곳의 일부분이다.
```go
{Rectangle{12, 6}, 72.0},
{Circle{10}, 314.1592653589793},
{Triangle{12, 6}, 36.0}
```
구조체를 선언할 때 각 Field가 무엇을 나타내는지 선언했지만 위와 같이 테스트 케이스를 추가할 때 어떤 역할을 하는지 구분하기가 힘들다. 여기서 가독성을 더 좋게 하기 위해 field name 을 앞에 붙여줄 수 있다.

```go
{shape: Rectangle{12, 6}, want: 72.0},
{shape: Circle{10}, want: 314.1592653589793},
{shape: Triangle{12, 6}, want: 36.0},
```

또 다른 가독성을 높이는 방법이 있다. `Triangle` 테스트를 처음 추가하고 test를 실행했을 때 출력 된 `error message`를 보자.
```go
shapes_test.go:31: got 0.00 want 36.00
```
문제될 건 없지만 만약 테스트가 3개가 아니고 100만개가 있다고 가정 ~~(오버해서)~~ 해보자. 어떤 테스트 케이스가 조건을 만족하지 않는지 알기가 힘들다. 물론 `line number`를 알려주고 있긴 하지만 그것만 보기에는 힘들다. 개발자가 이해하기 더 쉽게 만들기 위해 test case 에 이름을 붙여보고 이를 `error message`에 출력해보자.

:heavy_check_mark: func TestArea() 수정
```go
func TestArea(t *testing.T) {
	areaTests := []struct {
		name string
		shape Shape
		hasArea float64
	} {
		{name: "Rectangle", shape:Rectangle{12, 6}, hasArea: 72.0},
		{name: "Circle", shape: Circle{10}, hasArea: 314.1592653589793},
		{name: "Triangle", shape: Triangle{12, 6}, hasArea: 36.0},
	}

	for _, tt := range areaTests {
		// Using tt.name from the case to use it as the `t.Run` test name
		t.Run(tt.name, func(t *testing.T) {
			got := tt.shape.Area()
			if got != tt.hasArea {
				t.Errorf("%#v got %.2f want %.2f", tt.shape, got, tt.hasArea)
			}
		})
	}
}
```
`Table driven test`를 사용하면 `t.Run`을 사용하여 테스트 케이스를 확인해볼 수 있다.
