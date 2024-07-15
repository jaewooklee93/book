##  단일 스레드 서버를 다중 스레드 서버로 변환하기

현재 서버는 각 요청을 차례로 처리하기 때문에, 첫 번째 요청이 완료되지 않으면 두 번째 연결을 처리하지 않습니다. 서버에 요청이 점점 더 많이 들어오면 이 시리얼 실행은 점점 더 비효율적입니다. 서버에 오랫동안 처리되는 요청이 들어오면 새 요청은 오랫동안 처리되는 요청이 완료될 때까지 기다려야 합니다. 이를 해결해야 합니다. 하지만 먼저 문제 상황을 살펴보겠습니다.

### 현재 서버 구현에서 느린 요청을 시뮬레이션하기

우리는 현재 서버 구현에 느리게 처리되는 요청이 다른 요청에 어떤 영향을 미치는지 살펴보겠습니다. 20-10번 목록은 *sleep* 요청을 처리하는 방법을 구현합니다. 이는 서버가 응답하기 전에 5초 동안 수면을 취하는 시뮬레이션 느린 응답을 제공합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-10/src/main.rs:here}}
```

<span class=\"caption\">Listing 20-10: 5초 동안 수면을 취하여 느린 요청을 시뮬레이션하기</span>

우리는 이제 세 가지 경우를 처리하기 위해 `if`를 `match`로 변경했습니다. `match`는 `=` 연산자처럼 자동 참조 및 해제를 하지 않기 때문에 `request_line` 슬라이스에 대한 명시적인 패턴 매칭이 필요합니다.

첫 번째 팔은 20-9번 목록의 `if` 블록과 동일합니다. 두 번째 팔은 *sleep* 요청에 일치합니다. 해당 요청이 수신되면 서버는 응답하기 전에 5초 동안 수면을 취합니다. 세 번째 팔은 20-9번 목록의 `else` 블록과 동일합니다.

서버가 얼마나 기본적인지 알 수 있습니다. 실제 라이브러리는 여러 요청을 인식하는 방법을 훨씬 간결하게 처리할 것입니다.

`cargo run`을 사용하여 서버를 시작합니다. 그런 다음 *http://127.0.0.1:7878/* 와 *http://127.0.0.1:7878/sleep* 에 대한 두 개의 브라우저 창을 엽니다. */* URI를 여러 번 입력하면 이전과 같이 빠르게 응답합니다. 그러나 *sleep* 를 입력하고 */* 를 로드하면 */* 가 *sleep* 가 5초 동안 수면을 취하기 전까지 로드되지 않습니다.

느린 요청 뒤로 요청이 쌓이는 것을 방지하기 위해 사용할 수 있는 여러 가지 기술이 있습니다. 우리가 구현할 기술은 스레드 풀입니다.

### 스레드 풀을 사용하여 처리량 향상

*스레드 풀*은 작업을 처리할 준비가 되어 있는 스레드의 그룹입니다. 프로그램이 새 작업을 수신하면 스레드 풀에서 스레드 하나를 작업에 할당하고 해당 스레드가 작업을 처리합니다. 스레드 풀의 나머지 스레드는 첫 번째 스레드가 작업을 처리하는 동안 들어오는 다른 작업을 처리할 수 있습니다. 첫 번째 스레드가 작업을 처리하면 스레드 풀로 되돌려져 새로운 작업을 처리할 준비가 됩니다. 스레드 풀은 연결을 동시에 처리할 수 있도록 하여 서버의 처리량을 높일 수 있습니다.

스레드 풀의 스레드 수를 제한하여 Denial of Service(DoS) 공격으로부터 자신을 보호합니다. 각 요청이 들어오면 새 스레드를 생성한다면 1000만 개의 요청을 서버에 보내는 사람이 서버의 모든 자원을 사용하여 요청 처리를 완전히 중단시킬 수 있습니다.

따라서 제한된 스레드 수를 유지하여 들어오는 요청을 처리하는 대신, 요청을 처리하는 스레드가 있는 큐를 유지합니다. 스레드 풀의 각 스레드는 이 큐에서 요청을 꺼내 처리하고 다시 큐에 요청을 요청합니다. 이러한 설계를 통해 `N` 개의 요청을 동시에 처리할 수 있습니다. `N`은 스레드 수입니다. 각 스레드가 오랫동안 처리되는 요청에 응답하고 있다면, 이후 요청은 여전히 큐에 쌓일 수 있지만, 해당 지점에 도달하기 전에 처리할 수 있는 오랫동안 처리되는 요청의 수를 늘렸습니다.

이 기술은 웹 서버의 처리량을 향상시키는 데 사용할 수 있는 여러 가지 방법 중 하나입니다. 다른 옵션으로는 *fork/join 모델*, *asynchronous I/O* 등이 있습니다.

다음과 같은 두 가지 모델이 있습니다: *단일 스레드 비동기 I/O 모델* 또는 *다중 스레드 비동기 I/O 모델*. 이 주제에 관심이 있다면 다른 솔루션에 대해 더 많이 읽고 구현해 보세요. Rust와 같은 저수준 언어에서는 이러한 모든 옵션이 가능합니다.

스레드 풀을 구현하기 전에 스레드 풀을 사용하는 방식에 대해 이야기해 보겠습니다. 코드를 설계할 때 클라이언트 인터페이스를 먼저 작성하면 디자인을 안내하는 데 도움이 될 수 있습니다. 코드를 호출하는 방식으로 구조화된 API를 작성한 다음 해당 구조 내에서 기능을 구현하는 것이 좋습니다.

제12장의 프로젝트에서 테스트 주도 개발을 사용했던 것처럼, 여기에서는 컴파일러 주도 개발을 사용할 것입니다. 원하는 함수를 호출하는 코드를 작성하고, 컴파일러의 오류를 살펴보면 코드가 작동하도록 변경해야 할 부분을 파악할 수 있습니다. 그러나 이를 하기 전에 사용하지 않을 기술을 살펴보겠습니다.

<!-- 이전 제목. 링크가 깨지지 않도록 제거하지 마세요. -->
<a id=\"code-structure-if-we-could-spawn-a-thread-for-each-request\"></a>

#### 각 요청에 대해 스레드를 생성하는 경우

먼저, 각 연결에 대해 새 스레드를 생성하는 방식으로 코드가 어떻게 보일지 살펴보겠습니다. 앞서 언급했듯이, 제한 없는 수의 스레드를 생성할 수 있다는 문제 때문에 이것이 최종 계획은 아니지만, 먼저 작동하는 다중 스레드 서버를 얻기 위한 시작점입니다. 그런 다음 스레드 풀을 추가하여 개선하고, 두 가지 솔루션을 비교하면 쉬워집니다. 20-11번 표에서 `main`을 수정하여 `for` 루프 내에서 각 스트림을 처리하는 새 스레드를 생성하는 방법을 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-11/src/main.rs:here}}
```

<span class=\"caption\">Listing 20-11: 각 스트림에 대해 새 스레드를 생성하는 경우</span>

제16장에서 배운 것처럼 `thread::spawn`은 새 스레드를 생성하고 폐쇄 코드를 새 스레드에서 실행합니다. 이 코드를 실행하고 브라우저에 */sleep*를 로드한 다음 두 개의 브라우저 탭에서 */*를 로드하면 */sleep*가 완료될 때까지 기다리지 않고 */*에 대한 요청이 처리되는 것을 확인할 수 있습니다. 그러나 앞서 언급했듯이, 제한 없이 새 스레드를 생성하므로 시스템을 궁극적으로 압도하게 됩니다.

<!-- 이전 제목. 링크가 깨지지 않도록 제거하지 마세요. -->
<a id=\"creating-a-similar-interface-for-a-finite-number-of-threads\"></a>

#### 유한 개수의 스레드를 사용하는 경우

스레드 풀이 동일한 방식으로 작동하도록 하여 스레드에서 스레드 풀로 전환할 때 API를 사용하는 코드에 큰 변경이 필요하지 않도록 합니다. 20-12번 표는 사용하려는 `ThreadPool` 구조의 가상 인터페이스를 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/listing-20-12/src/main.rs:here}}
```

<span class=\"caption\">Listing 20-12: 우리가 원하는 `ThreadPool` 인터페이스</span>

`ThreadPool::new`를 사용하여 스레드 수가 4개인 스레드 풀을 생성합니다. 그런 다음 `for` 루프에서 `pool.execute`는 `thread::spawn`과 유사한 인터페이스를 가지고 있으며, 스레드 풀이 실행해야 하는 폐쇄를 가져옵니다. 스레드 풀에 폐쇄를 전달하고 실행하도록 하는 `pool.execute`를 구현해야 합니다. 이 코드는 아직 컴파일되지 않지만, 컴파일러가 우리를 도와 문제를 해결하는 방향으로 이끌어 줄 것입니다.

<!-- 이전 제목. 링크가 깨지지 않도록 제거하지 마세요. -->
<a id=\"building-the-threadpool-struct-using-compiler-driven-development\"></a>

#### 컴파일러 주도 개발을 사용하여 `ThreadPool` 구축하기

20-12번 표의 변경 사항을 *src/main.rs*에 적용하고, 컴파일러 주도 개발을 사용하여 `ThreadPool`를 구현해 보겠습니다.


컴파일러 오류에서 `cargo check`를 통해 개발을 이끌어갈 것입니다. 첫 번째 오류는 다음과 같습니다.

```console
{{#include ../listings/ch20-web-server/listing-20-12/output.txt}}
```

훌륭합니다! 이 오류는 `ThreadPool` 유형 또는 모듈이 필요하다는 것을 알려줍니다. 따라서 지금은 `ThreadPool`을 구축할 것입니다. `ThreadPool` 구현은 웹 서버가 수행하는 작업의 종류와 관계없이 독립적일 것입니다. 따라서 `hello` crate를 바이너리 crate에서 라이브러리 crate로 변경하여 `ThreadPool` 구현을 저장하겠습니다. 라이브러리 crate로 변경한 후에는 스레드 풀 라이브러리를 사용하여 스레드 풀을 사용하여 수행하려는 작업을 웹 요청을 처리하는 것뿐만 아니라 사용할 수 있습니다.

`src/lib.rs`를 생성하여 현재 가질 수 있는 가장 간단한 `ThreadPool` 구조의 정의를 포함합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

그런 다음 `ThreadPool`을 라이브러리 crate에서 가져오도록 `main.rs` 파일을 편집하여 `src/main.rs`의 맨 위에 다음 코드를 추가합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

이 코드는 여전히 작동하지 않지만 다음 오류를 해결해야 할 다음 단계를 확인해 보겠습니다.

```console
{{#include ../listings/ch20-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

이 오류는 다음 단계로 `ThreadPool`에 `new`이라는 연관 함수를 만들어야 함을 나타냅니다. 또한 `new`가 `4`를 인수로 받을 수 있는 하나의 매개변수를 가져야 하며 `ThreadPool` 인스턴스를 반환해야 한다는 것을 알 수 있습니다. 가장 간단한 `new` 함수를 구현하여 이러한 특징을 가질 수 있습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

`size` 매개변수의 유형으로 `usize`를 선택했습니다. 왜냐하면 음수의 스레드 수는 의미가 없기 때문입니다. 또한 이 `4`를 스레드의 컬렉션의 요소 수로 사용할 것이라는 것을 알고 있기 때문입니다. `usize` 유형은 제3장의 [\u201c정수 유형\u201d][integer-types]<!--
ignore --> 섹션에서 논의된 것처럼 스레드 수를 나타내는 데 적합합니다.

다음과 같이 코드를 다시 확인해 보겠습니다.

```console
{{#include ../listings/ch20-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

이제 오류는 `ThreadPool`에 `execute` 메서드가 없기 때문에 발생합니다. [\u201c유한 개수의 스레드 생성\u201d](#creating-a-finite-number-of-threads)<!-- ignore --> 섹션에서 우리가 스레드 풀이 `thread::spawn`과 유사한 인터페이스를 가져야 한다고 결정했기 때문에 이를 기억하십시오. `execute` 함수를 구현하여 주어진 콜로저를 가져와 풀에서 실행할 수 있는 여유가 있는 스레드에 전달하도록 하겠습니다.

`ThreadPool`의 `execute` 메서드를 콜로저를 매개변수로 받도록 정의합니다. 제13장의 [\u201c콜로저를 움직이고 `Fn` 트레이트\u201d][fn-traits]<!-- ignore --> 섹션에서 기억하시면 콜로저를 매개변수로 사용할 때 `Fn`, `FnMut`, `FnOnce` 세 가지 다른 트레이트를 사용할 수 있습니다. 여기에서 어떤 종류의 콜로저를 사용해야 하는지 결정해야 합니다. 표준 라이브러리 `thread::spawn` 구현과 유사한 작업을 수행할 것이라는 것을 알고 있으므로 `thread::spawn`의 매개변수에 대한 서명의 경계를 살펴볼 수 있습니다. 문서는 다음과 같이 보여줍니다.

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

 `F` 유형 매개변수가 우리가 관심 있는 부분입니다. `T` 유형 매개변수는 반환 값과 관련이 있으며, 우리는 그에 관심이 없습니다. `spawn`이 `FnOnce`를 `F`에 대한 트레이트 경계로 사용한다는 것을 알 수 있습니다. 이것이 우리가 원하는 것일 가능성이 높습니다. 왜냐하면 우리는 `execute`에서 받는 인수를 `spawn`에 전달할 것입니다. `FnOnce`가 우리가 원하는 트레이트라는 것을 더 확신할 수 있습니다. 왜냐하면 요청을 실행하는 스레드는 요청의 클로저를 한 번만 실행하기 때문입니다. 이는 `FnOnce`의 `Once`와 일치합니다.

`F` 유형 매개변수에는 `Send` 트레이트 경계와 `'static` 라이프타임 경계가 있습니다. 이는 우리 상황에 유용합니다. 다른 스레드로 클로저를 전송하려면 `Send`가 필요하며, 스레드가 실행을 완료하는 데 걸리는 시간을 모르기 때문에 `'static`가 필요합니다. `ThreadPool`에 `execute` 메서드를 만들어 `F` 유형의 일반 매개변수를 사용하여 이러한 경계를 받도록 하겠습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

`FnOnce` 뒤에 여전히 `()`를 사용하는 이유는 이 `FnOnce`가 매개변수가 없고 단위 유형 `()`를 반환하는 클로저를 나타내기 때문입니다. 함수 정의와 마찬가지로 반환 유형은 시그니처에서 생략할 수 있지만, 매개변수가 없더라도 괄호가 필요합니다.

다시 한번, `execute` 메서드의 가장 간단한 구현은 아무것도 하지 않습니다. 하지만 우리는 코드를 컴파일하도록 만드는 것에만 집중하고 있습니다. 다시 한번 확인해 보겠습니다.

```console
{{#include ../listings/ch20-web-server/no-listing-03-define-execute/output.txt}}
```

컴파일됩니다! 하지만 브라우저에서 `cargo run`을 시도하고 브라우저에서 요청을 보내면, 이 장의 처음에 본 오류 메시지를 브라우저에서 볼 것입니다. 우리 라이브러리가 `execute`에 전달된 클로저를 호출하지 않습니다!

> 참고: Haskell과 Rust와 같은 엄격한 컴파일러를 사용하는 언어에 대해 들을 수 있는 말은 \u201c컴파일이 되면 작동한다\u201d입니다. 하지만 이 말은 항상 사실은 아닙니다. 우리 프로젝트는 컴파일되지만 실제로 아무것도 하지 않습니다! 만약 실제 완전한 프로젝트를 구축하고 있다면, 코드가 컴파일되고 우리가 원하는 동작을 하는지 확인하기 위해 단위 테스트를 작성하는 것이 좋을 때입니다.

#### `new`에서 스레드 수를 검증

`new`와 `execute`의 매개변수를 아무것도 하지 않습니다. 이 함수의 몸체를 원하는 동작으로 구현하겠습니다. 먼저 `new`에 대해 생각해 보겠습니다. 이전에 `size` 매개변수에 부호가 없는 유형을 선택한 이유는 스레드 수가 음수인 풀이 의미가 없기 때문입니다. 그러나 스레드 수가 0인 풀도 의미가 없지만, 0은 완벽히 유효한 `usize`입니다. `size`가 0보다 크다는 것을 확인하기 전에 `ThreadPool` 인스턴스를 반환하고 프로그램이 0을 받으면 `panic`하도록 `assert!` 매크로를 추가합니다. Listing 20-13에서 보여주는 것처럼.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-13/src/lib.rs:here}}
```

<span class=\"caption\">Listing 20-13: `ThreadPool::new`를 구현하여 `size`가 0이면 `panic`</span>

`ThreadPool`에 대한 설명서도 추가했습니다. Chapter 14에서 논의된 것처럼, 함수가 `panic`할 수 있는 상황을 명시하는 섹션을 추가하여 좋은 설명서 작성 규칙을 따랐습니다. `cargo doc --open`을 실행하고 `ThreadPool` 구조체를 클릭하여 `new`에 대한 생성된 설명서를 확인해 보세요!

우리가 여기서 `assert!` 매크로를 추가한 대신, Listing 12-9의 I/O 프로젝트에서 `Config::build`와 같이 `new`를 `build`로 변경하고 `Result`를 반환할 수도 있습니다. 하지만 이 경우, 스레드가 없는 스레드 풀을 만드는 것은 되돌릴 수 없는 오류라고 결정했습니다. 만약 실제 완전한 프로젝트를 구축하고 있다면, 이러한 오류를 처리하는 방법을 고려해야 합니다.열정적인 분이시라면, 다음과 같은
`build` 함수를 작성해보세요.

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### 스레드를 저장할 공간을 만드는 방법

이제 스레드 풀에 저장할 유효한 스레드 수를 알고 있으므로, 스레드를 생성하고
`ThreadPool` 구조체에 저장하기 전에 스레드를 저장하는 방법을 살펴보겠습니다.
스레드를 어떻게 \u201c저장\u201d 하는지 알아보겠습니다. `thread::spawn` 함수의
구조를 다시 살펴보겠습니다.

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`spawn` 함수는 `JoinHandle<T>`를 반환하며, `T`는 클로저가 반환하는 유형입니다.
우리가 스레드 풀에 전달하는 클로저는 연결을 처리하고 아무것도 반환하지 않으므로,
`T`는 단위 유형 `()`입니다.

Listing 20-14의 코드는 컴파일되지만 아직 스레드를 생성하지 않습니다.
`ThreadPool`의 정의를 변경하여 `thread::JoinHandle<()>` 인스턴스의 벡터를
저장하고, 이 벡터를 반환합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-14/src/lib.rs:here}}
```

<span class=\"caption\">Listing 20-14: `ThreadPool`에 스레드를 저장하기 위한
벡터 생성</span>

`std::thread`를 라이브러리 크레인에 범위 내에 가져왔습니다. `ThreadPool`에서
`thread::JoinHandle` 유형의 요소를 저장하기 때문입니다.

유효한 크기가 받으면 `ThreadPool`은 `size` 개의 요소를 저장할 수 있는 새로운
벡터를 생성합니다. `with_capacity` 함수는 `Vec::new`와 동일한 작업을 수행하지만,
중요한 차이점이 있습니다. 즉, 벡터에 미리 공간을 할당합니다. `size` 개의
요소를 벡터에 저장해야 한다는 것을 알고 있기 때문에, 이러한 할당을 미리
수행하는 것은 `Vec::new`를 사용하는 것보다 약간 더 효율적입니다. `Vec::new`는
요소가 삽입될 때마다 크기를 조정합니다.

`cargo check`를 다시 실행하면 성공해야 합니다.

#### `ThreadPool`과 스레드 간의 코드 전달을 담당하는 `Worker` 구조체

Listing 20-14의 `for` 루프에 대한 주석을 살펴보았습니다. 이제 스레드를
실제로 생성하는 방법을 살펴보겠습니다. 표준 라이브러리는 `thread::spawn`을
사용하여 스레드를 생성하는 방법을 제공하며, `thread::spawn`은 스레드가
생성되자마자 실행해야 하는 코드를 가져옵니다. 그러나 우리의 경우, 스레드를
생성하고 나중에 전달할 코드를 *기다려야 합니다*. 표준 라이브러리의 스레드
구현에는 이러한 방법이 포함되어 있지 않으므로 수동으로 구현해야 합니다.

이러한 동작을 구현하기 위해 `ThreadPool`와 스레드 사이에 새로운 데이터
구조를 도입할 것입니다. 이 데이터 구조를 *Worker*라고 부르며, 풀 구현에서
흔히 사용되는 용어입니다. `Worker`는 실행해야 할 코드를 가져와 이미 실행 중인
스레드에서 실행합니다. 요리사가 레스토랑의 주방에서 일하는 사람들을 생각해보세요.
`Worker`는 주문이 들어올 때까지 기다리고, 그런 다음 주문을 받아서 처리합니다.

`ThreadPool`에서 `JoinHandle<()>` 인스턴스의 벡터를 저장하는 대신,
`Worker` 구조체의 인스턴스를 저장할 것입니다. 각 `Worker`는 하나의 `JoinHandle<()>` 인스턴스를 저장합니다. 그런 다음 `Worker`에 `JoinHandle<()>` 인스턴스를 가져와서 실행할 코드를 실행하는 메서드를 구현합니다. 각 `Worker`에 `id`를 부여하여 풀에서 다른 `Worker`를 구분할 수 있도록 합니다.

`ThreadPool`를 생성할 때 발생하는 새로운 프로세스는 다음과 같습니다.
`Worker`를 이렇게 설정한 후에 클로저를 `Worker`로 전달하는 코드를 구현할 것입니다.

1. `Worker` 구조체를 정의하여 `id`와 `JoinHandle<()>`를 포함합니다.
2. `ThreadPool`을 `Worker` 인스턴스의 벡터를 포함하도록 변경합니다.
3. `Worker::new` 함수를 정의하여 `id` 숫자를 받아 `id`를 포함하고 빈 
   closure를 사용하여 스레드가 생성된 `Worker` 인스턴스를 반환합니다.
4. `ThreadPool::new`에서 `for` 루프 카운터를 사용하여 `id`를 생성하고 해당 `id`를 사용하여 새로운 `Worker`를 생성하고 `workers` 벡터에 저장합니다.

만약 도전을 즐기신다면, 다음 코드를 살펴보기 전에 스스로 이러한 변경 사항을 구현해 보세요.

준비되셨나요? 다음은 Listing 20-15에서 하나의 방법으로 이러한 변경 사항을 적용한 코드입니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-15/src/lib.rs:here}}
```

<span class=\"caption\">Listing 20-15: `ThreadPool`을 `Worker` 인스턴스를 직접 저장하도록 변경</span>

`ThreadPool`의 필드 이름을 `threads`에서 `workers`로 변경했습니다. 이는 이제 `Worker` 인스턴스를 `JoinHandle<()>` 인스턴스 대신 저장하기 때문입니다. `for` 루프의 카운터를 `Worker::new` 함수의 인자로 사용하고, 각 새로운 `Worker`를 `workers`라는 벡터에 저장합니다.

외부 코드(예: `src/main.rs`의 서버)는 `ThreadPool` 내부에서 `Worker` 구조체를 사용하는 구현 세부 사항을 알 필요가 없으므로, `Worker` 구조체와 `new` 함수를 private로 만듭니다. `Worker::new` 함수는 제공된 `id`를 사용하고 빈 closure를 사용하여 새 스레드를 생성하는 `JoinHandle<()>` 인스턴스를 저장합니다.

> 주의: 운영 체제에서 스레드를 생성할 수 없다면(시스템 리소스가 충분하지 않은 경우) `thread::spawn`은 panic합니다. 이로 인해 서버 전체가 panic하게 되며, 일부 스레드 생성이 성공했더라도 그렇습니다. 단순성을 위해 이러한 동작은 괜찮지만, 실제 프로덕션 스레드 풀 구현에서는 [`std::thread::Builder`][builder]<!-- ignore --> 및 그 `spawn`<!-- ignore --> 메서드가 `Result`를 반환하는 것을 사용하는 것이 좋습니다.

이 코드는 컴파일되고 `ThreadPool::new`에 지정된 `Worker` 인스턴스 수를 저장합니다. 그러나 `execute` 메서드에서 받는 closure를 처리하지는 않습니다. 다음 단계로 넘어가서 이를 수행하는 방법을 살펴보겠습니다.

#### 채널을 사용하여 스레드에 요청 전달하기

다음으로 해결해야 할 문제는 `thread::spawn`에 전달되는 closure가 아무것도 하지 않는 것입니다. 현재 `execute` 메서드에서 실행하려는 closure를 받습니다. 하지만 각 `Worker`를 생성할 때 `thread::spawn`에 실행할 closure를 제공해야 합니다.

`ThreadPool` 내부에서 `Worker` 인스턴스가 실행할 코드를 가져와 `ThreadPool`의 큐에 저장하는 채널을 사용하면 좋습니다. `execute` 메서드는 `ThreadPool`에서 실행하려는 작업을 채널을 통해 전송합니다. `Worker` 인스턴스는 스레드에서 루프를 돌면서 받은 작업을 실행합니다.

다음은 계획입니다.

1. `ThreadPool`는 채널을 생성하고 보내는 쪽을 보유합니다.
2. 각 `Worker`는 받는 쪽을 보유합니다.
3. `Job` 구조체를 새로 만들어 실행하려는 closure를 저장합니다.
4. `execute` 메서드는 실행하려는 작업을 채널을 통해 보냅니다.
5. `Worker`의 스레드에서 루프를 돌면서 받은 작업의 closure를 실행합니다.

Listing 20-16에서 `ThreadPool::new`에서 채널을 생성하고 보내는 쪽을 `ThreadPool` 인스턴스에 저장하는 방법을 살펴보겠습니다. `Job` 구조체는 현재 아무것도 포함하지 않지만, 채널을 통해 전송할 항목의 유형이 될 것입니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
## 20.2. 다중 스레드

{{#rustdoc_include ../listings/ch20-web-server/listing-20-16/src/lib.rs:here}}
```

<span class=\"caption\">Listing 20-16: `ThreadPool`을 수정하여 `Job` 인스턴스를 전송하는 채널의 발신자를 저장</span>

`ThreadPool::new`에서 새로운 채널을 생성하고 풀이 채널의 발신자를 저장합니다. 이렇게 하면 코드가 성공적으로 컴파일됩니다.

채널의 수신자를 각 작업자로 전달하여 작업자 스레드가 생성될 때 채널의 수신자를 사용할 수 있도록 하려고 합니다. `receiver` 매개변수를 closure 내에서 참조해야 하므로, Listing 20-17의 코드는 아직 컴파일되지 않습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/listing-20-17/src/lib.rs:here}}
```

<span class=\"caption\">Listing 20-17: 작업자에게 수신자를 전달</span>

작은 변화를 몇 가지 적용했습니다. `Worker::new`에 수신자를 전달하고, closure 내에서 사용합니다.

이 코드를 확인해 보면 다음과 같은 오류가 발생합니다.

```console
{{#include ../listings/ch20-web-server/listing-20-17/output.txt}}
```

코드는 `receiver`를 여러 `Worker` 인스턴스로 전달하려고 합니다. 이는 Rust에서 제공하는 채널 구현 방식과 일치하지 않습니다. Rust의 채널 구현 방식은 여러 *생산자*, 하나의 *소비자*입니다. 따라서 이 코드를 수정하려면 채널의 소비 쪽을 복사할 수 없습니다. 또한 여러 소비자에게 메시지를 여러 번 전송하는 것은 원하지 않습니다. 각 메시지가 한 번씩 처리되도록 하나의 메시지 목록과 여러 작업자를 사용해야 합니다.

또한 채널 큐에서 작업을 가져오는 것은 `receiver`를 변경하는 작업이므로, 스레드가 `receiver`를 안전하게 공유하고 변경할 수 있는 방법이 필요합니다. 그렇지 않으면 레이스 조건(Chapter 16에서 다루었습니다)이 발생할 수 있습니다.

Chapter 16에서 다룬 스레드 안전한 스마트 포인터를 기억하세요. 여러 스레드에서 소유권을 공유하고 스레드가 값을 변경할 수 있도록 하려면 `Arc<Mutex<T>>`를 사용해야 합니다. `Arc`는 여러 작업자가 `receiver`의 소유권을 가질 수 있도록 하고, `Mutex`는 한 번에 한 작업자만 `receiver`에서 작업을 가져올 수 있도록 합니다. Listing 20-18은 이러한 변경 사항을 보여줍니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-18/src/lib.rs:here}}
```

<span class=\"caption\">Listing 20-18: `Arc`와 `Mutex`를 사용하여 작업자 간에 `receiver`를 공유</span>

`ThreadPool::new`에서 `receiver`를 `Arc`와 `Mutex`에 넣습니다. 각 새로운 작업자에게는 `Arc`를 복사하여 작업자가 `receiver`의 소유권을 공유할 수 있도록 참조 카운트를 증가시킵니다.

이러한 변경 사항을 적용하면 코드가 컴파일됩니다! 점점 가까워지고 있습니다.

#### `execute` 메서드 구현

마지막으로 `ThreadPool`의 `execute` 메서드를 구현해야 합니다. `Job`를 구현 방식을 변경하여 `execute`가 받는 closure의 유형을 갖는 트레이트 객체로 변경합니다. Chapter 19의 [\u201cCreating Type Synonyms with Type Aliases\u201d][creating-type-synonyms-with-type-aliases]<!-- ignore -->
섹션에서 설명했듯이, 타입 별칭은 긴 타입을 간결하게 만들어 사용하기 쉽게 합니다. Listing 20-19를 참조하세요.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-19/src/lib.rs:here}}
```

<span class=\"caption\">Listing 20-19: `Job` 타입 별칭을 `Box`를 사용하여 각 closure를 갖는 타입으로 만들고 채널을 통해 작업을 전송</span>

`execute`에서 받는 closure를 사용하여 새로운 `Job` 인스턴스를 생성한 후, 채널의 발신 쪽으로 작업을 전송합니다. `send`에서 `unwrap`을 호출하여 전송 실패 시 발생하는 경우를 처리합니다. 이는 예를 들어 채널이 이미 닫혔을 때 발생할 수 있습니다.


스레드가 실행되지 않도록 막아서, 수신 측이 더 이상 메시지를 수신하지 않는다는 것을 의미합니다. 현재 우리는 스레드를 실행되지 않도록 막을 수 없습니다. 스레드는 풀이 존재하는 동안 계속 실행됩니다. `unwrap`을 사용하는 이유는 실패 사례가 발생하지 않을 것이라는 것을 알고 있기 때문입니다. 하지만 컴파일러는 그렇지 않습니다. 

하지만 아직 끝나지 않았습니다! 작업자에서 `thread::spawn`에 전달되는 폐쇄는 수신 측의 채널을 *참조*할 뿐입니다. 대신, 폐쇄가 영원히 루프를 돌려 채널의 수신 측에 작업을 요청하고 작업을 받으면 실행해야 합니다. 20-20번 목록에 나와 있는 변경 사항을 `Worker::new`에 적용해 보겠습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-20/src/lib.rs:here}}
```

<span class=\"caption\">Listing 20-20: 작업자의 스레드에서 작업을 수신하고 실행하기</span>

여기서 먼저 `lock`을 사용하여 `receiver`에 잠금을 획득하고, 그런 다음 `unwrap`을 사용하여 오류 발생 시 에러를 발생시킵니다. 잠금을 획득하는 것은 다른 스레드가 잠금을 획득한 상태에서 에러가 발생하여 잠금을 해제하지 않은 경우, 즉 *오염된* 상태인 경우 실패할 수 있습니다. 이러한 상황에서는 `unwrap`을 사용하여 현재 스레드가 에러를 발생시키는 것이 올바른 조치입니다. `expect`로 변경하여 의미 있는 오류 메시지를 포함할 수 있습니다.

만약 잠금을 획득하면 `recv`를 호출하여 채널에서 `Job`을 수신합니다. 마지막 `unwrap`은 오류를 무시하여 진행합니다. 이 오류는 보내는 스레드를 갖는 스레드가 종료된 경우 발생할 수 있습니다. `send` 메서드가 수신자가 종료된 경우 `Err`를 반환하는 것과 유사합니다.

`recv` 호출은 차단되므로 작업이 아직 없으면 현재 스레드는 작업이 사용 가능해질 때까지 기다립니다. `Mutex<T>`는 한 번에 한 개의 `Worker` 스레드만 작업을 요청하려고 시도하도록 보장합니다.

이제 스레드 풀이 작동 상태입니다! `cargo run`을 실행하고 몇 가지 요청을 보내세요:

<!-- manual-regeneration
cd listings/ch20-web-server/listing-20-20
cargo run
127.0.0.1:7878에 몇 가지 요청을 보내세요
요청을 보내는 것은 자동화할 수 없습니다. 출력은 요청을 보내는 것에 따라 달라지기 때문입니다
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field is never read: `workers`
 --> src/lib.rs:7:5
  |
7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: field is never read: `id`
  --> src/lib.rs:48:5
   |
48 |     id: usize,
   |     ^^^^^^^^^

warning: field is never read: `thread`
  --> src/lib.rs:49:5
   |
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: `hello` (lib) generated 3 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 1.40s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

성공! 이제 작업을 비동기적으로 실행하는 스레드 풀을 가지고 있습니다. 최대 4개의 스레드만 생성되므로 서버에 많은 요청이 들어오더라도 시스템이 과부하되지 않습니다. */sleep*에 요청을하면 서버는 다른 스레드가 다른 요청을 실행하여 다른 요청을 처리할 수 있습니다.

> 주의: */sleep*를 여러 브라우저 창에서 동시에 열면 5초 간격으로 한 번씩 로드될 수 있습니다. 일부 웹 브라우저는 동시에 실행됩니다.다음은 캐싱 목적으로 동일한 요청의 여러 인스턴스를 순차적으로 처리하는 데 사용되는 웹 서버의 한계입니다. 이 제한은 우리의 웹 서버에서 발생하지 않습니다.

제18장에서 `while let` 루프를 배웠다면, 왜 `Worker::new` 함수 코드를 표시된 것처럼 작성하지 않았는지 궁금할 수 있습니다. 20-21번 목록.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-21/src/lib.rs:here}}
```

<span class=\"caption\">Listing 20-21: `while let`을 사용하여 `Worker::new`의 대안 구현</span>

이 코드는 컴파일되고 실행되지만, 원하는 스레드 동작을 생성하지 않습니다. 느린 요청이 다른 요청이 처리되기 전까지 기다리게 됩니다. 이유는 다음과 같습니다. `Mutex` 구조체에는 `unlock` 메서드가 없기 때문입니다. `MutexGuard<T>` 내부의 잠금 소유권이 `Mutex` 구조체의 라이프타임에 기반하기 때문입니다. 컴파일 시점에, 보로 체커는 `Mutex`로 보호된 리소스는 잠금을 갖고 있지 않으면 액세스할 수 없다는 규칙을 강제할 수 있습니다. 그러나 이 구현은 `MutexGuard<T>`의 라이프타임에 주의하지 않으면 잠금이 예상보다 오래 유지될 수 있습니다.

20-20번 목록에서 `let job =
receiver.lock().unwrap().recv().unwrap();`를 사용하는 코드는 `let`을 사용하기 때문에, 등호 기호의 오른쪽 식의 임시 값은 `let` 문이 끝날 때 즉시 삭제됩니다. 그러나 `while let` (및 `if let` 및 `match`)은 관련 블록의 끝까지 임시 값을 삭제하지 않습니다. 20-21번 목록에서 잠금은 `job()` 호출의 지속 동안 유지되므로 다른 작업자가 작업을 수신할 수 없습니다.

[creating-type-synonyms-with-type-aliases]:
ch19-04-advanced-types.html#creating-type-synonyms-with-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[fn-traits]:
ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
