## 오류 처리

오류는 소프트웨어에서 피할 수 없는 현실입니다. 따라서 Rust은 문제가 발생했을 때 처리하는 데 다양한 기능을 제공합니다. 많은 경우 Rust은 코드가 컴파일되기 전에 오류 발생 가능성을 인지하고 적절한 조치를 취하도록 요구합니다. 이러한 요구 사항은 프로그램이 더욱 견고해지도록 하여 오류를 발견하고 생산 환경에 배포하기 전에 적절하게 처리하도록 합니다!

Rust는 오류를 *복구 가능한 오류*와 *복구 불가능한 오류* 두 가지 주요 범주로 나눕니다. *파일을 찾을 수 없음*과 같은 복구 가능한 오류의 경우, 일반적으로 문제를 사용자에게 보고하고 작업을 다시 시도하는 것이 좋습니다. 복구 불가능한 오류는 배열의 끝을 넘어서 접근하려는 것과 같은 버그의 증상이며, 프로그램 실행을 즉시 중지해야 합니다.

대부분의 언어는 이 두 가지 유형의 오류를 구분하지 않고 예외와 같은 메커니즘을 사용하여 동일하게 처리합니다. Rust은 예외를 사용하지 않습니다. 대신, 복구 가능한 오류에 대해 `Result<T, E>` 유형을 사용하고 프로그램이 복구 불가능한 오류를 발생시키면 실행을 중지하는 `panic!` 매크로를 사용합니다. 이 장에서는 먼저 `panic!` 을 호출하는 방법을 설명하고, `Result<T, E>` 값을 반환하는 방법에 대해 논의합니다. 또한 오류에서 복구하려는지, 아니면 실행을 중지해야 하는지 결정할 때 고려해야 할 사항을 살펴보겠습니다.
