## Cargo에 사용자 정의 명령어 추가하기

Cargo는 `$PATH`에 있는 바이너리가 `cargo-something`이라는 이름이라면 `cargo something`과 같이 실행할 수 있도록 설계되었습니다. 이러한 사용자 정의 명령어는 `cargo --list`를 실행하면 나열됩니다. `cargo install`을 사용하여 확장 프로그램을 설치하고 내장 Cargo 도구와 마찬가지로 실행할 수 있다는 것은 Cargo의 설계의 매우 편리한 이점입니다!

## 요약

Cargo와 [crates.io](https://crates.io/)<!-- ignore -->를 통해 코드를 공유하는 것은 Rust 생태계가 다양한 작업에 유용하게 사용되는 데 기여하는 요소입니다. Rust의 표준 라이브러리는 작고 안정적이지만, crates는 공유, 사용 및 언어의 시간표와 다른 시간표로 개선하기 쉽습니다. [crates.io](https://crates.io/)<!-- ignore -->에 자신에게 유용한 코드를 공유하는 것을 주저하지 마세요; 다른 사람에게도 유용할 가능성이 높습니다!
