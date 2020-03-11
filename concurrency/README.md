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
