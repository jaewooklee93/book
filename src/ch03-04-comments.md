## 주석

모든 프로그래머는 코드를 이해하기 쉽게 만들려고 노력하지만, 때때로 추가 설명이 필요합니다. 이러한 경우 프로그래머는 컴파일러가 무시하지만 코드를 읽는 사람들에게 유용할 수 있는 *주석*을 소스 코드에 남깁니다.

다음은 간단한 주석입니다.

```rust
// hello, world
```

Rust에서 고유한 주석 스타일은 두 개의 슬래시로 주석을 시작하며, 주석은 줄의 끝까지 계속됩니다. 여러 줄에 걸쳐지는 주석의 경우, 각 줄에 `//`을 포함해야 합니다.

```rust
// 여기서 복잡한 작업을 하고 있습니다. 설명을 위해 여러 줄의 주석이 필요합니다. 흠! 이 주석이 무슨 일이 일어나는지 설명할 것입니다.
```

주석은 코드가 포함된 줄의 끝에 배치할 수도 있습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

그러나 주석을 코드를 설명하는 별도의 줄에 배치하는 형식을 더 자주 볼 수 있습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust에는 또 다른 종류의 주석인 문서 주석이 있습니다. 이 주석은 제14장의 [\u201cCrates.io에 crate 게시하기\u201d][publishing]<!-- ignore -->
 섹션에서 논의할 것입니다.

[publishing]: ch14-02-publishing-to-crates-io.html
