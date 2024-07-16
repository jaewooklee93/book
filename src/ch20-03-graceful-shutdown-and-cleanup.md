## 우아한 종료 및 정리

20-20번 목록의 코드는 스레드 풀을 사용하여 비동기적으로 요청에 응답하고 있습니다. 우리가 직접 사용하지 않는 `workers`, `id`, `thread` 필드에 대한 경고 메시지가 나타나서 정리 작업을 수행하지 않고 있다는 것을 상기시켜줍니다. `ctrl`-`c`를 사용하여 메인 스레드를 중지할 때, 다른 스레드는 요청을 처리하는 중이더라도 즉시 중지됩니다.

다음으로, 스레드 풀의 각 스레드에 대해 `join`을 호출하여 요청을 완료하기 전에 종료되도록 `Drop` 트레이트를 구현합니다. 또한 스레드가 새로운 요청을 수신하는 것을 중지하고 종료하도록 하는 방법을 구현합니다. 이 코드를 실행하려면 서버를 두 개의 요청만 처리한 후 스레드 풀을 우아하게 종료하도록 수정합니다.

### `ThreadPool` 에 `Drop` 트레이트 구현

스레드 풀이 삭제될 때 스레드가 모두 합쳐져 작업을 완료해야 합니다. 스레드 풀의 `Drop` 구현을 시작하는 것은 20-22번 목록에서 볼 수 있습니다. 이 코드는 아직 작동하지 않습니다.

Filename: src/lib.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/listing-20-22/src/lib.rs:here}}
```

Listing 20-22: 스레드 풀이 범위를 벗어날 때 각 스레드를 합치기

먼저 스레드 풀의 각 `workers`를 반복합니다. `&mut`를 사용하는 이유는 `self`가 가변 참조이기 때문이며, `worker`를 변경할 수 있어야 하기 때문입니다. 각 `worker`에 대해 작업 중인 스레드가 종료되도록 메시지를 출력하고 `join`을 호출합니다. `join` 호출이 실패하면 `unwrap`을 사용하여 Rust가 오류를 발생시키고 무예의식적인 종료로 넘어갑니다.

이 코드를 컴파일하면 다음과 같은 오류 메시지를 받습니다.

```console
{{#include ../listings/ch20-web-server/listing-20-22/output.txt}}
```

오류 메시지는 `join`을 호출할 수 없다는 것을 알려줍니다. 왜냐하면 `worker`에 대한 가변 참조만 가지고 있기 때문입니다. `join`은 인수를 소유해야 합니다. 이 문제를 해결하려면 `Worker` 인스턴스에서 `thread`를 옮겨 `join`이 스레드를 소비할 수 있도록 해야 합니다. 이 작업은 17-15번 목록에서 수행했습니다. `Worker`가 `Option<thread::JoinHandle<()>>`을 가지고 있다면, `Option`의 `take` 메서드를 사용하여 값을 `Some` 변형에서 옮겨 `None` 변형으로 남겨 `Worker`가 실행 중인 스레드가 없도록 할 수 있습니다. 즉, 실행 중인 `Worker`는 `Some` 변형을 가지고 있고, 정리할 때 `Some`를 `None`으로 바꿔 `Worker`가 실행할 스레드가 없도록 합니다.

따라서 `Worker` 정의를 다음과 같이 업데이트해야 합니다.

Filename: src/lib.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/no-listing-04-update-worker-definition/src/lib.rs:here}}
```

이제 컴파일러가 변경해야 하는 다른 부분을 찾도록 도와줍니다. 이 코드를 확인하면 두 가지 오류가 발생합니다.

```console
{{#include ../listings/ch20-web-server/no-listing-04-update-worker-definition/output.txt}}
```

두 번째 오류는 `Worker::new` 함수의 끝 부분을 가리키고 있으며, `thread` 값을 `Some`으로 감싸서 새 `Worker`를 생성해야 합니다. 이 오류를 수정하려면 다음과 같은 변경 사항을 적용합니다.

Filename: src/lib.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/no-listing-05-fix-worker-new/src/lib.rs:here}}
```

첫 번째 오류는 `Drop` 구현에서 발생합니다. 이전에 언급했듯이, 우리는 `Option` 값에 `take`를 호출하여 `thread`를 `worker`에서 옮기려고 했습니다. 다음 변경 사항이 이를 수행합니다.

Filename: src/lib.rs

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/no-listing-06-fix-threadpool-drop/src/lib.rs:here}}
```

17장에서 언급했듯이, `Option`의 `take` 메서드는 `Some` 변환을 가져와 `None`을 그 자리에 남깁니다. `if let`을 사용하여 `Some`을 해체하고 스레드를 가져오고, 그런 다음 스레드에 `join`을 호출합니다. 작업자의 스레드가 이미 `None`이라면, 해당 작업자는 이미 스레드를 정리했으므로 그런 경우에는 아무것도 일어나지 않습니다.

### 스레드에 작업을 중지하도록 신호 보내기

우리가 한 모든 변경 사항으로 코드가 경고 없이 컴파일됩니다. 그러나 이 코드는 아직 원하는 방식으로 작동하지 않습니다. 핵심은 `Worker` 인스턴스의 스레드가 실행하는 closure의 논리에 있습니다. 현재는 `join`을 호출하지만, 이는 스레드가 작업을 계속해서 찾아 `loop`하는 동안 스레드를 종료하지 않습니다. 현재 `ThreadPool`의 `drop` 구현을 사용하면 우리의 메인 스레드가 첫 번째 스레드가 완료될 때까지 영원히 차단됩니다.

이 문제를 해결하려면 `ThreadPool`의 `drop` 구현을 명시적으로 변경하고 `Worker` 루프를 변경해야 합니다.

먼저, `ThreadPool`의 `drop` 구현을 변경하여 스레드가 완료될 때까지 기다리기 전에 `sender`를 명시적으로 삭제합니다. 20-23번 목록은 `ThreadPool`에 `sender`를 명시적으로 삭제하는 변경 사항을 보여줍니다. `Option`과 `take` 기술을 `thread`과 동일하게 사용하여 `ThreadPool`에서 `sender`를 옮길 수 있습니다.

Filename: src/lib.rs

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-23/src/lib.rs:here}}
```

20-23번 목록: `sender`를 `join`하기 전에 명시적으로 삭제

`sender`를 삭제하면 채널이 닫히고, 더 이상 메시지가 전송되지 않음을 나타냅니다. 이 경우 작업자가 `loop`에서 하는 모든 `recv` 호출은 오류를 반환합니다. 20-24번 목록에서는 `Worker` 루프를 변경하여 이 경우에 루프에서 조심스럽게 나가도록 합니다. 즉, `ThreadPool`의 `drop` 구현이 `join`을 호출하면 스레드가 완료됩니다.

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-24/src/lib.rs:here}}
```

20-24번 목록: `recv`가 오류를 반환하면 루프에서 조심스럽게 나가기

이 코드를 실제로 실행하려면 `main`을 변경하여 서버를 두 개의 요청만 처리한 후 조심스럽게 종료하도록 합니다. 20-25번 목록에 `main`의 변경 사항이 나와 있습니다.

Filename: src/main.rs

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/listing-20-25/src/main.rs:here}}
```

20-25번 목록: 두 개의 요청을 처리한 후 루프를 종료하여 서버를 종료하기

실제 웹 서버가 두 개의 요청만 처리한 후 종료되는 것은 원하지 않습니다. 이 코드는 조심스러운 종료 및 정리 작업이 제대로 작동하는지 보여주는 데 사용됩니다.

`take` 메서드는 `Iterator` 트레이트에 정의되어 있으며, 최대 두 개의 항목까지 반복을 제한합니다. `ThreadPool`은 `main` 함수의 끝에서 범위를 벗어나고 `drop` 구현이 실행됩니다.

`cargo run`으로 서버를 시작하고 세 개의 요청을 보냅니다. 세 번째 요청은 오류가 발생해야 하며, 터미널에 다음과 같은 출력이 표시되어야 합니다.

<!-- manual-regeneration
"


cd listings/ch20-web-server/listing-20-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
세 번째 요청은 서버가 종료되었기 때문에 오류가 발생합니다
출력을 복사하세요
자동화할 수 없습니다. 출력은 요청을 보내는 데 달려 있습니다
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

출력 순서는 다를 수 있습니다. 이 코드가 어떻게 작동하는지 메시지에서 볼 수 있습니다. 작업자 0과 3은 첫 번째 두 개의 요청을 받았습니다. 서버는 두 번째 연결 후에 연결을 받는 것을 중단했으며, `ThreadPool`에서의 `Drop` 구현은 작업자 3이 작업을 시작하기 전에 실행되기 시작합니다. `sender`를 삭제하면 모든 작업자가 연결을 끊고 종료하도록 알립니다. 각 작업자는 연결이 끊어질 때 메시지를 출력하고, 스레드 풀은 각 작업자 스레드가 완료될 때까지 `join`을 호출합니다.

이 특정 실행의 하나의 흥미로운 측면은 `ThreadPool`가 `sender`를 삭제했고, 작업자가 오류를 받기 전에 작업자 0을 조인하려고 시도했던 것입니다. 작업자 0은 `recv`에서 오류를 받지 않았기 때문에 메인 스레드가 작업자 0가 완료될 때까지 차단되었습니다. 한편으로 작업자 3은 작업을 받았고 모든 스레드가 오류를 받았습니다. 작업자 0이 완료되면 메인 스레드가 나머지 작업자들이 완료될 때까지 기다렸습니다. 그때까지 그들은 모두 루프를 종료하고 중지했습니다.

축하합니다! 이제 프로젝트를 완료했습니다. 작업자 풀을 사용하여 비동기적으로 응답하는 기본 웹 서버를 구축했습니다. 모든 스레드를 풀에서 정리하여 서버를 차분하게 종료할 수 있습니다.

참고로 전체 코드가 있습니다.

Filename: src/main.rs

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/no-listing-07-final-code/src/main.rs}}
```

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-07-final-code/src/lib.rs}}
```

이 프로젝트를 계속 개선하고 싶다면 다음과 같은 아이디어가 있습니다.

* `ThreadPool`과 그 공개 메서드에 대한 설명을 추가합니다.
* 라이브러리의 기능에 대한 테스트를 추가합니다.
* `unwrap` 호출을 더 견고한 오류 처리로 변경합니다.
* `ThreadPool`을 사용하여 웹 요청을 처리하는 것 외에 다른 작업을 수행합니다.
* [crates.io](https://crates.io/)에서 스레드 풀 crate를 찾아 대신 crate를 사용하여 유사한 웹 서버를 구현합니다. 그런 다음 구현한 스레드 풀의 API와 견고성을 비교합니다.

## 요약

잘했습니다! 책을 마무리했습니다! Rust를 배우는 데 함께 해주셔서 감사합니다. 이제 자신의 Rust 프로젝트를 구현하고 다른 사람들의 프로젝트에 도움을 줄 준비가 되었습니다. Rust 여정에서 어떤 어려움을 겪더라도 도움을 줄 수 있는 다른 Rustaceans의 친절한 커뮤니티가 있습니다.
