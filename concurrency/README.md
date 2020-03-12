# Concurrency

url 목록들을 받아 url의 상태들을 반환하는 `CheckWebsites` 함수를 작성해보자. 이 함수는 url 들의 상태를 저장한 `map`을 반환하며 reponse의 상태를 `true`, `false`로 저장한다.

:heavy_check_mark: CheckWebsites.go
```go
package concurrency

type WebsiteChecker func(string) bool

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)

	for _, url := range urls {
		results[url] = wc(url)
	}
	return results
}
```
함수에서 **하나의 url을 검사하고 그 상태값을 반환하는 `WebsiteChecker` 함수를 인자로 받는다.**

`Dependency Injection`을 사용하여 테스트에서 실제 HTTP Call을 하지 않고 빠르게 테스트를 진행할 수 있게 한다.

이제 테스트 코드를 작성해보자.

:heavy_check_mark: CheckWebsites_test.go
```go
package concurrency

import (
	"reflect"
	"testing"
)

func mockWebsiteChecker(url string) bool {
	if url == "waat://furhurterwe.geds" {
		return false
	}
	return true
}

func TestCheckWebsites(t *testing.T) {
	websites := []string{
		"http://google.com",
		"http://blog.gypsydave5.com",
		"waat://furhurterwe.geds",
	}

	want := map[string]bool{
		"http://google.com":          true,
		"http://blog.gypsydave5.com": true,
		"waat://furhurterwe.geds":    false,
	}

	got := CheckWebsites(mockWebsiteChecker, websites)

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

만약 이 기능이 **프로덕션 코드라고 생각해보자.** 동료가 속도가 너무 느리다고 개선해달라고 했을 때를 가정했을 때 코드를 변경해보자.

먼저 `Benchmark`를 이용하여 `CheckWebsites` 함수 변경이 속도에 영향을 끼치는 것을 확인할 수 있게 작성해보자.

:heavy_check_mark: CheckWebsites_benchmark_test.go
```go
package concurrency

import (
	"testing"
	"time"
)

func slowStubWebsiteChecker(_ string) bool {
	time.Sleep(20 * time.Millisecond)
	return true
}

func BenchmarkCheckWebsites(b *testing.B) {
	urls := make([]string, 100)

	for i := 0; i < len(urls); i++ {
		urls[i] = "a url"
	}

	for i := 0; i < b.N; i++ {
		CheckWebsites(slowStubWebsiteChecker, urls)
	}
}
```
위 벤치마크는 100개의 가짜 url을 만들고 일부러 속도를 느리게 만드는 `slowStubWebsiteChecker` 함수를 `CheckWebsites`에 넘깁니다.  

이를 벤치마크 하기 위해 `go test -bench=.`를 입력해보자. 예제에서는 약 4분의 1초가 걸린다.

이제 빠르게 만들어보자.

# 빠르게 만들기
동작을 빠르게 만들기 위해서 `goroutine`을 사용해보자.

`goroutine`을 이해하기 전 먼저 Go 프로그램이 동작할 때를 간략하게 이해 해야한다. 프로그램이 실행이 되면 순차적으로 명령어들이 실행되고 body의 끝부분에 가면 함수가 종료된다.

예를들어 `got := doSomething()`을 실행하면 `doSomething` 함수가 종료되고 값을 반환 할 때까지 메인 프로그램은 멈춰 있다. 즉, `Blocking`된다. 만약에 메인에서 실행되는 명령어들과 `doSomething`에서 실행되는 명령어들이 `동시에 진행` 된다면? 순차적으로 진행하는 것 보다 더 빠르게 명령어들을 수행할 수 있을 것이다.

`goroutine`은 **어떤 행동을 별도의 프로세스에서 동작하게 만든다.** 예를들어, `doSomething`이라는 for문으로 100번을 돌린다고 했을 때 그냥 돌리면 100번 모두 끝날 때까지 기다리지만, `goroutine`을 사용하면 100개의 `doSomething` 함수는 **각자의 별도 프로세스에서 동작하게 된다.**

`goroutine`을 실행하는 키워드는 앞에 `go`를 붙이면 된다. 즉, `go doSomething()`이다. 이제 `CheckWebsites`를 `goroutine`을 사용하여 빠르게 변경해보자.

:heavy_check_mark: CheckWebsites.go 수정
```go
package concurrency

type WebsiteChecker func(string) bool

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)

	for _, url := range urls {
		go func() {
			results[url] = wc(url)
		}()
	}
	return results
}
```
기존 함수와는 다를게 없다. 단지 `WebsiteCheckr` 함수를 호출하고 그 반환값을 `results` map에 저장하는 걸 `goroutine`으로 변경했을 뿐이다.

**하지만 테스트를 진행해보면 `FAIL`이 발생한다.** 이유는 결과값 (got) map이 비어(또는 1개 밖에 없는)있다는 에러가 나타날 뿐이다.

달라진게 없는데 왜 이런 현상이 발생하는 걸까? 그 이유는 `goroutine`이 결과를 map에 추가하기 전에 `CheckWebsites`가 종료되기 때문이다. for loop를 돌면서 각 `goroutine`의 결과를 받기 전 `CheckWebsites` 함수가 종료되어 `results` map 에 결과가 저장되지 않은 채 반환한다.

코드를 수정해보자.

:heavy_check_mark: CheckWebsites.go
```go
package concurrency

type WebsiteChecker func(string) bool

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)

	for _, url := range urls {
		go func(u string) {
			results[u] = wc(u)
		}(url)
	}
	return results
}
```
익명함수에 각 url을 넘겨주도록 수정하고 테스트를 돌려보자. 테스트는 성공 할 것이다. 하지만 benchmark를 돌려보면 끔찍한 결과를 얻게된다. 이런 오류 메세지가 나오는 이유는 `race condition` 때문이다.

# Channels
`Channel`을 이용하여 `race condition`을 해결할 수 있다.

`Channel`은 값을 수신 및 전송할 수 있는 `Go 데이터 구조`이다. 이러한 구조는 세부정보와 함께 **다른 프로세스 간 통신할 수 있게 만들 수 있다.**

위 예제에서는 `부모 프로세스`와 각각의 `goroutine`과의 통신을 예로들 수 있다. 코드를 수정해보자.

:heavy_check_mark: CheckWebsites.go
```go
package concurrency

type WebsiteChecker func(string) bool
type result struct {
	string
	bool
}

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)
	resultChannel := make(chan result)

	for _, url := range urls {
		go func(u string) {
			resultChannel <- result {u, wc(u)}
		}(url)
	}

	for i := 0; i < len(urls); i++ {
		result := <- resultChannel
		results[result.string] = result.bool
	}
	return results
}
```
결과를 `Channel`로 보내 `results` map에 write 타이밍을 제어하여 한 번에 하나씩 쓸 수 하게 하였다.

위 코드를 수정함으로써 **속도를 빠르게 하기 위해 병렬화 하고 싶은 부분을 `goroutine`을 이용해 병렬화 하고** 병렬화 할 수 없는 부분은 **선형으로 진행되도록 하였다.**
