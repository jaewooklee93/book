## 참조와 대여

4-5번 목록의 튜플 코드의 문제는 `String`을 반환해야 하기 때문입니다. `calculate_length` 함수 호출 후에도 `String`을 사용할 수 있기 때문입니다. `String`이 `calculate_length`로 이동되었기 때문입니다. 대신, `String` 값에 대한 참조를 제공할 수 있습니다.
A *참조*는 포인터와 같이 특정 데이터에 접근할 수 있는 주소입니다. 그러나 해당 데이터는 다른 변수가 소유하고 있습니다.
포인터와 달리, 참조는 해당 참조의 수명 동안 특정 유형의 유효한 값을 가리키는 것을 보장합니다.

`calculate_length` 함수를 정의하고 사용하는 방법은 다음과 같습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

첫째, 변수 선언 및 함수 반환 값에 있는 모든 튜플 코드가 사라졌습니다. 둘째, `&s1`을 `calculate_length`에 전달하고, 정의에서 `&String`을 `String` 대신 사용하는 것을 알 수 있습니다. 이러한 `&` 기호는 *참조*를 나타내며, 값을 소유하지 않고도 참조할 수 있도록 합니다. 그림 4-5는 이 개념을 보여줍니다.

<img alt=\"Three tables: the table for s contains only a pointer to the table
for s1. The table for s1 contains the stack data for s1 and points to the
string data on the heap.\" src=\"img/trpl04-05.svg\" class=\"center\" />

<span class=\"caption\">Figure 4-5: `&String s`가 `String s1`을 가리키는 그림</span>

> 참고: `&`를 사용하여 참조하는 것의 반대는 *해제*이며, 해제 연산자 `*`로 이루어집니다. 해제 연산자에 대한 자세한 내용은 8장에서 살펴보고, 15장에서 해제의 세부 사항을 논의합니다.

`calculate_length` 함수 호출을 자세히 살펴보겠습니다.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

`&s1` 문법은 `s1`의 값을 *참조*하는 참조를 만드는 데 사용됩니다. 하지만 `s1`을 소유하지는 않습니다. 소유하지 않기 때문에 참조가 사용되지 않을 때 가리키는 값은 삭제되지 않습니다.

함수의 서명은 `&`를 사용하여 매개변수 `s`의 유형이 참조임을 나타냅니다. `s` 변수의 유효 범위는 함수 매개변수와 동일하지만, 참조가 가리키는 값은 `s`가 사용되지 않을 때 삭제되지 않습니다. 함수에 참조가 매개변수로 사용되면 실제 값을 반환할 필요가 없습니다. 왜냐하면 소유권을 가지고 있지 않았기 때문입니다.

참조를 만드는 작업을 *대여*라고 합니다. 실생활에서 주인이 소유하는 것을 대여받는 것과 같습니다. 대여받은 후에는 반납해야 합니다. 소유하지 않습니다.

그렇다면 대여받은 것을 수정하려고 시도하면 어떻게 될까요? 4-6번 목록의 코드를 시도해 보세요. 스포일러 알림: 작동하지 않습니다!

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

<span class=\"caption\">Listing 4-6: 대여받은 값을 수정하려는 시도</span>

오류 메시지는 다음과 같습니다.

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

변수가 기본적으로 불변과 같이 참조도 불변입니다. 참조를 가리키는 것을 수정할 수 없습니다.

### 변경 가능한 참조

Listing 4-6의 코드를 수정하여 변경 가능한 값을 참조하여 수정할 수 있습니다.
몇 가지 작은 수정을 사용하여 *변경 가능한 참조*를 사용하면 됩니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

먼저 `s`를 `mut`로 변경합니다. 그런 다음 `&mut s`를 사용하여 변경 가능한 참조를 생성하고 `change` 함수 호출 시 `some_string: &mut String`으로 함수 시그니처를 변경합니다. 이를 통해 `change` 함수가 가져오는 값을 변경할 것이라는 점이 명확해집니다.

변경 가능한 참조에는 하나의 큰 제한이 있습니다. 변경 가능한 참조를 가진 경우 해당 값에 다른 참조를 가질 수 없습니다. 다음 코드는 `s`에 두 개의 변경 가능한 참조를 생성하려고 하므로 오류가 발생합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

다음은 오류 메시지입니다.

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

이 오류 메시지는 현재 코드가 유효하지 않기 때문에 `s`에 변경 가능한 참조를 한 번 이상 동시에 가져올 수 없기 때문이라고 말합니다. 첫 번째 변경 가능한 참조는 `r1`에서 생성되며 `println!`에서 사용될 때까지 유지되어야 하지만, 변경 가능한 참조가 생성되고 사용되기 전에 `r2`에서 동일한 데이터를 가져오려는 또 다른 변경 가능한 참조를 생성하려고 시도했습니다.

변경 가능한 참조를 동시에 여러 개 가져올 수 없는 제한은 변경을 매우 제어된 방식으로 허용합니다. Rustacean이 처음 접하는 경우에는 대부분의 언어에서 원하는 대로 변경할 수 있기 때문에 어려움을 겪을 수 있습니다. 하지만 이 제한이 있는 이유는 Rust가 컴파일 시간에 데이터 레이스를 방지할 수 있기 때문입니다. *데이터 레이스*는 포인터가 동시에 동일한 데이터에 액세스하는 경우 발생하는 오류와 유사하며 다음 세 가지 행동이 발생할 때 발생합니다.

* 두 개 이상의 포인터가 동시에 동일한 데이터에 액세스합니다.
* 포인터 중 적어도 하나가 데이터에 쓰기 위해 사용됩니다.
* 데이터에 대한 액세스를 동기화하는 메커니즘이 없습니다.

데이터 레이스는 정의되지 않은 동작을 일으키고 실행 시간에 추적하고 수정하는 것이 어려울 수 있습니다. Rust는 데이터 레이스가 있는 코드를 컴파일하지 않도록 하여 이러한 문제를 방지합니다!

언제나 마찬가지로, 새 스코프를 만들기 위해 괄호를 사용하면 여러 개의 변경 가능한 참조를 사용할 수 있습니다. 단, 동시에 사용할 수는 없습니다.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust는 변경 가능한 참조와 불변 참조를 결합하는 것에 대해서도 유사한 규칙을 적용합니다. 다음 코드는 오류를 발생시킵니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

다음은 오류 메시지입니다.

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

우와! 동일한 값에 대해 불변 참조와 변경 가능한 참조를 동시에 가질 수 없습니다.

불변 참조를 사용하는 사용자는 값이 갑자기 바뀌지 않기를 기대하지 않습니다. 그러나 여러 개의 불변 참조는 허용됩니다. 왜냐하면 읽기만 하는 사람은 누구도 데이터를 읽는 다른 사람에게 영향을 미칠 수 없기 때문입니다.

참고로 참조의 범위는 소개된 위치에서 시작하여 마지막으로 참조가 사용될 때까지 지속됩니다. 예를 들어, 다음 코드는 불변 참조 `r1`과 `r2`의 마지막 사용이 `println!` 이후에 변경 가능한 참조가 소개되기 때문에 컴파일됩니다.

```rust,edition2021
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

`r1`과 `r2`의 불변 참조 범위는 `println!` 이후에 종료됩니다참조와 대여를 사용하는 방법을 살펴보겠습니다. 참조는 데이터에 대한 별도의 포인터와 유사하지만, Rust에서 참조는 데이터의 소유권을 갖지 않습니다. 대신, 데이터에 대한 참조만을 가리킵니다. 이는 Rust의 소유권 시스템과 깊은 관련이 있습니다.

Rust에서 참조는 데이터의 소유권을 갖지 않기 때문에, 데이터가 소멸되기 전에 참조가 유효해야 합니다. 이를 방지하기 위해 Rust 컴파일러는 참조가 유효하지 않을 때 오류를 발생시킵니다.

**예시:**

```rust
let s = String::from("hello");

let r1 = &s; // immutable reference
let r2 = &s; // another immutable reference

let r3 = &mut s; // mutable reference

println!("{}, {}, {}", r1, r2, r3);
```

이 코드에서는 `s`에 대한 두 개의 불변 참조 (`r1`과 `r2`)와 하나의 가변 참조 (`r3`)가 생성됩니다. 컴파일러는 `r3`가 생성되기 전에 `r1`과 `r2`가 사용되지 않는다는 것을 알 수 있습니다.

**참조 오류:**

참조 오류는 일반적으로 데이터가 소멸되기 전에 참조가 유효하지 않을 때 발생합니다. 예를 들어, 다음 코드는 오류를 발생시킵니다.

```rust
let s = String::from("hello");

let r1 = &s;

let r2 = &mut s; // Error!

println!("{}, {}", r1, r2);
```

이 코드에서는 `s`에 대한 불변 참조 (`r1`)와 가변 참조 (`r2`)가 동시에 생성됩니다. 컴파일러는 `r2`가 생성되기 전에 `r1`이 사용되기 때문에 이 코드가 오류를 발생시킨다고 판단합니다.

**Dangling References:**

포인터를 사용하는 언어에서는, 메모리 할당이 해제된 후에도 포인터가 해당 메모리 주소를 가리키는 경우 *dangling pointer*가 발생합니다. Rust에서는 컴파일러가 참조가 유효하지 않을 때 오류를 발생시켜 dangling pointer를 방지합니다.

```rust
fn dangle() -> &String {
    let s = String::from("hello");
    &s
}

fn main() {
    let r = dangle();
    println!("{}", r);
}
```

이 코드는 `dangle` 함수에서 `String`을 반환하는데, `dangle` 함수가 종료되면 `s`는 소멸됩니다. 그러나 `dangle` 함수는 `&s`를 반환하기 때문에, `r`은 소멸된 `String`을 가리키는 dangling reference가 됩니다. Rust 컴파일러는 이를 방지하기 위해 오류를 발생시킵니다.

**참조 규칙:**

* 특정 시점에서, 하나의 가변 참조 또는 여러 개의 불변 참조를 가질 수 있습니다.
* 참조는 항상 유효해야 합니다.



