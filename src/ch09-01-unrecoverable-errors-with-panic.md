## `panic!`을 사용한 복구 불가능한 오류

때때로 코드에서 나쁜 일이 일어나고, 그에 대해 할 수 있는 일이 없습니다. 이러한 경우 Rust는 `panic!` 매크로를 제공합니다. 실제로 panic을 발생시키는 두 가지 방법은 코드가 panic하게 만드는 행동을 취하는 것(예: 배열의 끝을 넘어서 액세스하는 것) 또는 명시적으로 `panic!` 매크로를 호출하는 것입니다. 두 경우 모두 프로그램에서 panic을 발생시킵니다. 기본적으로 이러한 panic은 오류 메시지를 출력하고 스택을 되돌려서 정리한 후 종료합니다. 환경 변수를 통해 Rust가 panic 발생 시 호출 스택을 표시하도록 설정하여 panic의 원인을 추적하는 데 도움을 받을 수도 있습니다.

> ### Panic에 대한 스택 되돌리기 또는 중단
>
> 기본적으로 panic이 발생하면 프로그램이 *스택 되돌리기*를 시작합니다. 즉, Rust는 각 함수를 거슬러 올라가 데이터를 정리합니다. 그러나 되돌아가서 정리하는 것은 많은 작업입니다. 따라서 Rust는 프로그램을 즉시 *중단*하는 대안을 제공합니다. 이는 데이터를 정리하지 않고 프로그램을 종료하는 것입니다.
>
> 프로그램이 사용하던 메모리는 운영 체제에서 정리해야 합니다. 프로젝트에서 생성되는 바이너리 파일을 최대한 작게 만들어야 하는 경우, `panic` 발생 시 unwinding 대신 aborting을 사용할 수 있습니다. 이를 위해 `Cargo.toml` 파일의 적절한 `[profile]` 섹션에 `panic = 'abort'`를 추가하면 됩니다. 예를 들어, 릴리즈 모드에서 panic 시 중단하려면 다음과 같이 추가합니다.
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

간단한 프로그램에서 `panic!`를 호출해 보겠습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

프로그램을 실행하면 다음과 같은 내용이 표시됩니다.

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

`panic!` 호출은 마지막 두 줄에 포함된 오류 메시지를 유발합니다. 첫 번째 줄은 우리의 panic 메시지와 panic이 발생한 소스 코드의 위치를 보여줍니다. *src/main.rs:2:5*는 두 번째 줄, src/main.rs 파일의 다섯 번째 문자임을 나타냅니다.

이 경우, 지정된 줄은 우리의 코드에 있으며, 해당 줄로 이동하면 `panic!` 매크로 호출을 볼 수 있습니다. 다른 경우에는 `panic!` 호출이 우리의 코드가 호출하는 코드에서 발생할 수 있으며, 오류 메시지에서 보고되는 파일 이름과 줄 번호는 우리의 코드가 이끌어낸 `panic!` 호출이 아닌, `panic!` 매크로가 호출된 다른 사람의 코드에서 나타날 수 있습니다.

<!-- Old heading. Do not remove or links may break. -->
<a id=\"using-a-panic-backtrace\"></a>

`panic!` 백트레이스를 사용하여 `panic!` 호출이 어디에서 발생했는지 이해할 수 있습니다. `panic!` 호출이 라이브러리에서 오류로 인해 발생하는 경우, 우리의 코드가 문제를 일으키는지 이해하는 데 도움이 됩니다. 9-1번 목록은 우리의 코드가 유효한 인덱스 범위를 벗어나 있는지 확인하는 코드를 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

<span class=\"caption\">Listing 9-1: 유효한 인덱스 범위를 벗어나 있는 요소에 액세스하려는 시도, 이는 `panic!`를 발생시킵니다</span>

여기서는 벡터의 100번째 요소(인덱스 99)에 액세스하려고 하지만, 벡터에는 3개의 요소만 있습니다. 이 경우 Rust는 panic합니다.

C에서 데이터 구조의 끝을 넘어 읽으려고 시도하면 정의되지 않은
동작이 발생합니다. 데이터 구조에 해당하는 메모리 위치에 있는 내용을
받을 수도 있습니다. 이는 *버퍼 오버리드*라고 불리며, 공격자가 인덱스를
조작하여 허용되지 않는 데이터를 읽을 수 있도록 하여 보안 취약성을
유발할 수 있습니다.

프로그램이 이러한 취약성으로부터 보호되도록 하려면, Rust는 존재하지
않는 인덱스에서 요소를 읽으려고 시도하면 실행을 중단하고 계속되지
않도록 합니다. 아래 코드를 실행해 보세요:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

이 오류는 `v` 안에 있는 벡터의 인덱스 `99`를 액세스하려는 `main.rs` 파일의
4번째 줄을 가리킵니다.

`note:` 줄은 `RUST_BACKTRACE` 환경 변수를 설정하여 오류 발생 원인에 대한
백트레이스를 얻을 수 있다고 알려줍니다. 백트레이스는 오류 발생까지
호출된 모든 함수 목록입니다. Rust의 백트레이스는 다른 언어와 마찬가지로
작동합니다. 백트레이스를 읽는 핵심은 가장 위에서 시작하여 작성한 파일을
보고 있는 곳까지 읽는 것입니다. 그곳이 문제의 근원입니다. 그 위의 줄은
코드가 호출한 코드이며, 아래의 줄은 코드가 호출한 코드입니다. 이러한
전후 줄에는 핵심 Rust 코드, 표준 라이브러리 코드 또는 사용하는 crate가
있을 수 있습니다. `RUST_BACKTRACE` 환경 변수를 0이 아닌 값으로 설정하여
백트레이스를 얻어 보세요. 9-2번 목록은 유사한 출력을 보여줍니다.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/std/src/panicking.rs:645:5
   1: core::panicking::panic_fmt
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/panicking.rs:72:14
   2: core::panicking::panic_bounds_check
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/panicking.rs:208:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/slice/index.rs:255:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/alloc/src/vec/mod.rs:2770:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/ops/function.rs:250:5에서 발생했습니다.
note: 몇 가지 세부 정보는 생략되었습니다. 자세한 백트레이스를 확인하려면 `RUST_BACKTRACE=full`을 사용하십시오.
```

<span class=\"caption\">Listing 9-2: `panic!` 호출 시 생성되는 백트레이스
`RUST_BACKTRACE` 환경 변수가 설정되었을 때 표시</span>

꽤 많은 출력이 나오네요! 정확한 출력은 운영 체제와 Rust 버전에 따라 다를 수 있습니다. 이러한 정보를 포함한 백트레이스를 얻으려면 디버그 심볼이 활성화되어 있어야 합니다. 디버그 심볼은 `cargo build` 또는 `cargo run`을 사용하여 `--release` 플래그 없이 실행할 때 기본적으로 활성화됩니다. Listing 9-2의 출력에서 백트레이스의 6번째 줄은 우리 프로젝트에서 문제를 일으키는 줄인 *src/main.rs*의 4번째 줄을 가리킵니다. 우리 프로그램이 panic하지 않도록 하려면, 우리가 작성한 파일을 나타내는 첫 번째 줄에서 언급된 위치에서 조사를 시작해야 합니다. Listing 9-1에서 우리가 의도적으로 panic을 일으키는 코드를 작성한 경우, panic을 해결하는 방법은 벡터 인덱스 범위를 벗어난 요소를 요청하지 않는 것입니다. 코드가 앞으로 panic할 때, 코드가 어떤 값으로 어떤 작업을 수행하여 panic을 일으키는지 파악하고 코드가 대신 어떻게 해야 하는지 알아야 합니다.

`panic!`과 `panic!`을 사용하여 오류 조건을 처리해야 할 때, 그리고 언제 사용해야 하는지에 대해 `panic!`를 다시 살펴보겠습니다. [“To `panic!` or Not to
`panic!`”][to-panic-or-not-to-panic]<!-- ignore --> 섹션에서 이 장의 나중에.
다음으로 `Result`을 사용하여 오류에서 벗어나는 방법을 살펴보겠습니다.

[to-panic-or-not-to-panic]:
ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
