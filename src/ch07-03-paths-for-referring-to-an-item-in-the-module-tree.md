## 모듈 트리에서 항목을 참조하는 경로

Rust 에 항목이 어디에 있는지 보여주려면 파일 시스템을 탐색할 때 사용하는 경로와 같은 방식으로 모듈 트리에서의 경로를 사용합니다. 함수를 호출하려면 그 함수의 경로를 알아야 합니다.

경로는 두 가지 형태를 가질 수 있습니다.

* 절대 경로는 crate 루트에서 시작하는 전체 경로입니다. 외부 crate 에서의 코드의 경우 절대 경로는 crate 이름으로 시작하며, 현재 crate 에서의 코드의 경우 `crate` 라는 문자열로 시작합니다.
* 상대 경로는 현재 모듈에서 시작하여 `self`, `super` 또는 현재 모듈의 식별자를 사용합니다.

절대 경로와 상대 경로 모두는 두 개의 콜론(`::`)으로 구분된 하나 이상의 식별자로 이루어져 있습니다.

7-1번 목록을 다시 살펴보겠습니다. 예를 들어 `add_to_waitlist` 함수를 호출하고 싶다고 가정해 보겠습니다. 이것은 `add_to_waitlist` 함수의 경로가 무엇인지 묻는 것과 같습니다.
7-3번 목록은 일부 모듈과 함수가 제거된 7-1번 목록을 포함합니다.

`eat_at_restaurant` 함수에서 `add_to_waitlist` 함수를 호출하는 두 가지 방법을 보여드리겠습니다. 이 경로는 올바르지만, 이 예제가 컴파일되지 않게 하는 다른 문제가 있습니다. 잠시 후에 설명하겠습니다.

`eat_at_restaurant` 함수는 우리의 라이브러리 crate의 공개 API의 일부이므로 `pub` 키워드로 표시합니다. [\u201c`pub` 키워드로 경로를 노출하는 방법\u201d][pub]<!-- ignore --> 섹션에서 `pub`에 대해 자세히 알아보겠습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

<span class=\"caption\">Listing 7-3: `add_to_waitlist` 함수를 절대 경로와 상대 경로를 사용하여 호출하는 방법</span>

`eat_at_restaurant` 함수에서 `add_to_waitlist` 함수를 처음 호출할 때 절대 경로를 사용합니다. `add_to_waitlist` 함수는 `eat_at_restaurant` 함수와 같은 crate에 정의되어 있으므로 `crate` 키워드를 사용하여 절대 경로를 시작할 수 있습니다. 그런 다음 `add_to_waitlist`에 도달할 때까지 차례로 모듈을 포함합니다. 파일 시스템의 구조를 상상해 보세요. 동일한 구조를 가진 파일 시스템에서 `add_to_waitlist` 프로그램을 실행하려면 `/front_of_house/hosting/add_to_waitlist`와 같은 경로를 지정해야 합니다. crate 이름을 사용하여 crate 루트에서 시작하는 것은 쉘에서 `/`를 사용하여 파일 시스템 루트에서 시작하는 것과 같습니다.

`add_to_waitlist` 함수를 `eat_at_restaurant` 함수에서 두 번째로 호출할 때는 상대 경로를 사용합니다. 경로는 `front_of_house`로 시작합니다. 이는 `eat_at_restaurant` 함수와 같은 모듈 트리 레벨에 정의된 모듈의 이름입니다. 여기서는 파일 시스템의 동등물은 `front_of_house/hosting/add_to_waitlist` 경로입니다. 모듈 이름으로 시작하는 것은 경로가 상대적임을 의미합니다.

절대 경로 또는 상대 경로를 사용할지 여부는 프로젝트에 따라 결정하며, 항목 정의 코드를 사용하는 코드와 별도로 움직일 가능성이 더 높은지에 따라 결정됩니다. 예를 들어 `front_of_house` 모듈과 `eat_at_restaurant` 함수를 `customer_experience`이라는 모듈로 이동하면 `add_to_waitlist` 함수를 호출하는 절대 경로를 업데이트해야 하지만, 상대 경로는 여전히 유효합니다. 그러나 `eat_at_restaurant` 함수를 `dining`이라는 모듈로 별도로 이동하면 `add_to_waitlist` 함수 호출의 절대 경로는 동일하지만 상대 경로를 업데이트해야 합니다. 일반적으로 우리는 절대 경로를 지정하는 것을 선호합니다. 왜냐하면 코드 정의와 항목 호출을 독립적으로 움직일 가능성이 더 높기 때문입니다.

7-3번 목록을 컴파일해 보도록 하겠습니다. 왜 컴파일되지 않는지 확인해 보겠습니다! 오류 메시지는 7-4번 목록에 나와 있습니다.

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

<span class=\"caption\">표 7-4: Listing 7-3에서 작성된 코드를 빌드할 때 발생하는 컴파일러 오류</span>

오류 메시지는 모듈 `hosting`이 프라이빗이라고 말합니다. 즉, 우리는 `hosting` 모듈과 `add_to_waitlist` 함수에 대한 올바른 경로를 가지고 있지만, Rust는 접근 권한이 없기 때문에 사용할 수 없습니다. Rust에서 모든 항목(함수, 메서드, 구조체, 열거형, 모듈 및 상수)은 기본적으로 부모 모듈에 대해 프라이빗합니다. 특정 항목(함수 또는 구조체)을 프라이빗하게 만들고 싶다면 모듈에 넣습니다.

부모 모듈의 항목은 자식 모듈 내부의 프라이빗 항목을 사용할 수 없습니다. 그러나 자식 모듈의 항목은 조상 모듈의 항목을 사용할 수 있습니다. 이는 자식 모듈이 구현 세부 사항을 감싸고 숨기기 때문입니다. 하지만 자식 모듈은 정의된 맥락을 볼 수 있습니다. 우리의 은유를 생각해 보면, 프라이버시 규칙은 레스토랑의 백 오피스와 같다고 생각할 수 있습니다. 거기에서 일어나는 일은 레스토랑 고객에게는 프라이빗하지만, 사무 관리자는 운영하는 레스토랑의 모든 것을 볼 수 있습니다.

Rust는 내부 구현 세부 사항을 숨기는 것이 기본이 되도록 모듈 시스템을 작동하도록 선택했습니다. 그래서 내부 코드의 어떤 부분을 변경하면 외부 코드가 깨지지 않는지 알 수 있습니다. 그러나 Rust는 `pub` 키워드를 사용하여 자식 모듈의 코드의 내부 부분을 외부 조상 모듈에 노출하는 옵션을 제공합니다.

### `pub` 키워드를 사용하여 경로 노출

Listing 7-4의 오류에서 `hosting` 모듈이 프라이빗이라고 알려줍니다. `eat_at_restaurant` 함수가 부모 모듈에서 `add_to_waitlist` 함수에 접근할 수 있도록 `hosting` 모듈에 `pub` 키워드를 사용하여 선언합니다. Listing 7-5와 같습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs}}
```

<span class=\"caption\">Listing 7-5: `eat_at_restaurant` 함수에서 사용할 수 있도록 `hosting` 모듈을 `pub`으로 선언</span>

그러나 Listing 7-5의 코드는 여전히 Listing 7-6과 같은 컴파일러 오류를 발생시킵니다.

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

<span class=\"caption\">Listing 7-6: Listing 7-5에서 작성된 코드를 빌드할 때 발생하는 컴파일러 오류</span>

무슨 일이 일어났을까요? `mod hosting` 앞에 `pub` 키워드를 추가하면 모듈이 공개됩니다. 이 변경 사항으로 인해 `front_of_house`에 접근할 수 있다면 `hosting`에 접근할 수 있습니다. 그러나 `hosting`의 *내용*은 여전히 프라이빗합니다. 모듈을 공개하는 것은 모듈 내부 코드에 대한 접근을 허용하지 않습니다. 모듈에 대한 `pub` 키워드는 그 모듈을 조상 모듈에서 참조할 수 있도록 허용하는 것일 뿐입니다.

모듈이 컨테이너이기 때문에 `pub` 키워드만 사용하면 별로 도움이 되지 않습니다. 모듈 내부의 하나 이상의 항목을 공개로 선언해야 합니다.

Listing 7-6의 오류는 `add_to_waitlist` 함수가 프라이빗하다는 것을 말합니다. 프라이버시 규칙은 모듈뿐만 아니라 구조체, 열거형, 함수 및 메서드에도 적용됩니다.

`add_to_waitlist` 함수를 `pub`으로 만들어 Listing 7-7과 같이 `pub` 키워드를 추가합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs}}
```

<span class=\"caption\">Listing 7-7: `add_to_waitlist` 함수와 `mod hosting`에 `pub` 키워드를 추가하여 `eat_at_restaurant` 함수에서 함수를 호출할 수 있도록 합니다</span>

이제 코드가 컴파일됩니다! `pub` 키워드를 추가하여 이러한 경로를 `eat_at_restaurant`에서 절대 경로와 상대 경로의 관점에서 프라이버시 규칙에 따라 사용할 수 있게 하는 이유를 살펴보겠습니다.

절대 경로에서는 크레이트의 루트인 `crate`로 시작합니다. `front_of_house` 모듈은 크레이트 루트에 정의되어 있습니다. `front_of_house`가 공개되지 않았더라도, `eat_at_restaurant` 함수는 `front_of_house`와 같은 모듈(즉, `eat_at_restaurant`와 `front_of_house`는 형제 관계)에 정의되어 있기 때문에 `eat_at_restaurant`에서 `front_of_house`를 참조할 수 있습니다. 다음은 `pub`로 표시된 `hosting` 모듈입니다. 부모 모듈인 `hosting`에 액세스할 수 있으므로 `hosting`에 액세스할 수 있습니다. 마지막으로, `add_to_waitlist` 함수는 `pub`로 표시되어 있으며 부모 모듈에 액세스할 수 있으므로 이 함수 호출이 작동합니다!

상대 경로에서는 절대 경로와 동일한 논리이지만 첫 번째 단계가 다릅니다. 크레이트 루트에서 시작하는 대신, 경로는 `front_of_house`에서 시작합니다. `front_of_house` 모듈은 `eat_at_restaurant`이 정의된 동일한 모듈 내에 정의되어 있기 때문에 `eat_at_restaurant`이 정의된 모듈에서 시작하는 상대 경로가 작동합니다. 그런 다음 `hosting`과 `add_to_waitlist`가 `pub`로 표시되어 있기 때문에 나머지 경로가 작동하고 이 함수 호출이 유효합니다!

다른 프로젝트에서 코드를 사용할 수 있도록 라이브러리 크레이트를 공유하려는 경우, 공개 API는 사용자와의 계약이며 코드와 상호 작용하는 방법을 결정합니다. 공개 API의 변경 관리에 대한 많은 고려 사항이 있으며, 크레이트에 의존하는 사람들에게 쉽게 변경을 적용할 수 있도록 합니다. 이러한 고려 사항은 이 책의 범위를 벗어나지만, 이 주제에 관심이 있다면 [Rust API 가이드라인](https://doc.rust-lang.org/api-guidelines/)을 참조하십시오.

> #### 바이너리와 라이브러리 모두 포함하는 패키지에 대한 최선의 방법

> 패키지에 *src/main.rs* 바이너리 크레이트 루트와 *src/lib.rs* 라이브러리 크레이트 루트가 모두 포함될 수 있으며, 두 크레이트 모두 기본적으로 패키지 이름을 가질 수 있습니다. 일반적으로 바이너리 크레이트와 라이브러리 크레이트 모두를 포함하는 패키지는 바이너리 크레이트에서 라이브러리 크레이트 내의 코드를 호출하는 실행 가능한 프로그램을 시작하는 데 필요한 코드만 포함합니다. 이렇게 하면 다른 프로젝트가 패키지가 제공하는 대부분의 기능을 활용할 수 있기 때문입니다. 라이브러리 크레이트의 코드는 공유될 수 있습니다.

> [Chapter 12](https://doc.rust-lang.org/book/ch12-00-command-line-programs.html)<!-- ignore -->에서 바이너리 크레이트와 라이브러리 크레이트를 모두 포함하는 명령줄 프로그램을 보여드리겠습니다.

### `super`를 사용하여 상대 경로 시작하기

`super`를 경로의 시작에 사용하여 현재 모듈 또는 크레이트 루트가 아닌 부모 모듈에서 시작하는 상대 경로를 구성할 수 있습니다. 이는 파일 시스템 경로에서 `..` 문법을 사용하는 것과 같습니다. `super`를 사용하면 부모 모듈에 있는 항목을 참조할 수 있으므로 모듈 트리가 밀접하게 관련되어 있지만 부모가 모듈 트리의 다른 위치로 이동될 수도 있는 경우 모듈 트리를 재정렬하는 것이 더 쉬워집니다.

`back_of_house` 모듈에서 잘못된 주문을 수정하고 직접 고객에게 가져다주는 요리사의 상황을 모델링하는 Listing 7-8의 코드를 고려하십시오. `back_of_house` 모듈에서 정의된 `fix_incorrect_order` 함수는 `super`를 사용하여 부모 모듈에서 정의된 `deliver_order` 함수를 호출합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

<span class=\"caption\">Listing 7-8: `super`를 사용하여 함수를 호출하는 상대 경로</span>

`fix_incorrect_order` 함수는 `back_of_house` 모듈에 있으므로 `super::deliver_order`를 사용하여 부모 모듈의 `deliver_order` 함수를 호출할 수 있습니다.


 `super`를 사용하여 `back_of_house`의 부모 모듈로 이동합니다. 이 경우 `crate` 즉 루트입니다. 그곳에서 `deliver_order`를 찾습니다. 성공! `back_of_house` 모듈과 `deliver_order` 함수가 서로의 관계를 유지하고 함께 이동할 가능성이 높다고 생각하므로, crate의 모듈 트리 재구성 시 함께 옮겨질 것입니다. 따라서 `super`를 사용하여 향후 코드 이동 시 업데이트해야 할 위치를 줄였습니다. 

### 구조체와 열거형을 공개하기

`pub`를 사용하여 구조체와 열거형을 공개로 지정할 수도 있지만, 구조체와 열거형에 `pub`를 사용하는 데는 몇 가지 추가 세부 사항이 있습니다. `pub`를 구조체 정의 앞에 사용하면 구조체를 공개로 하지만, 구조체의 필드는 여전히 private입니다. 각 필드를 필요에 따라 공개 또는 비공개로 설정할 수 있습니다. 7-9번 목록에서 `back_of_house::Breakfast` 구조체를 정의했습니다. 이 구조체는 `toast` 필드는 공개이지만 `seasonal_fruit` 필드는 private입니다. 이는 식당에서 고객이 식사와 함께 제공되는 빵 종류를 선택할 수 있지만, 요리사가 계절과 재고에 따라 식사와 함께 제공되는 과일을 결정하는 경우와 같습니다. 제공 가능한 과일은 빠르게 변경되므로 고객은 과일을 선택하거나 어떤 과일을 받을지 볼 수 없습니다. 

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

<span class=\"caption\">Listing 7-9: 일부 공개 필드와 일부 private 필드를 가진 구조체</span>

`back_of_house::Breakfast` 구조체의 `toast` 필드가 공개이기 때문에 `eat_at_restaurant`에서 `toast` 필드에 대한 읽기 및 쓰기를 점 notation을 사용하여 수행할 수 있습니다. `seasonal_fruit` 필드를 사용할 수 없다는 것을 알 수 있습니다. `seasonal_fruit` 필드 값을 수정하는 줄을 주석을 풀어보면 어떤 오류가 발생하는지 확인해보세요! 

또한, `back_of_house::Breakfast` 구조체가 private 필드를 가지고 있기 때문에, 구조체는 `Breakfast` 인스턴스를 생성하는 공개 연관 함수를 제공해야 합니다(여기서는 `summer`라고 명명했습니다). `Breakfast`가 이러한 함수를 가지고 있지 않았다면 `eat_at_restaurant`에서 `Breakfast` 인스턴스를 생성할 수 없었을 것입니다. `seasonal_fruit` 필드의 private 값을 `eat_at_restaurant`에서 설정할 수 없기 때문입니다. 

반면에 열거형을 공개로 지정하면 모든 변이가 공개가 됩니다. `enum` 키워드 앞에 `pub`만 사용하면 됩니다. 7-10번 목록을 참조하세요. 

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

<span class=\"caption\">Listing 7-10: 열거형을 공개로 지정하면 모든 변이가 공개됩니다</span>

`Appetizer` 열거형을 공개로 지정했기 때문에 `eat_at_restaurant`에서 `Soup`와 `Salad` 변이를 사용할 수 있습니다. 

열거형은 변이가 공개되어야만 유용합니다. 모든 열거형 변이에 대해 `pub`를 사용하여 지정해야 하는 것은 불편하므로, 열거형 변이의 기본값은 공개입니다. 구조체는 필드가 공개되지 않아도 유용할 수 있으므로 구조체 필드는 `pub`로 지정되지 않는 한 기본적으로 private입니다. 

`pub`와 관련하여 하나 더 다루지 않은 상황이 있습니다. 바로 마지막 모듈 시스템 기능인 `use` 키워드입니다. 먼저 `use`를 개별적으로 설명하고, 그런 다음 `pub`과 `use`를 함께 사용하는 방법을 보여드리겠습니다. 

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
