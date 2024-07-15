## 동시에 코드를 실행하는 데 사용되는 스레드

대부분의 현대 운영 체제에서 실행되는 프로그램의 코드는
*프로세스*에서 실행되며, 운영 체제는 여러 프로세스를 동시에 관리합니다.
프로그램 내부에서도 동시에 실행될 수 있는 독립적인 부분이 있습니다.
이러한 독립적인 부분을 실행하는 기능을 *스레드*라고 합니다. 예를 들어, 웹 서버는 여러 스레드를 가질 수 있으며, 이를 통해 동시에 여러 요청에 응답할 수 있습니다.

프로그램의 계산을 여러 스레드로 분할하여 여러 작업을 동시에 실행하면 성능이 향상될 수 있지만, 복잡성도 증가합니다.
스레드가 동시에 실행되기 때문에 코드의 다른 스레드에서 실행되는 부분의 순서에 대해서는 보장되지 않습니다. 이는 다음과 같은 문제로 이어질 수 있습니다.

* 레이스 조건: 스레드가 데이터나 리소스를 일관되지 않은 순서로 액세스하는 경우
* 데드락: 두 스레드가 서로를 기다리는 경우, 두 스레드 모두 계속 진행하지 못하는 경우
* 특정 상황에서만 발생하는 버그로, 재현 및 수정이 어려운 경우

Rust는 스레드 사용의 부정적인 영향을 완화하려고 노력하지만, 다중 스레드 환경에서 프로그래밍은 여전히 신중한 사고를 필요로 하며, 단일 스레드 프로그램과 다른 코드 구조가 필요합니다.

프로그래밍 언어는 스레드를 다양한 방식으로 구현하며, 많은 운영 체제는 언어가 사용할 수 있는 API를 제공합니다. Rust 표준 라이브러리는 1:1 스레드 구현 모델을 사용하며, 프로그램이 하나의 언어 스레드에 대해 하나의 운영 체제 스레드를 사용합니다. 다른 스레드 모델을 구현하는 crate도 있으며, 이러한 crate는 1:1 모델과 다른 트레이드오프를 제공합니다.

### `spawn`을 사용하여 새 스레드 생성

새 스레드를 생성하려면 `thread::spawn` 함수를 호출하고, 새 스레드에서 실행할 코드를 포함하는 클로저(Chapter 13에서 다뤘던 클로저)를 전달합니다. Listing 16-1은 메인 스레드에서 텍스트를 출력하고 새 스레드에서 다른 텍스트를 출력하는 예입니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

<span class=\"caption\">Listing 16-1: 메인 스레드에서 하나의 것을 출력하는 새 스레드를 생성
동시에 다른 스레드에서 다른 것을 출력</span>

이 프로그램의 출력은 매번 약간 다를 수 있지만, 다음과 같이 보일 것입니다.

<!-- 출력을 추출하지 않음. 출력의 변화는 중요하지 않으며,
변화는 스레드가 다르게 실행되는 것 때문일 가능성이 높습니다. -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

`thread::sleep` 호출은 스레드가 짧은 시간 동안 실행을 중지하도록 강제하여 다른 스레드가 실행될 수 있도록 합니다. 스레드는 아마 번갈아가며 실행될 것입니다. 그러나 그것은 보장되지 않습니다. 운영 체제가 스레드를 스케줄링하는 방식에 따라 달라집니다. 이 실행에서는 메인 스레드가 먼저 출력했지만, 생성된 스레드의 출력 문이 코드에서 먼저 나타납니다. 또한 생성된 스레드가 `i`가 9까지 출력하도록 지시했지만, 메인 스레드가 종료되기 전에 5까지만 출력되었습니다.

이 코드를 실행하고 메인 스레드에서만 출력이 나타나거나, 겹치는 부분이 보이지 않으면, 스레드가 실행될 기회를 더 많이 만들기 위해 범위 내의 숫자를 늘려보세요.

### `join`을 사용하여 모든 스레드가 완료될 때까지 기다리기

Listing 16-2는 `thread::spawn`을 사용하여 생성된 스레드가 완료될 때까지 기다리는 방법을 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

<span class=\"caption\">Listing 16-2: 모든 스레드가 완료될 때까지 기다리는 방법</span>

`thread::spawn` 함수는 `JoinHandle`를 반환합니다. `JoinHandle`는 스레드가 완료될 때까지 기다리는 데 사용되는 소유 값입니다. `join` 메서드를 호출하면 스레드가 완료될 때까지 기다립니다.

이 코드를 실행하면 메인 스레드가 모든 스레드가 완료될 때까지 기다리기 때문에, 모든 스레드에서 출력이 나타납니다.

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

16-1절에서 만든 스레드의 `JoinHandle`을 사용하는 방법과
`join`을 호출하여 스레드가 `main`이 종료하기 전에 완료되도록 하는 방법을 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

<span class=\"caption\">Listing 16-2: 스레드가 완료될 때까지 `JoinHandle`을 저장하여 보장</span>

`join`을 해당 핸들에 호출하면 현재 실행 중인 스레드가 해당 핸들로 표현되는 스레드가 종료될 때까지 차단됩니다. 스레드를 차단하는 것은 스레드가 작업을 수행하거나 종료하는 것을 방지하는 것을 의미합니다. `main` 스레드의 `for` 루프 뒤에 `join` 호출을 넣었기 때문에 16-2번 목록을 실행하면 다음과 같은 출력이 생성됩니다.

<!-- 출력을 추출하지 않음. 출력의 변화는 스레드가 다르게 실행되는 것 때문일 가능성이 높습니다. -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

두 스레드는 계속 번갈아 실행되지만, `handle.join()` 호출 때문에 메인 스레드가 기다리고 메인 스레드가 완료될 때까지 스레드가 완료될 때까지 기다립니다.

하지만 `handle.join()`을 `for` 루프 앞으로 옮기면 어떻게 될까요? 다음과 같이:

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

메인 스레드는 스레드가 완료될 때까지 기다리고 `for` 루프를 실행하므로 출력이 더 이상 겹치지 않습니다. 다음과 같이 보입니다.

<!-- 출력을 추출하지 않음. 출력의 변화는 스레드가 다르게 실행되는 것 때문일 가능성이 높습니다. -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

`join`이 호출되는 위치와 같은 작은 세부 사항이 스레드가 동시에 실행되는지 여부에 영향을 미칠 수 있습니다.

### `move` Closure를 사용하여 스레드

`thread::spawn`에 전달되는 closure에 `move` 키워드를 사용하는 경우가 많습니다. 이는 closure가 사용하는 값에 대한 소유권을 가져와서 메인 스레드에서 생성된 값의 소유권을 다른 스레드로 전달하기 때문입니다. 13장의 `\u201cCapturing References or Moving Ownership\u201d` 부분에서 `move`를 closure의 맥락에서 논의했습니다. 이제 `move`와 `thread::spawn`의 상호 작용에 더 집중해 보겠습니다.

16-1절에서 보는 것처럼 `thread::spawn`에 전달하는 closure는 인수를 받지 않습니다. 즉, 생성된 스레드에서 메인 스레드의 데이터를 사용하지 않습니다. 생성된 스레드에서 메인 스레드의 데이터를 사용하려면 생성된 스레드의 closure가 필요한 값을 캡처해야 합니다. 16-3번 목록은 메인 스레드에서 벡터를 생성하고 생성된 스레드에서 사용하려는 시도를 보여줍니다. 그러나 아직 작동하지 않습니다. 다음은 이유입니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
## 16.1. 스레드"
Listing 16-4와 동일하게 변경하면, 메인 스레드에서 `v`를 사용하려고 할 때 소유권 규칙을 위반하게 됩니다. `move` 키워드는 Rust의 보수적인 대출 기본값을 무효화합니다. 대출을 허용하지 않고 소유권 규칙을 위반하지 않도록 합니다.

스레드와 스레드 API에 대한 기본적인 이해를 바탕으로, 스레드로 무엇을 할 수 있는지 살펴보겠습니다.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
