## I/O 프로젝트: 명령줄 프로그램 구축

이 장은 지금까지 배운 많은 기술을 복습하고 몇 가지 표준 라이브러리 기능을 탐색합니다. 명령줄 도구를 만들어서 지금까지 배운 Rust 개념을 실제로 적용해 보겠습니다.

Rust의 속도, 안전성, 단일 바이너리 출력 및 플랫폼 지원은 명령줄 도구를 만들기에 이상적인 언어입니다. 따라서 프로젝트에서는 고전적인 명령줄 검색 도구인 `grep`을 우리만의 버전으로 만들겠습니다. (`grep`은 **g**lobally search a **r**egular **e**xpression and **p**rint의 약자입니다.) 가장 간단한 사용 사례에서 `grep`은 지정된 파일에서 지정된 문자열을 검색합니다. 이를 위해 `grep`은 파일 경로와 문자열을 인수로 받습니다. 그런 다음 파일을 읽고 해당 파일의 문자열 인수를 포함하는 줄을 찾아 출력합니다.

이 과정에서 명령줄 도구가 다른 명령줄 도구에서 사용하는 터미널 기능을 사용하는 방법을 보여드리겠습니다. 환경 변수의 값을 읽어 사용자가 도구의 동작을 구성하도록 할 것입니다. 또한 오류 메시지를 표준 오류 콘솔 스트림 (`stderr`)으로 출력하여 사용자가 성공적인 출력을 파일로 리디렉션하더라도 화면에 오류 메시지를 볼 수 있도록 합니다.

Rust 커뮤니티 회원 Andrew Gallant는 `ripgrep`이라는 완벽하고 매우 빠른 `grep` 버전을 이미 만들었습니다. 우리 버전은 상대적으로 간단하지만 이 장은 `ripgrep`와 같은 실제 프로젝트를 이해하는 데 필요한 배경 지식을 제공합니다.

우리의 `grep` 프로젝트는 지금까지 배운 몇 가지 개념을 결합합니다.

* 코드 정리 (제7장에서 배운 모듈에 대한 지식을 사용하여 [Chapter 7][ch7]<!--
  ignore -->)
* 벡터와 문자열 사용 (컬렉션, [Chapter 8][ch8]<!-- ignore -->)
* 오류 처리 ([Chapter 9][ch9]<!-- ignore -->)
* 트레이트와 라이프타임 적절히 사용 ([Chapter 10][ch10]<!-- ignore
  -->)
* 테스트 작성 ([Chapter 11][ch11]<!-- ignore -->)

또한 클로저, 이터레이터, 트레이트 객체를 간략하게 소개합니다. 이러한 내용은 [Chapter 13][ch13]<!-- ignore --> 및 [Chapter 17][ch17]<!-- ignore -->에서 자세히 다룹니다.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch17]: ch17-00-oop.html
