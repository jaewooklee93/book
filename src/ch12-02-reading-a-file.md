## 파일 읽기

이제 `file_path` 인자로 지정된 파일을 읽는 기능을 추가하겠습니다. 먼저 테스트할 샘플 파일이 필요합니다. 몇 줄에 걸쳐 텍스트가 있는 파일을 사용하고, 몇 개의 단어가 반복되는 것을 테스트할 것입니다. 12-3번 목록에는 작은 텍스트 파일이 있는데, 이를 사용해 보세요! 프로젝트의 루트 레벨에 *poem.txt* 라는 파일을 만들고, "I'm Nobody! Who are you?"라는 시를 입력하세요.

<Listing number=\"12-3\" file-name=\"poem.txt\" caption=\"Emily Dickinson 시인의 시는 좋은 테스트 케이스입니다\">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

텍스트를 입력한 후 *src/main.rs* 파일을 편집하여 파일을 읽는 코드를 추가하세요. 12-4번 목록을 참조하세요.

<Listing number=\"12-4\" file-name=\"src/main.rs\" caption=\"두 번째 인자로 지정된 파일의 내용을 읽습니다\">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

먼저 `use` 문으로 표준 라이브러리의 관련 부분을 가져옵니다. 파일을 처리하기 위해 `std::fs`가 필요합니다.

`main` 함수에서 `fs::read_to_string` 함수는 `file_path`를 받아 파일을 열고 파일 내용의 `std::io::Result<String>`을 반환합니다.

그 후, 파일이 읽힌 후 `contents` 값을 출력하는 임시 `println!` 문을 추가하여 프로그램이 작동하는지 확인합니다.

첫 번째 명령줄 인자로 임의의 문자열을 사용하고 (아직 검색 부분을 구현하지 않았기 때문에), 두 번째 인자로 *poem.txt* 파일을 사용하여 코드를 실행해 보세요.

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

훌륭합니다! 코드가 파일을 읽고 내용을 출력했습니다. 하지만 코드에는 몇 가지 문제가 있습니다. 현재 `main` 함수는 여러 가지 책임을 가지고 있습니다. 일반적으로 각 함수는 하나의 개념만 담당하는 것이 명확하고 유지 관리하기 쉽습니다. 또 다른 문제는 오류를 제대로 처리하지 않는 것입니다. 프로그램이 아직 작기 때문에 이러한 문제는 큰 문제가 되지 않지만, 프로그램이 커질수록 문제를 명확하게 해결하기 어려워집니다. 프로그램을 개발할 때는 작은 코드 양을 재작성하는 것이 훨씬 쉬우므로, 가능한 빨리 재작성하는 것이 좋습니다. 다음 단계로 이를 수행하겠습니다.
