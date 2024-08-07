## 자동화된 테스트 작성

1972년 에드거 W. 다이스트라가 쓴 에세이 “The Humble Programmer” 에서 그는
“프로그램 테스트는 버그의 존재를 보여주는 데 매우 효과적인 방법이지만,
그 부재를 보여주는 데는 완전히 부족합니다.” 라고 말했습니다. 그러나 이는 우리가
가능한 한 많이 테스트하지 않아야 한다는 의미는 아닙니다!

우리 프로그램의 정확성은 우리 코드가 우리가 의도하는 대로 작동하는 정도입니다. Rust는 프로그램의 정확성에 대한 높은 관심을 가지고 설계되었지만, 정확성은 복잡하고 증명하기 쉽지 않습니다. Rust의 타입 시스템은 이러한 부담의 큰 부분을 맡지만, 타입 시스템은 모든 것을 잡을 수 없습니다. 따라서 Rust는 자동화된 소프트웨어 테스트를 작성하는 데 대한 지원을 제공합니다.

예를 들어, `add_two`라는 함수를 작성하여 전달된 숫자에 2를 더하는 함수를 작성해 보겠습니다. 이 함수의 서명은 정수를 매개변수로 받고 정수를 결과로 반환합니다. 우리가 이 함수를 구현하고 컴파일할 때, Rust는 이제까지 배운 모든 타입 검사 및 보로 검사를 수행하여 예를 들어 `String` 값이나 이 함수에 대한 유효하지 않은 참조를 전달하지 않는지 확인합니다. 그러나 Rust는 이 함수가 정확히 우리가 의도하는 대로 작동할 것이라고 확인할 수 없습니다. 즉, 매개변수에 2를 더하는 것이 아니라, 예를 들어 매개변수에 10을 더하거나 매개변수에서 50을 빼는 것입니다! 이때 테스트가 들어옵니다.

`3`을 `add_two` 함수에 전달하면 반환 값이 `5`임을 확인하는 테스트를 작성할 수 있습니다. 우리가 코드를 변경할 때마다 이러한 테스트를 실행하여 기존의 올바른 동작이 변경되지 않았는지 확인할 수 있습니다.

테스트는 복잡한 기술입니다. 한 장에서 좋은 테스트를 작성하는 방법에 대한 모든 세부 사항을 다룰 수는 없지만, 이 장에서는 Rust의 테스트 기능의 메커니즘에 대해 논의할 것입니다. 테스트를 작성할 때 사용할 수 있는 어노테이션과 매크로, 테스트를 실행하는 데 제공되는 기본 동작 및 옵션, 그리고 테스트를 단위 테스트와 통합 테스트로 구성하는 방법에 대해 알아보겠습니다.
