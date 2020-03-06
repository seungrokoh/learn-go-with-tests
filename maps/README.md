# Maps
`arrays & slices`를 이용해 데이터를 순차적으로 저장하는 방법에 대해서 살펴봤다면 `<key, value>` 형태로 데이터를 저장하는 방법에 대해서 살펴보자.

`map`은 데이터를 `<key, value>` 형태로 저장한다. 이는 데이터를 빠르게 저장하고, 검색할 수 있으며 많은 곳에서 활용된다. `map`을 이용해 `단어`를 검색하면 `뜻`을 알려주는 `Dictionary`를 만들어보자. 먼저 Test Code를 작성해보자.

:heavy_check_mark: dictionary_test.go
```go
package maps

import "testing"

func TestSearch(t *testing.T) {
	dictionary := map[string]string {"test":"this is just a test"}

	got := Search(dictionary, "test")
	want := "this is just a test"

	if got != want {
		t.Errorf("got %q want %q given %q", got, want, "test")
	}
}
```
간단하게 테스트용으로 `key`값은 `test`, `value`값은 `this is just a test`를 map에 저장해놨다. 이제 `Search()`함수를 작성해보자.

:heavy_check_mark: dictionary.go
```go
package maps

func Search(dictionary map[string]string, word string) string {
    return dictionary[word]
}
```
`map`은 map[key]value로 정의 하고 `map[key]`로 `value` 값을 가져올 수 있다.

## Refactor

이제 테스트 코드를 깔끔하게 `refactoring` 해보자.  

- 첫 번재로 `Custom type`으로 `map[string]string`을 `Dictionary`라고 정의한다.  
- 두 번째로 `assertStrings` Helper를 정의한다.

:heavy_check_mark: dictionary.go
```go
package maps

type Dictionary map[string]string

func (d Dictionary) Search(word string) string{
	return d[word]
}
```

:heavy_check_mark: dictionary_test.go
```go
package maps

import "testing"

func TestSearch(t *testing.T) {
	dictionary := Dictionary {"test":"this is just a test"}

	got := dictionary.Search("test")
	want := "this is just a test"

	assertStrings(t, got, want)
}

func assertStrings(t *testing.T, got, want string) {
	t.Helper()

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```
