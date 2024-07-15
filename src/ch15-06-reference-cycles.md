## 참조 사이클은 메모리 누수를 일으킬 수 있습니다.

Rust의 메모리 안전성 보장은 의도치 않게 메모리 해제가 되지 않는 메모리(메모리 누수라고 합니다)를 생성하는 것을 어렵게 하지만 불가능하지는 않습니다. 메모리 누수를 완전히 방지하는 것은 Rust의 보장이 아니므로 Rust에서 메모리 누수는 메모리 안전합니다. `Rc<T>`와 `RefCell<T>`를 사용하면 Rust가 메모리 누수를 허용한다는 것을 볼 수 있습니다. 참조가 서로 참조하는 사이클을 만들 수 있습니다. 이는 각 항목의 참조 카운트가 0에 도달하지 않기 때문에 항목의 메모리가 삭제되지 않기 때문에 메모리 누수를 일으킵니다.

### 참조 사이클 생성

Listing 15-25에서 정의된 `List` 열거형과 `tail` 메서드를 살펴보면서 참조 사이클이 어떻게 발생하는지와 어떻게 방지하는지 살펴보겠습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs}}
```

<span class=\"caption\">Listing 15-25: `List` 정의와 `tail` 메서드</span>

Listing 15-5에서의 `List` 정의의 또 다른 변형입니다. `Cons` 변형의 두 번째 요소는 `RefCell<Rc<List>>`가 되므로 Listing 15-24에서와 같이 `i32` 값을 수정할 수 있는 대신 `Cons` 변형이 가리키는 `List` 값을 수정하려는 것입니다. `Cons` 변형이 가리키는 두 번째 항목에 액세스하기 위해 `tail` 메서드를 추가했습니다.

Listing 15-26에서는 Listing 15-25에서 정의된 내용을 사용하는 `main` 함수를 추가합니다. 이 코드는 `a`에 있는 리스트와 `b`에 있는 리스트를 생성하고 `a`에 있는 리스트가 `b`에 있는 리스트를 가리키도록 합니다. 그런 다음 `a`에 있는 리스트를 `b`로 변경하여 참조 사이클을 만듭니다. 참조 카운트가 다양한 단계에서 어떻게 변하는지 보여주는 `println!` 문이 있습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

<span class=\"caption\">Listing 15-26: 서로 참조하는 두 `List` 값의 참조 사이클 생성</span>

`a` 변수에 `Rc<List>` 인스턴스를 생성하여 `List` 값을 저장합니다. `b` 변수에 또 다른 `Rc<List>` 인스턴스를 생성하여 10 값을 포함하고 `a`에 있는 리스트를 가리킵니다.

`a`를 `b`로 변경하여 사이클을 만듭니다. `tail` 메서드를 사용하여 `a`의 `RefCell<Rc<List>>`에 대한 참조를 가져와 `link` 변수에 저장합니다. 그런 다음 `RefCell<Rc<List>>`의 `borrow_mut` 메서드를 사용하여 내부 값을 `Nil` 값을 가진 `Rc<List>`에서 `b`에 있는 `Rc<List>`로 변경합니다.

이 코드를 실행하면 마지막 `println!` 문을 일시적으로 주석 처리하면 다음과 같은 출력을 얻습니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

`a`와 `b`에 있는 `Rc<List>` 인스턴스의 참조 카운트는 `a`를 `b`로 변경한 후 2가 됩니다. `main` 함수가 끝날 때 Rust는 `b` 변수를 삭제하여 `b` `Rc<List>` 인스턴스의 참조 카운트를 1로 감소시킵니다. 이때 `Rc<List>`가 가리키는 메모리는 아직 삭제되지 않습니다. `Rc<List>`의 참조 카운트가 0이 아니기 때문입니다. 그런 다음 Rust는 `a`를 삭제하여 `a` `Rc<List>` 인스턴스의 참조 카운트를 1로 감소시킵니다. 이 인스턴스의 메모리도 아직 삭제되지 않습니다. 다른 참조가 있기 때문입니다.

 `Rc<List>` 인스턴스가 여전히 참조하고 있기 때문에 할당된 메모리의 해제는 영원히 지연됩니다. 참조 주기가 발생하는 것을 시각적으로 보여주기 위해 15-4 그림을 제시했습니다.

<img alt=\"리스트의 참조 주기\" src=\"img/trpl15-04.svg\" class=\"center\" />

<span class=\"caption\">그림 15-4: 서로 참조하는 리스트 `a` 와 `b` 의 참조 주기</span>

마지막 `println!`을 해제하고 프로그램을 실행하면 Rust는 `a` 가 `b` 에, `b` 가 `a` 에 이르는 순환 참조를 출력하려고 합니다. 이 과정은 스택 오버플로우를 일으킬 때까지 계속됩니다.

실제 프로그램과 비교했을 때, 이 예제에서 참조 주기를 만들면 발생하는 결과는 크게 심각하지 않습니다. 참조 주기가 생성된 직후 프로그램이 종료되기 때문입니다. 그러나 더 복잡한 프로그램에서 많은 메모리를 할당하고 오랫동안 참조하는 경우 프로그램은 필요 이상의 메모리를 사용하여 시스템이 메모리 부족으로 멈추게 할 수 있습니다.

참조 주기를 만들기는 쉽지 않지만 불가능한 것은 아닙니다. `RefCell<T>` 값이 `Rc<T>` 값을 포함하거나 유사한 내부 변경 가능성과 참조 카운팅이 있는 복합적인 유형이 포함된 경우 참조 주기를 만들지 않도록 주의해야 합니다. Rust는 이를 감지하지 못하므로 참조 주기를 만들면 프로그램의 논리 오류가 발생합니다. 자동화된 테스트, 코드 검토 및 기타 소프트웨어 개발 방법을 사용하여 참조 주기를 최소화해야 합니다.

참조 주기를 방지하는 또 다른 방법은 소유권을 나타내는 참조와 소유권을 나타내지 않는 참조로 데이터 구조를 재구성하는 것입니다. 그 결과 소유권 관계와 소유권 관계가 아닌 관계로 구성된 주기가 만들어지며, 소유권 관계만이 값이 삭제될 수 있는지 여부에 영향을 미칩니다. 15-25 목록에서 `Cons` 변형이 항상 자신의 리스트를 소유하도록 하기 때문에 데이터 구조를 재구성하는 것은 불가능합니다. 그래프를 사용하여 부모 노드와 자식 노드를 보여주는 예제를 살펴보고 소유권 관계가 아닌 관계가 참조 주기를 방지하는 데 적절한 경우를 살펴보겠습니다.

### 참조 주기 방지: `Rc<T>` 를 `Weak<T>` 로 변환하기

지금까지 `Rc::clone` 을 호출하면 `Rc<T>` 인스턴스의 `strong_count` 가 증가한다는 것을 보여주었습니다. `Rc<T>` 인스턴스는 `strong_count` 가 0일 때만 정리됩니다. `Rc<T>` 인스턴스 내부의 값에 대한 약 참조를 만들려면 `Rc::downgrade` 를 호출하고 `Rc<T>` 에 대한 참조를 전달할 수 있습니다. 강 참조는 `Rc<T>` 인스턴스의 소유권을 공유하는 방법입니다. 약 참조는 소유권 관계를 나타내지 않으며, `Rc<T>` 인스턴스가 정리될 때까지 카운트가 영향을 미치지 않습니다. 약 참조로 인해 참조 주기가 발생하지 않습니다. 약 참조가 포함된 주기는 강 참조 카운트가 0이 되면 끊어집니다.

`Rc::downgrade` 를 호출하면 `Weak<T>` 유형의 스마트 포인터를 얻습니다. `Rc<T>` 인스턴스의 `strong_count` 를 1 증가시키는 대신 `Rc::downgrade` 를 호출하면 `weak_count` 가 1 증가합니다. `Rc<T>` 유형은 `weak_count` 를 사용하여 `Weak<T>` 참조의 개수를 추적합니다. 차이점은 `weak_count` 가 `Rc<T>` 인스턴스가 정리될 때까지 0이어야 한다는 것입니다.

`Weak<T>` 가 참조하는 값이 삭제되었을 수 있기 때문에 `Weak<T>` 가 가리키는 값을 사용하려면 값이 여전히 존재하는지 확인해야 합니다. 이를 위해 `Weak<T>` 인스턴스에 `upgrade` 메서드를 호출하면 `Option<Rc<T>>` 를 반환합니다. `Rc<T>` 값이 아직 삭제되지 않았다면 `Some` 을 반환하고, `Rc<T>` 값이 삭제된 경우 `None` 을 반환합니다. `upgrade` 가 `Option<Rc<T>>` 를 반환하기 때문에 Rust는 `Some` 케이스와 `None` 케이스를 처리하는 데 유의하고 유효하지 않은 포인터가 발생하지 않습니다.

예를 들어, 항목이 다음 항목만 알고 있는 리스트 대신, 항목이 자식 항목과 부모 항목을 모두 알고 있는 트리를 사용합니다.

#### 트리 데이터 구조 만들기: 자식 노드를 가진 `Node`

처음부터 자식 노드를 알고 있는 트리를 만들겠습니다.
`Node`라는 구조체를 만들어서 자신의 `i32` 값과 자식 `Node` 값에 대한 참조를 저장합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

`Node`가 자식을 소유하고, 이 소유권을 변수와 공유하여 트리의 각 `Node`에 직접 액세스하고 싶습니다. 이를 위해 `Vec<T>` 항목을 `Rc<Node>` 유형의 값으로 정의합니다. 또한 다른 노드가 자식 노드가 되도록 변경하려면 `children`에서 `Vec<Rc<Node>>` 주위에 `RefCell<T>`를 사용합니다.

다음으로, `leaf`라는 이름의 `Node` 인스턴스를 하나 만들고 값 3을 가지고 자식이 없는 `Node`를 만들고, `branch`라는 이름의 다른 인스턴스를 만들고 값 5를 가지고 `leaf`를 하나의 자식으로 만듭니다. Listing 15-27에서 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

<span class=\"caption\">Listing 15-27: 자식이 없는 `leaf` 노드를 만들고 `branch` 노드를 `leaf`를 하나의 자식으로 만듭니다</span>

`leaf`의 `Rc<Node>`를 복사하여 `branch`에 저장합니다. 즉, `leaf` 노드는 이제 `leaf`와 `branch` 두 개의 소유자를 가지게 됩니다. `branch`에서 `leaf`에 도달할 수 있지만, `leaf`에서 `branch`에 도달할 수는 없습니다. 이유는 `leaf`가 `branch`에 대한 참조를 가지고 있지 않고 관계가 없기 때문입니다. `leaf`가 `branch`가 부모임을 알 수 있도록 해야 합니다. 다음 단계에서 이를 해결합니다.

#### 자식에서 부모로 참조 추가

자식 노드가 부모 노드를 인지하도록 하려면 `Node` 구조체 정의에 `parent` 필드를 추가해야 합니다. 하지만 `parent`의 유형을 어떻게 정의해야 할지 고민해야 합니다. `Rc<T>`를 사용할 수 없습니다. `leaf.parent`가 `branch`를 가리키고 `branch.children`이 `leaf`를 가리키기 때문에 `strong_count` 값이 0이 되지 않기 때문입니다.

다른 관점에서 생각해 보면, 부모 노드는 자식 노드를 소유해야 합니다. 부모 노드가 삭제되면 자식 노드도 함께 삭제되어야 합니다. 그러나 자식은 부모를 소유해서는 안 됩니다. 자식 노드가 삭제되면 부모 노드는 여전히 존재해야 합니다. 이것은 약 참조의 경우입니다!

따라서 `parent`의 유형은 `Rc<T>`가 아닌 `Weak<T>`를 사용하여 `RefCell<Weak<Node>>`로 설정합니다. 이제 `Node` 구조체 정의는 다음과 같습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

노드는 부모 노드를 참조할 수 있지만 부모를 소유하지 않습니다. Listing 15-28에서 `main`을 이 새로운 정의를 사용하여 업데이트하여 `leaf` 노드가 부모 노드 `branch`를 참조할 수 있도록 합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

<span class=\"caption\">Listing 15-28: 부모 노드 `branch`에 약 참조를 가진 `leaf` 노드</span>

`leaf` 노드를 만드는 것은 Listing 15-27과 유사하지만 `parent` 필드가 있습니다. `leaf` 노드는 처음에는 부모가 없으므로 새롭고 비어있는 `Weak<Node>` 참조 인스턴스를 만듭니다.

이제 `leaf`의 부모 노드에 대한 참조를 가져오려고 `upgrade` 메서드를 사용하면 `None` 값을 얻습니다. 첫 번째 `println!` 문에서 출력되는 내용을 통해 이를 확인할 수 있습니다.

```text
leaf parent = None
```

`branch` 노드를 생성할 때, `parent` 필드에 또 다른 `Weak<Node>` 참조가 생성됩니다. `branch`는 부모 노드가 없기 때문입니다. `leaf`는 여전히 `branch`의 자식 노드 중 하나입니다. `Node` 인스턴스를 `branch`에 가질 때, `leaf`에 부모 노드인 `branch`에 대한 `Weak<Node>` 참조를 할당할 수 있습니다. `leaf`의 `parent` 필드에 있는 `RefCell<Weak<Node>>`에 `borrow_mut` 메서드를 사용하고, `Rc::downgrade` 함수를 사용하여 `branch`의 `Rc<Node>`에서 `branch`에 대한 `Weak<Node>` 참조를 생성합니다.

`leaf`의 부모를 다시 출력하면 이제 `Some` 변형을 가진 `branch`를 포함하고 있습니다. 이제 `leaf`는 부모 노드에 액세스할 수 있습니다! `leaf`를 출력할 때, Listing 15-26에서 발생했던 스택 오버플로우로 이어지는 무한 루프를 피할 수 있습니다. `Weak<Node>` 참조는 `(Weak)`로 출력됩니다.

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

출력이 무한히 계속되지 않는 것은 이 코드가 참조 주기를 만들지 않았음을 나타냅니다. `Rc::strong_count` 및 `Rc::weak_count`를 호출하여 얻는 값을 살펴보면 이것을 알 수 있습니다.

#### `strong_count` 및 `weak_count`의 변화를 시각화

내부 범위를 생성하고 `branch`의 생성을 그 범위로 옮겨서 `Rc<Node>` 인스턴스의 `strong_count` 및 `weak_count` 값이 어떻게 변하는지 살펴보겠습니다. 이렇게 하면 `branch`가 생성되고 범위를 벗어날 때 발생하는 변화를 볼 수 있습니다. 수정된 내용은 Listing 15-29에 나와 있습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

<span class=\"caption\">Listing 15-29: 내부 범위에서 `branch`를 생성하고 강 및 약 참조 카운트를 검사</span>

`leaf`가 생성된 후, `Rc<Node>`의 강 참조 카운트는 1이고 약 참조 카운트는 0입니다. 내부 범위에서 `branch`를 생성하고 `leaf`와 연결하면 출력할 때 `branch`의 `Rc<Node>`의 강 참조 카운트는 1이고 약 참조 카운트는 1( `leaf.parent`가 `branch`를 가리키는 `Weak<Node>` 참조를 가짐)입니다. `leaf`에서 카운트를 출력하면 강 참조 카운트가 2가 되고, `branch`가 `leaf`의 `Rc<Node>`를 `branch.children`에 저장하기 때문에 강 참조 카운트가 증가하지만 약 참조 카운트는 여전히 0입니다.

내부 범위가 종료되면 `branch`가 범위를 벗어나고 `Rc<Node>`의 강 참조 카운트가 0으로 감소하여 `Node`가 삭제됩니다. `leaf.parent`에서 유래하는 1의 약 참조 카운트는 `Node`가 삭제되는지 여부에 영향을 미치지 않으므로 메모리 누수가 발생하지 않습니다!

범위가 종료된 후 `leaf`의 부모 노드에 액세스하려고 하면 다시 `None`을 얻습니다. 프로그램이 종료될 때 `leaf`의 `Rc<Node>`는 강 참조 카운트가 1이고 약 참조 카운트가 0입니다. `leaf` 변수가 `Rc<Node>`의 유일한 참조가 되기 때문입니다.

`Rc<T>` 및 `Weak<T>`와 그들의 `Drop` 트레이트 구현은 카운트를 관리하고 값을 삭제하는 모든 논리를 구현합니다. `Node`의 정의에서 자식 노드에서 부모 노드로의 관계를 `Weak<T>` 참조로 지정함으로써 부모 노드에 대한 참조 주기를 피할 수 있습니다.
노드가 서로 연결되어 있지만, 참조 주기 및 메모리 누수를 발생시키지 않습니다.

## 요약

이 장에서는 기본적으로 Rust가 일반 참조를 사용하여 제공하는 보장과
권리에서 다른 보장과 권리를 제공하는 스마트 포인터를 사용하는 방법을
 behandeln했습니다. `Box<T>` 유형은 알려진 크기를 가지며 힙에 할당된 데이터를
 가리킵니다. `Rc<T>` 유형은 힙에 있는 데이터에 대한 참조 수를 추적하여
 여러 소유자가 데이터를 가질 수 있도록 합니다. `RefCell<T>` 유형은
 불변 유형이지만 해당 유형의 내부 값을 변경해야 하는 경우에 사용할 수
 있는 유형이며, 컴파일 시점이 아닌 실행 시점에서 대출 규칙을 강제합니다.

또한 `Deref` 및 `Drop` 추상화 인터페이스를 설명했으며, 이는 스마트 포인터의
 많은 기능을 가능하게 합니다. 참조 주기가 메모리 누수를 일으킬 수 있는
 이유와 `Weak<T>`를 사용하여 이를 방지하는 방법을 살펴보았습니다.

이 장이 관심을 끌었고 스마트 포인터를 직접 구현하고 싶다면,
 [\u201cThe Rustonomicon\u201d][nomicon]을 참조하십시오.

다음으로 Rust에서의 병렬 처리에 대해 알아보겠습니다. 몇 가지 새로운
 스마트 포인터를 배우게 될 것입니다.

[nomicon]: ../nomicon/index.html
