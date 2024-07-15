## I/O 프로젝트 개선

이제 이터레이터에 대한 새로운 지식을 갖고 있으므로, 12장에서 소개된 I/O 프로젝트를 이터레이터를 사용하여 코드를 더 명확하고 간결하게 만들어 개선할 수 있습니다. `Config::build` 함수와 `search` 함수의 구현을 이터레이터가 개선하는 방식을 살펴보겠습니다.

### 이터레이터를 사용하여 `clone` 제거

12-6번 목록에서, `String` 값의 슬라이스를 가져와 `Config` 구조체의 인스턴스를 생성하는 코드를 추가했습니다. 이 코드는 슬라이스를 인덱싱하고 값을 복사하여 `Config` 구조체가 이 값들을 소유하도록 했습니다. 13-17번 목록에서는 12-23번 목록에서 `Config::build` 함수의 구현을 다시 작성했습니다.

<Listing number=\"13-17\" file-name=\"src/lib.rs\" caption=\"12-23번 목록에서 재현된 `Config::build` 함수\">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/lib.rs:ch13}}
```

</Listing>

당시에는 `clone` 호출이 비효율적이지만, 나중에 제거할 것이라고 말했습니다. 이제 그 시간이 왔습니다!

`build` 함수는 `args`라는 매개변수로 슬라이스를 받지만, `build` 함수는 `args`를 소유하지 않습니다. `Config` 인스턴스를 반환하기 위해서는 `query`와 `file_path` 필드의 값을 복사해야 합니다. 이렇게 하면 `Config` 인스턴스가 값을 소유할 수 있습니다.

이터레이터에 대한 새로운 지식을 사용하면 `build` 함수를 이터레이터를 인자로 받도록 변경할 수 있습니다. 이터레이터 기능을 사용하여 슬라이스의 길이를 확인하고 특정 위치에 인덱싱하는 코드를 대체할 수 있습니다. 이렇게 하면 `Config::build` 함수가 무엇을 하는지 명확해집니다. 왜냐하면 이터레이터가 값에 액세스하기 때문입니다.

`Config::build` 함수가 이터레이터를 소유하게 되고 인덱싱 작업을 사용하지 않게 되면, 이터레이터에서 `String` 값을 `Config`로 이동시킬 수 있습니다. 이때 `clone`을 호출하고 새로운 할당을 생성하지 않아도 됩니다.

#### 반환된 이터레이터를 직접 사용하기

I/O 프로젝트의 *src/main.rs* 파일을 열고, 다음과 같이 보이는지 확인하십시오.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

이제 12-24번 목록에서 사용했던 코드를 13-18번 목록의 코드로 변경합니다. 이 코드는 이터레이터를 사용합니다. `Config::build` 함수를 업데이트할 때까지는 컴파일되지 않습니다.

<Listing number=\"13-18\" file-name=\"src/main.rs\" caption=\"`env::args`의 반환값을 `Config::build`에 전달하기\">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

`env::args` 함수는 이터레이터를 반환합니다! `env::args` 함수에서 반환되는 이터레이터 값을 벡터로 수집한 후 슬라이스를 `Config::build`에 전달하는 대신, 이제 `env::args`에서 반환되는 이터레이터의 소유권을 `Config::build`에 직접 전달합니다.

다음으로 `Config::build` 함수의 정의를 업데이트해야 합니다. I/O 프로젝트의 *src/lib.rs* 파일에서 `Config::build` 함수의 서명을 13-19번 목록과 같이 변경합니다. 이 또한 컴파일되지 않을 것입니다. 왜냐하면 함수 본문을 업데이트해야 하기 때문입니다.

<Listing number=\"13-19\" file-name=\"src/lib.rs\" caption=\"`Config::build` 함수의 서명을 이터레이터를 기대하도록 업데이트하기\">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/lib.rs:here}}
```

</Listing>

`env::args` 함수의 표준 라이브러리 문서는 이 함수가 반환하는 이터레이터의 유형이 `std::env::Args`이며, 이 유형이 이터레이터 인터페이스를 구현한다고 표시합니다.
 `Iterator` 트레이트를 사용하여 `String` 값을 반환합니다.

`Config::build` 함수의 서명을 업데이트하여 매개변수 `args`가 `impl Iterator<Item = String>`과 같은 트레이트 경계를 가진 일반 유형이 되도록 했습니다. 이 `impl Trait` 구문은 제10장의 "트레이트를 매개변수로 사용하기"<!-- ignore --> 섹션에서 논의한 것처럼 `args`가 `Iterator` 트레이트를 구현하고 `String` 항목을 반환하는 유형이 될 수 있음을 의미합니다.

`args`에 소유권을 가져가고 `args`를 반복하여 변경하기 때문에 `args` 매개변수의 명시에 `mut` 키워드를 추가하여 변경 가능하도록 할 수 있습니다.

#### `Iterator` 트레이트 메서드 사용

다음으로 `Config::build`의 바디를 수정합니다. `args`가 `Iterator` 트레이트를 구현하기 때문에 `next` 메서드를 호출할 수 있다는 것을 알 수 있습니다. 13-20번 목록은 12-23번 목록에서 `next` 메서드를 사용하는 코드로 업데이트합니다.

<Listing number=\"13-20\" file-name=\"src/lib.rs\" caption=\"`Config::build`의 바디를 이터레이터 메서드를 사용하여 변경\">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/lib.rs:here}}
```

</Listing>

`env::args`의 반환 값의 첫 번째 값은 프로그램의 이름입니다. 이를 무시하고 다음 값을 가져오려면 먼저 `next`를 호출하고 반환 값을 무시합니다. 두 번째로 `next`를 호출하여 `query` 필드에 들어갈 값을 가져옵니다. `next`가 `Some`을 반환하면 `match`를 사용하여 값을 추출합니다. `None`을 반환하면 인수가 충분하지 않아 `Err` 값으로 조기에 반환합니다. `file_path` 값에 대해서도 동일한 작업을 수행합니다.

### 이터레이터 어댑터를 사용하여 코드를 명확하게 만들기

또한 I/O 프로젝트의 `search` 함수에서도 이터레이터를 활용할 수 있습니다. 13-21번 목록은 12-19번 목록에서와 같이 `search` 함수를 재현합니다.

<Listing number=\"13-21\" file-name=\"src/lib.rs\" caption=\"12-19번 목록에서 `search` 함수의 구현\">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

이터레이터 어댑터 메서드를 사용하여 이 코드를 더 간결하게 작성할 수 있습니다. 이렇게 하면 변경 가능한 중간 `results` 벡터를 가지지 않아도 됩니다. 함수형 프로그래밍 스타일은 코드를 명확하게 하기 위해 변경 가능한 상태의 양을 최소화하는 것을 선호합니다. 변경 가능한 상태를 제거하면 `results` 벡터에 대한 동시 액세스를 관리하지 않아도 되므로 미래에 병렬 검색을 수행하는 것을 가능하게 할 수 있습니다.
13-22번 목록은 이 변경 사항을 보여줍니다.

<Listing number=\"13-22\" file-name=\"src/lib.rs\" caption=\"`search` 함수의 구현에서 이터레이터 어댑터 메서드 사용\">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

`search` 함수의 목적은 `contents`의 모든 줄에서 `query`를 포함하는 줄을 반환하는 것입니다. 13-16번 목록의 `filter` 예제와 유사하게 이 코드는 `filter` 어댑터를 사용하여 `line.contains(query)`가 `true`를 반환하는 줄만 유지합니다. 다음으로 `collect`를 사용하여 일치하는 줄을 다른 벡터에 수집합니다. 훨씬 간단합니다! `search_case_insensitive` 함수에서도 이터레이터 메서드를 사용하여 동일한 변경을 적용하는 것을 자유롭게 해보세요.

### 루프와 이터레이터 중 선택하기

다음 논리적인 질문은 자신의 코드에서 루프 스타일과 이터레이터 스타일 중 어떤 스타일을 선택해야 하는지, 그리고 왜 그런지에 대한 것입니다. 대부분의 Rust 프로그래머는 이터레이터 스타일을 선호합니다. 처음에는 익숙해지기가 조금 어려울 수 있지만, 다양한 이터레이터 어댑터와 그들이 하는 일에 대한 이해를 갖게 되면 이터레이터는 코드를 작성하는 데 더 쉽게 사용할 수 있습니다이해하기 위해서 반복문과 새로운 벡터를 만드는 여러 부분을 조작하는 대신, 코드는 반복문의 고수준 목표에 중점을 둡니다. 이는 일상적인 코드를 추상화하여 이 코드만의 독특한 개념, 예를 들어 이터레이터의 각 요소가 통과해야 하는 필터링 조건과 같은 것을 더 쉽게 볼 수 있게 합니다.

하지만 두 구현은 정말로 동등한가요? 직관적인 가정은 더 저수준의 루프가 더 빠를 것이라는 것입니다. 성능에 대해 이야기해 보겠습니다.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
