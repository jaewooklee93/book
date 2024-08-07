## 객체지향 언어의 특징

프로그래밍 커뮤니티에서 객체지향 언어로 간주되어야 할 특징에 대한 합의는 없습니다. Rust은 함수형 프로그래밍의 특징을 Chapter 13에서 살펴본 것처럼, 객체지향을 포함한 여러 프로그래밍 패러다임에 영향을 받았습니다. 어떤 특징을 공유하는지에 대한 주장은 객체, 캡슐화, 상속입니다. 각 특징의 의미와 Rust이 지원하는지 살펴보겠습니다.

### 객체는 데이터와 동작을 포함합니다

Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides가 저술한 *Design Patterns: Elements of Reusable Object-Oriented Software* 책(Addison-Wesley Professional, 1994)은 객체지향 설계 패턴의 카탈로그로, *The Gang of Four* 책으로도 알려져 있습니다. 이 책은 객체지향을 다음과 같이 정의합니다.

> 객체지향 프로그램은 객체로 구성됩니다. *객체*는 데이터와 그 데이터를 조작하는 절차를 모두 묶습니다. 절차는 일반적으로 *메서드* 또는 *연산*이라고 불립니다.

이 정의를 사용하면 Rust은 객체지향입니다. 구조체와 열거형은 데이터를 가지고 있으며, `impl` 블록은 구조체와 열거형에 메서드를 제공합니다. 구조체와 열거형이 메서드를 가지고 있더라도 *객체*로 불리지 않더라도, *The Gang of Four*의 객체 정의에 따라 동일한 기능을 제공합니다.

### 캡슐화는 구현 세부 정보를 숨깁니다

객체지향과 흔히 연결되는 또 다른 측면은 *캡슐화*입니다. 캡슐화는 객체의 구현 세부 정보가 해당 객체를 사용하는 코드에 액세스할 수 없다는 것을 의미합니다. 따라서 객체와 상호 작용하는 유일한 방법은 공개 API를 통해서이며, 사용하는 코드는 객체 내부에 들어가서 데이터나 동작을 직접 변경할 수 없습니다. 이를 통해 프로그래머는 객체의 내부를 변경하고 재작성할 수 있습니다. 객체를 사용하는 코드를 변경할 필요 없이.

Chapter 7에서 캡슐화를 제어하는 방법을 살펴보았습니다. `pub` 키워드를 사용하여 코드의 모듈, 유형, 함수, 메서드가 공개되어야 하는지 결정할 수 있습니다. 기본적으로 나머지는 프라이빗입니다. 예를 들어, `AveragedCollection`이라는 구조체를 정의할 수 있습니다. 이 구조체는 `i32` 값의 벡터를 포함하는 필드를 가지고 있습니다. 구조체에는 또한 벡터의 값의 평균을 포함하는 필드가 있을 수 있습니다. 즉, 누구나 평균을 필요로 할 때마다 계산할 필요가 없습니다. 즉, `AveragedCollection`은 평균을 캐시합니다. Listing 17-1은 `AveragedCollection` 구조체의 정의를 보여줍니다.

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-01/src/lib.rs}}
```

Listing 17-1: `AveragedCollection` 구조체의 정의
`AveragedCollection`은 `i32` 값의 리스트와 리스트의 평균을 유지합니다.

구조체는 다른 코드가 사용할 수 있도록 `pub`으로 표시되지만, 구조체 내부의 필드는 프라이빗입니다. 이는 `list`에 값이 추가되거나 제거될 때마다 평균도 업데이트되어야 하기 때문에 중요합니다. `add`, `remove`, `average` 메서드를 구조체에 구현하여 이를 처리합니다. Listing 17-2는 `AveragedCollection`에 대한 공개 메서드의 구현을 보여줍니다.

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-02/src/lib.rs:here}}
```

Listing 17-2: `AveragedCollection`에 대한 `add`, `remove`, `average` 메서드의 구현

`add`, `remove`, `average`와 같은 공개 메서드는 `AveragedCollection` 인스턴스의 데이터에 액세스하거나 수정하는 유일한 방법입니다. `add` 메서드를 사용하여 `list`에 값이 추가되거나 `remove` 메서드를 사용하여 값이 제거될 때, 각 메서드의 구현은 `list` 필드가 변경될 때마다 `average` 필드를 업데이트하는 `update_average` 메서드를 호출합니다.

`list`와 `average` 필드를 프라이빗으로 설정하여 외부 코드가 `list` 필드에 직접 값을 추가하거나 제거할 수 없도록 합니다. 그렇지 않으면 `list`가 변경될 때 `average` 필드가 동기화되지 않을 수 있습니다. `average` 메서드는 `average` 필드의 값을 반환하여 외부 코드가 `average`를 읽을 수 있도록 하지만 수정할 수는 없습니다.

 `AveragedCollection` 구조체의 구현 세부 사항을 캡슐화했기 때문에, 데이터 구조와 같은 측면을 향후 쉽게 변경할 수 있습니다. 예를 들어, `list` 필드에 `HashSet<i32>`를 사용할 수 있습니다. `add`, `remove`, `average` 공개 메서드의 서명이 동일하도록 유지하면 `AveragedCollection`을 사용하는 코드는 컴파일되지 않도록 변경할 필요가 없습니다. `list`를 공개로 만든다면, 이 경우 반드시 그렇지 않을 수 있습니다. `HashSet<i32>`와 `Vec<i32>`는 항목 추가 및 제거에 대한 다른 메서드를 가지므로, `list`를 직접 수정하는 경우 외부 코드가 변경되어야 할 가능성이 높습니다.

만약 캡슐화가 객체 지향 언어로 간주될 수 있는 필수적인 요소라면, Rust는 그 요구 사항을 충족합니다. 코드의 다른 부분에 대해 `pub`를 사용하거나 사용하지 않는 옵션은 구현 세부 사항의 캡슐화를 가능하게 합니다.

### 상속: 유형 시스템과 코드 공유로서의 상속

*상속*은 하나의 객체가 다른 객체의 정의에서 요소를 상속받아 부모 객체의 데이터와 동작을 얻을 수 있는 메커니즘입니다.

언어가 객체 지향 언어로 간주되려면 상속이 필수적이어야 한다면, Rust는 그렇지 않습니다. 부모 구조체의 필드와 메서드 구현을 상속받는 구조체를 정의하는 방법은 매크로를 사용하지 않고 없습니다.

그러나 상속을 프로그래밍 도구 상자에 익숙하다면, Rust에서 상속을 위해 사용하는 다른 솔루션을 사용할 수 있습니다. 상속을 사용하는 이유에 따라 다릅니다.

상속을 선택하는 이유는 두 가지입니다. 하나는 코드 재사용입니다. 특정 동작을 하나의 유형에 대해 구현할 수 있으며, 상속을 통해 다른 유형에서 해당 구현을 재사용할 수 있습니다. Listing 10-14에서 `summarize` 메서드에 대한 기본 구현을 추가했을 때 본 것처럼 Rust 코드에서 제한된 방법으로 이를 수행할 수 있습니다. `Summary` 추상형을 구현하는 모든 유형은 `summarize` 메서드를 사용할 수 있습니다. 이것은 부모 클래스가 메서드의 구현을 가지고 있고, 상속받는 자식 클래스에서도 메서드의 구현을 가지고 있는 것과 유사합니다. `Summary` 추상형을 구현할 때 `summarize` 메서드의 기본 구현을 재정의할 수도 있습니다. 이것은 부모 클래스에서 상속받은 메서드의 구현을 자식 클래스에서 재정의하는 것과 유사합니다.

상속의 다른 이유는 유형 시스템과 관련이 있습니다. 자식 유형이 부모 유형의 위치에서 사용될 수 있도록 합니다. 이것을 *다형성*이라고도 합니다. 즉, 특정 특징을 공유하는 여러 개체를 런타임에 서로 대체할 수 있습니다.

> ### 다형성

> 다형성은 많은 사람들에게 상속과 동의어입니다. 그러나 코드가 여러 유형의 데이터와 작동할 수 있도록 하는 더 일반적인 개념입니다. 상속의 경우, 이러한 유형은 일반적으로 하위 클래스입니다.

> Rust는 대신 제네릭을 사용하여 다양한 가능한 유형을 추상화하고 추상형 경계를 사용하여 이러한 유형이 제공해야 하는 제약을 부과합니다. 이를 *제한된 매개변수 다형성*이라고 합니다.

상속은 최근에 프로그래밍 설계 솔루션으로서 많은 프로그래밍 언어에서 인기를 잃었습니다. 그 이유는 필요 이상의 코드를 공유하는 위험에 처해 있기 때문입니다. 하위 클래스는 항상 부모 클래스의 모든 특징을 공유해서는 안 되지만, 상속을 사용하면 그렇게 됩니다. 이는 프로그램 설계의 유연성을 떨어뜨릴 수 있습니다. 또한, 일부 언어는 단일 상속(하위 클래스가 한 클래스에서만 상속받을 수 있다는 의미)만 허용하기 때문에 프로그램 설계의 유연성이 더욱 제한됩니다.

이러한 이유로 Rust는 상속 대신 추상형 객체를 사용하는 다른 접근 방식을 채택합니다. 추상형 객체가 Rust에서 다형성을 가능하게 하는 방법을 살펴보겠습니다