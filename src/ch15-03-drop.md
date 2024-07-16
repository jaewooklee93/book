## 소멸 시 코드 실행: `Drop` 트레이트

스마트 포인터 패턴에서 중요한 두 번째 트레이트는 `Drop`입니다. 이 트레이트를 통해 값이 범위 밖으로 나갈 때 실행할 코드를 커스터마이징할 수 있습니다. 어떤 타입에 대해 `Drop` 트레이트의 구현을 제공하면 해당 코드를 사용하여 파일이나 네트워크 연결과 같은 리소스를 해제할 수 있습니다.

`Drop` 트레이트를 스마트 포인터의 맥락에서 소개하는 이유는 `Drop` 트레이트의 기능이 거의 항상 스마트 포인터를 구현할 때 사용되기 때문입니다. 예를 들어, `Box<T>`가 삭제될 때는 박스가 가리키는 해프의 공간을 해제합니다.

일부 언어에서는 일부 타입에 대해 프로그래머가 해당 타입의 인스턴스를 사용한 후마다 메모리나 리소스를 해제하는 코드를 호출해야 합니다. 예는 파일 핸들, 소켓 또는 잠금입니다. 이를 잊으면 시스템이 과부하되어 충돌할 수 있습니다. Rust에서는 특정 값이 범위 밖으로 나갈 때 실행할 코드를 지정할 수 있으며, 컴파일러가 이 코드를 자동으로 삽입합니다. 따라서 특정 타입의 인스턴스가 사용이 끝났을 때 프로그램의 모든 곳에 클리너 코드를 놓아두는 데 신경 쓰지 않아도 되며 리소스가 누출되지 않습니다!

값이 범위 밖으로 나갈 때 실행할 코드를 지정하려면 `Drop` 트레이트를 구현합니다. `Drop` 트레이트는 `self`에 대한 가변 참조를 받는 `drop`이라는 메서드를 구현하도록 요구합니다. Rust가 `drop`을 언제 호출하는지 확인하려면 현재 `println!` 문을 사용하여 `drop`을 구현해 보겠습니다.

15-14번 목록은 `CustomSmartPointer` 구조체를 보여주는데, 유일한 사용자 정의 기능은 인스턴스가 범위 밖으로 나갈 때 `Dropping CustomSmartPointer!`를 출력하여 Rust가 `drop` 함수를 실행할 때를 보여주는 것입니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

15-14번 목록: 범위 밖으로 나갈 때 클리너 코드를 넣을 `CustomSmartPointer` 구조체

`Drop` 트레이트는 예시에 포함되어 있으므로 스코프에 가져오지 않아도 됩니다. `CustomSmartPointer`에 `Drop` 트레이트를 구현하고 `drop` 메서드에 `println!`을 호출하는 구현을 제공합니다. `drop` 함수의 본문은 인스턴스가 범위 밖으로 나갈 때 실행할 코드를 넣을 곳입니다. 현재는 시각적으로 Rust가 `drop`를 언제 호출하는지 보여주기 위해 텍스트를 출력하고 있습니다.

`main`에서 `CustomSmartPointer`의 두 인스턴스를 생성한 후 `CustomSmartPointers created`를 출력합니다. `main`의 끝에서 `CustomSmartPointer`의 인스턴스가 범위 밖으로 나가고 Rust가 `drop` 메서드에서 지정한 코드를 호출합니다. `drop` 메서드를 명시적으로 호출할 필요가 없습니다.

이 프로그램을 실행하면 다음과 같은 출력이 나타납니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust는 인스턴스가 범위 밖으로 나갈 때 자동으로 `drop`을 호출하여 지정한 코드를 실행했습니다. 변수는 생성 순서의 역순으로 삭제되므로 `d`는 `c`보다 먼저 삭제되었습니다. 이 예제의 목적은 `drop` 메서드가 어떻게 작동하는지 시각적으로 보여주는 것입니다. 일반적으로는 `println!` 메시지 대신 타입이 필요로 하는 클리너 코드를 지정할 것입니다.

### `std::mem::drop`을 사용하여 값을 일찍 삭제하기

불행히도 `Drop` 트레이트의 `drop` 메서드를 수동으로 호출하는 것은 직관적이지 않습니다. `Drop` 트레이트의 `drop` 메서드를 호출하는 것은 일반적으로 필요하지 않습니다. `Drop` 트레이트의 전체 목적은 자동으로 처리되는 것입니다. 때때로는 스마트 포인터와 같은 리소스를 관리하는 경우, 잠금을 해제하는 `drop` 메서드를 강제로 호출하여 동일한 범위 내의 다른 코드가 잠금을 획득할 수 있도록 하고 싶을 수 있습니다. Rust는 `Drop` 트레이트의 `drop` 메서드를 수동으로 호출하지 못하도록 하며, 대신 표준 라이브러리에서 제공되는 `std::mem::drop` 함수를 사용해야 합니다. 만약 범위 밖으로 나갈 때까지 값을 삭제해야 한다면.

만약 `Drop` 트레이트의 `drop` 메서드를 수동으로 호출하려고 시도하면 Rust는 에러를 발생시킵니다. `std::mem::drop` 함수를 사용하여 값을 강제로 삭제해야 합니다.

Listing 15-14에서 보여진 `main` 함수를 참고하여 Listing 15-15와 같이 작성하면 컴파일러 오류가 발생합니다.

Filename: src/main.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

Listing 15-15: `Drop` 트레이트의 `drop` 메서드를 수동으로 호출하여 일찍 정리하려는 시도

이 코드를 컴파일하려고 시도하면 다음과 같은 오류 메시지가 표시됩니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

이 오류 메시지는 `drop`를 명시적으로 호출할 수 없음을 나타냅니다. 오류 메시지는 *소멸자*라는 용어를 사용하며, 이는 인스턴스를 정리하는 함수를 일반적으로 의미합니다. *소멸자*는 인스턴스를 생성하는 *생성자*와 유사합니다. Rust의 `drop` 함수는 특정 소멸자입니다.

Rust는 `drop`를 명시적으로 호출하지 못하게 하는 이유는 Rust가 `main` 함수의 끝에서 값에 대해 자동으로 `drop`을 호출하기 때문입니다. 이는 Rust가 동일한 값을 두 번 청소하려고 시도하기 때문에 *중복 해제* 오류가 발생하기 때문입니다.

값이 범위를 벗어날 때 `drop`의 자동 삽입을 비활성화하거나 `drop` 메서드를 명시적으로 호출할 수 없습니다. 따라서 값을 일찍 강제로 청소해야 하는 경우 `std::mem::drop` 함수를 사용합니다.

`std::mem::drop` 함수는 `Drop` 트레이트의 `drop` 메서드와 다릅니다. 값을 강제로 삭제하려면 `drop` 함수를 호출하여 해당 값을 인자로 전달합니다. 이 함수는 prelude에 있으므로 Listing 15-15의 `main` 함수를 수정하여 `drop` 함수를 호출할 수 있습니다. Listing 15-16과 같이:

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

Listing 15-16: `std::mem::drop`을 사용하여 값이 범위를 벗어나기 전에 명시적으로 삭제

이 코드를 실행하면 다음과 같은 출력이 표시됩니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

`CustomSmartPointer created.`와 `CustomSmartPointer dropped before the end of main.` 텍스트 사이에 `Dropping CustomSmartPointer with data `some data`!` 텍스트가 표시되어 `drop` 메서드 코드가 `c`를 삭제하기 위해 호출됨을 보여줍니다.

`Drop` 트레이트 구현에서 지정된 코드를 사용하여 정리 작업을 편리하고 안전하게 수행할 수 있습니다. 예를 들어, 자신의 메모리 할당기를 만들 수 있습니다! `Drop` 트레이트와 Rust의 소유권 시스템을 사용하면 Rust가 자동으로 청소하므로 청소 작업을 기억할 필요가 없습니다.

소유권 시스템이 참조가 항상 유효하도록 보장하는 것처럼 `drop`가 값이 더 이상 사용되지 않을 때만 한 번 호출되도록 보장합니다.

이제 `Box<T>`를 살펴보고 스마트 포인터의 특징을 살펴보았으므로 표준 라이브러리에서 정의된 다른 스마트 포인터 몇 가지를 살펴보겠습니다.
