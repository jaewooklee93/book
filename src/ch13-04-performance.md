## 성능 비교: 루프 vs. 이터레이터

루프와 이터레이터 중 어떤 것을 사용할지 결정하려면 `search` 함수의 구현 중 어떤 것이 더 빠른지 알아야 합니다. `for` 루프를 사용하는 `search` 함수 버전과 이터레이터를 사용하는 버전을 비교해 보겠습니다.

*셜록 홈즈의 모험* 전체 내용을 `String`에 로드하고 내용에서 *the* 단어를 찾는 작업을 통해 벤치마크를 실행했습니다. `for` 루프를 사용하는 `search` 버전과 이터레이터를 사용하는 버전의 벤치마크 결과는 다음과 같습니다.

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

이터레이터 버전이 약간 더 빠릅니다! 벤치마크 코드를 자세히 설명하지는 않겠습니다. 중요한 것은 두 버전이 동등한지 증명하는 것이 아니라, 이 두 가지 구현 방식이 성능 측면에서 어떻게 비교되는지에 대한 일반적인 감각을 얻는 것입니다.

보다 포괄적인 벤치마크를 위해서는 `contents`로 다양한 크기의 여러 텍스트를 사용하고, `query`로 다양한 단어와 길이의 단어를 사용하고, 다른 모든 변형을 테스트해야 합니다. 핵심은 이터레이터는 고수준 추상화이지만, 거의 동일한 코드로 컴파일되는 것입니다. 즉, 스스로 낮은 수준의 코드를 작성했을 때와 같습니다. 이터레이터는 Rust의 *비용이 없는 추상화* 중 하나이며, 이는 추상화를 사용하는 것이 추가적인 런타임 오버헤드를 초래하지 않는다는 것을 의미합니다. 이는 Bjarne Stroustrup가 C++의 원래 설계자이자 구현자로서 *비용이 없는* 것을 정의하는 방식과 유사합니다.

```cpp
> 일반적으로 C++ 구현은 비용이 없는 원칙을 따릅니다. 사용하지 않는 것은, 그 대가를 치르지 않습니다. 그리고 더 나아가, 사용하는 것은 손으로 작성할 수 없을 정도로 좋습니다.
```

또 다른 예로, 다음 코드는 오디오 디코더에서 가져온 것입니다. 디코딩 알고리즘은 이전 샘플에 대한 선형 함수를 기반으로 미래 값을 추정하는 선형 예측 수학적 연산을 사용합니다. 이 코드는 `buffer` 데이터 슬라이스, 12개의 `coefficients` 배열, `qlp_shift`에서 데이터를 이동하는 양에 대한 세 가지 변수에 대한 수학 연산을 수행하는 이터레이터 체인을 사용합니다. 변수는 이 예제 내에서 선언되었지만 값이 주어지지 않았습니다. 이 코드는 맥락 외부에서는 의미가 없지만, Rust가 고수준 개념을 낮은 수준 코드로 변환하는 방법에 대한 간결하고 실제 세계의 예입니다.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

`prediction` 값을 계산하기 위해 이 코드는 `coefficients`의 각 값을 순회하고 `zip` 메서드를 사용하여 계수 값과 `buffer`의 이전 12 값을 쌍으로 연결합니다. 그런 다음 각 쌍에 대해 값을 곱하고 모든 결과를 합산한 후 `qlp_shift` 비트를 오른쪽으로 이동합니다.

오디오 디코더와 같은 애플리케이션에서 계산은 종종 성능을 가장 중요하게 고려합니다. 여기에서는 이터레이터를 생성하고 두 개의 어댑터를 사용하고 값을 소비합니다. Rust 코드가 어떤 어셈블리 코드로 컴파일될까요? 현재까지 이 코드는 손으로 작성한 어셈블리 코드와 동일하게 컴파일됩니다. 루프를 순회하는 루프에 해당하는 어셈블리 코드가 전혀 없습니다. Rust는 반복이 12번이라는 것을 알고 있기 때문에 루프를 *펼칩니다*. *펼치기*는 루프 제어 코드의 오버헤드를 제거하고 각 반복에 대한 반복 코드를 생성하는 최적화입니다.

모든 계수가 레지스터에 저장되므로 값에 액세스하는 것이 매우 빠릅니다. 런타임에 배열 액세스에 대한 경계 검사가 없습니다. Rust가 적용할 수 있는 모든 이러한 최적화는 결과 코드를 매우 효율적으로 만듭니다.

매우 효율적입니다. 이제 이것을 알고 있다면, 이터레이터와 클로저를
두려워하지 않고 사용할 수 있습니다! 코드가 더 높은 수준처럼 보이지만
실행 시간 성능에 대한 페널티를 부과하지 않습니다.

## 요약

클로저와 이터레이터는 함수형 프로그래밍 언어 개념에서 영감을 받은 Rust
의 기능입니다. 이들은 Rust가 저수준 성능에서 높은 수준의 개념을 명확하게
 표현할 수 있는 능력에 기여합니다. 클로저와 이터레이터의 구현은 실행 시간
 성능에 영향을 미치지 않도록 설계되었습니다. 이는 Rust의 목표인
 제로 코스트 추상화를 제공하려는 노력의 일부입니다.

이제 I/O 프로젝트의 표현력을 향상시켰으므로, 프로젝트를 세상과 공유하는 데
 도움이 될 `cargo`의 몇 가지 더 많은 기능을 살펴보겠습니다.
