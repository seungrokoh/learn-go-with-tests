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

## Unknown Case
데이터를 찾는데 `Search` 함수는 문제가 없어보인다. **하지만,** 만약 없는 데이터를 뽑아내려고 하면 어떻게 될까? 존재하지 않는 데이터를 뽑아내려 할 때와 존재하는 데이터를 뽑아내려 할 때를 구분짓고 사용자에게 `Error`를 알릴 수 있도록 수정해보자.

:heavy_check_mark: dictionary_test.go 수정
```go
package maps

import "testing"

func TestSearch(t *testing.T) {
	dictionary := Dictionary{"test": "this is just a test"}

	t.Run("known word", func(t *testing.T) {
		got, _ := dictionary.Search("test")
		want := "this is just a test"

		assertStrings(t, got, want)
	})

	t.Run("unknown word", func(t *testing.T) {
		_, err := dictionary.Search("unknown")
		want := "could not find the word you were looking for"

		if err == nil {
			t.Fatal("expected to get an error.")
		}
		assertStrings(t, err.Error(), want)
	})
}

func assertStrings(t *testing.T, got, want string) {
	t.Helper()

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

:heavy_check_mark: dictionary.go 수정
```go
package maps

import "errors"

type Dictionary map[string]string

func (d Dictionary) Search(word string) (string, error) {
	definition, ok := d[word]
	if !ok {
		return "", errors.New("could not find the word you were looking for")
	}
	return definition, nil
}
```
변경 사항중 **참고해야 할 부분이 몇 가지 존재한다.**

* Go 에서 함수는 `리턴값이 2개 이상일 수 있다`
    * 선택적으로 리턴값을 받을 수 있다.
    * err는 2번째 리턴값으로 받는게 일반적이다.


* Go 에서 map의 return 값을 2개 받을 수 있다.
    * 첫 번째 리턴값은 value 이다.
    * 두 번째 리턴값은 value의 존재 여부(bool) 이다.


* error 는 .Error()를 통해 string 값을 반환할 수 있다.
    * builtin package 에 존재한다.
    * error interface를 참고해보자.

## Refactor
Err를 출력하는데 중복되는 코드를 제거해보자. 발생시키는 error를 변수화 시킨다.

:heavy_check_mark: dictionary.go 수정
```go
// in dictionary.go

var ErrNotFound = errors.New("could not find the word you were looking for")

type Dictionary map[string]string

func (d Dictionary) Search(word string) (string, error) {
	definition, ok := d[word]
	if !ok {
		return "", ErrNotFound
	}
	return definition, nil
}
```

:heavy_check_mark: dictionary_test.go
```go
// in dictionary_test.go

t.Run("unknown word", func(t *testing.T) {
    _, err := dictionary.Search("unknown")

    if err == nil {
        t.Fatal("expected to get an error.")
    }
    assertError(t, err, ErrNotFound)
})

// create new helper
func assertError(t *testing.T, got, want error) {
	t.Helper()

	if got != want {
		t.Errorf("got error %q want %q", got, want)
	}
}
```

## Add new words
Dictionary에 새로운 Word를 추가하는 메서드를 추가해보자. 먼저 새로운 Word를 추가하는 테스트 코드를 만들고, Add 메서드를 구현해보자.

:heavy_check_mark: dictionary_test.go
```go
// in dictionary_test.go

func TestAdd(t *testing.T) {
	dictionary := Dictionary{}
	dictionary.Add("test", "this is just a test")

	want := "this is just a test"
	got, err := dictionary.Search("test")

	if err != nil {
		t.Fatal("should find added word:", err)
	}
	if want != got {
		t.Errorf("got %q want %q", got, want)
	}
}
```

:heavy_check_mark: dictionary.go
```go
func (d Dictionary) Add(word, definition string) {
	d[word] = definition
}
```

## Refactor
테스트를 단순화할 수 있도록 약간의 `refactoring`을 해보자.

:heavy_check_mark: dictionary_test.go
```go
// in dictionary_test.go

// Something ...

// modify TestAdd
func TestAdd(t *testing.T) {
    dictionary := Dictionary{}
    word := "test"
    definition := "this is jus a test"

    dictionary.Add(word, definition)
    assertDefinition(t, dictionary, word, definition)
}

// add assertDefinition helper
func assertDefinition(t *testing.T, dictionary Dictionary, word, definition string) {
    t.Helper()

    got, err := dictionary.Search(word)
    if err != nil {
        t.Fatal("should find added word:", err)
    }
    if definition != got {
        t.Errorf("got %q want %q", got, definition)
    }
}
```

## 테스트 케이스 추가
위에서 만든 `Add`함수는 잘 동작하는 것처럼 보인다. **하지만,** 고려하지 않은 부분이 있다. 만약 **이미 존재 하는 값을 또 추가한다고 한다면?** 어떤 일이 발생할까

`map`은 이밎 존재하는 값에 대해 `error`를 던져주지 않는다. 이미 존재하는 값에 대해 다시 add하려 한다면 **새로운 값을 기존값에 덮어 씌운다.** 새로운 값을 덮어 씌우는건 문제가 되지 않지만 여기서 작성하는 `Add` 함수는 값을 업데이트 하는게 아닌 **새로운 값을 넣는 역할을 한다.** 따라서 `Add` 함수의 역할을 확실히 하기 위해 새로운 테스트 케이스를 추가해보자.

`Add`는 **오직 새로운 함수를 추가하는 역할만 하도록** 수정해보자.

:heavy_check_mark: dictionary_test.go 수정
```go
// in dictionary_test.go

// Something ...

// modify TestAdd
func TestAdd(t *testing.T) {
	t.Run("new word", func(t *testing.T) {
		dictionary := Dictionary{}
		word := "test"
		definition := "this is just a test"

		err := dictionary.Add(word, definition)

		assertError(t, err, nil)
		assertDefinition(t, dictionary, word, definition)
	})

	t.Run("existing word", func(t *testing.T) {
		word := "test"
		definition := "this is jus a test"
		dictionary := Dictionary{word: definition}

		err := dictionary.Add(word, "new test")

		assertError(t, err, ErrWordExists)
		assertDefinition(t, dictionary, word, definition)
	})
}
```

:heavy_check_mark: dictionary.go
```go
// in dictionary.go

// add error variables
var (
	ErrNotFound = errors.New("could not find the word you were looking for")
	ErrWordExists = errors.New("cannot add word because it already exists")
)

// Something ..

//modify Add
func (d Dictionary) Add(word, definition string) error {
	_, err := d.Search(word)

	switch err {
	case ErrNotFound:
		d[word] = definition
	case nil:
		return ErrWordExists
	default:
		return err

	}
	return nil
}
```

## Refactor
Error를 사용하는 곳이 많아짐에 따라 Error를 재사용 가능하게 `Refactoring` 해보자. 먼저 `Custom type`으로 `DictionaryErr`를 만들고 `error interface`를 구현하게 만든다.

그 다음 단어의 뜻을 수정할 수 있는 `Update` 메서드를 만들어보자.

* Dictionary에 이미 `word`가 존재한다면 새로운 `Definition`으로 변경한다.
* Dictionary에 `word`가 존재하지 않는다면 존재하지 않는다는 에러를 발생시킨다.

:heavy_check_mark: dictionary_test.go
```go
func TestUpdate(t *testing.T) {
	t.Run("existing word", func(t *testing.T) {
		word := "test"
		definition := "this is just a test"
		newDefinition := "new definition"
		dictionary := Dictionary{word: definition}

		err := dictionary.Update(word, newDefinition)

		assertError(t, err, nil)
		assertDefinition(t, dictionary, word, newDefinition)
	})

	t.Run("new word", func(t *testing.T) {
		word := "test"
		definition := "this is just a test"
		dictionary := Dictionary{}

		err := dictionary.Update(word, definition)
		assertError(t, err, ErrWordDoesNotExist)
	})
}
```

:heavy_check_mark: dictionary.go
```go
func (d Dictionary) Update(word, definition string) error {
	_, err := d.Search(word)

	switch err {
	case ErrNotFound:
		return ErrWordDoesNotExist
	case nil:
		d[word] = definition
	default:
		return err
	}
	return nil
}
```

## Add Delete Function
마지막으로 등록된 단어를 삭제하는 기능을 만들어보자.

:heavy_check_mark: dictionary_test.go 수정
```go
// in dictionary_test.go

// Something ...

// Add TestCode
func TestDelete(t *testing.T) {
	t.Run("existing word", func(t *testing.T) {
		word := "test"
		dictionary := Dictionary{word: "test definition"}

		dictionary.Delete(word)
		_, err := dictionary.Search(word)
		if err != ErrNotFound {
			t.Errorf("Expected %q to bi deleted", word)
		}
	})
}
```

:heavy_check_mark: dictionary.go
```go
func (d Dictionary) Delete(word string) {
	delete(d, word)
}
```

Delete 메서드는 다른 메서드들과 다르게 아무런 반환값이 없다. 굳이 `Error`를 만들고 검사한 뒤 Delete 할 필요가 없다. 왜냐하면 다른 함수들과는 다르게 삭제 메서드는 `side effect`가 없기 때문이다.
