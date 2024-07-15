## 설치

첫 번째 단계는 Rust를 설치하는 것입니다. `rustup`를 통해 Rust를 다운로드하여
명령줄 도구를 사용하여 Rust 버전과 관련된 도구를 관리합니다. 다운로드에는 인터넷 연결이 필요합니다.

> 참고: `rustup`을 사용하지 않고 싶은 경우, 다른 Rust 설치 방법에 대한 자세한 내용은 [다른 Rust 설치 방법 페이지][otherinstall]를 참조하십시오.

다음 단계는 Rust 컴파일러의 최신 안정 버전을 설치합니다. Rust의 안정성 보장은 책의 모든 예제가 컴파일되고 새롭게 출시된 Rust 버전에서도 계속 컴파일될 수 있음을 보장합니다. 출력은 Rust가 종종 오류 메시지와 경고를 개선하기 때문에 버전 간에 약간 다를 수 있습니다. 즉, 이 단계를 사용하여 설치한 모든 새롭고 안정적인 Rust 버전은 이 책의 내용과 함께 예상대로 작동해야 합니다.

> ### 명령줄 표기법
>
> 이 장과 책 전체에서 명령 프롬프트에서 입력해야 하는 명령을 보여줍니다. `$`로 시작하는 줄은 터미널에서 입력해야 하는 줄입니다. `$` 문자를 입력할 필요는 없습니다. 이는 각 명령의 시작을 나타내는 명령 프롬프트입니다. `$`로 시작하지 않는 줄은 일반적으로 이전 명령의 출력을 보여줍니다. 또한 PowerShell에 특화된 예제는 `>`를 사용하여 `$` 대신 표시됩니다.

### Linux 또는 macOS에서 `rustup` 설치

Linux 또는 macOS를 사용하는 경우 터미널을 열고 다음 명령을 입력하십시오.

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

이 명령은 스크립트를 다운로드하고 `rustup` 도구를 설치하여 최신 안정 버전의 Rust를 설치합니다. 암호를 입력하라는 메시지가 표시될 수 있습니다. 설치가 성공하면 다음 줄이 표시됩니다.

```text
Rust is installed now. Great!
```

또한 Rust가 컴파일된 출력을 하나의 파일로 연결하는 *링커*가 필요합니다. 이미 링커가 있는 경우가 많습니다. 링커 오류가 발생하면 C 컴파일러를 설치해야 합니다. C 컴파일러는 일반적으로 Rust 패키지에 의존하는 C 코드를 컴파일하기 위해 필요합니다.

macOS에서는 다음을 실행하여 C 컴파일러를 가져올 수 있습니다.

```console
$ xcode-select --install
```

Linux 사용자는 일반적으로 배포판 설명서에 따라 GCC 또는 Clang를 설치해야 합니다. 예를 들어 Ubuntu를 사용하는 경우 `build-essential` 패키지를 설치할 수 있습니다.

### Windows에서 `rustup` 설치

Windows에서 [https://www.rust-lang.org/tools/install][install]로 이동하여 Rust 설치 지침을 따르십시오. 설치 중에 Visual Studio를 설치하라는 메시지가 표시될 수 있습니다. 이는 Rust 프로그램을 컴파일하는 데 필요한 링커와 기본 라이브러리를 제공합니다. 이 단계에 대한 추가 도움이 필요하면 [https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]를 참조하십시오.

이 책의 나머지는 `cmd.exe`와 PowerShell에서 모두 작동하는 명령을 사용합니다. 차이점이 있는 경우 설명합니다.

### 문제 해결

Rust를 올바르게 설치했는지 확인하려면 셸을 열고 다음 줄을 입력하십시오.

```console
$ rustc --version
```

최신 안정 버전의 버전 번호, 커밋 해시 및 커밋 날짜가 다음과 같은 형식으로 표시되어야 합니다.

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

이 정보가 표시되면 Rust를 성공적으로 설치했습니다! 이 정보가 표시되지 않으면 Rust가 `%PATH%` 시스템 변수에 있는지 확인하십시오.

Windows CMD에서 사용하는 경우 다음을 사용하십시오.

```console
> echo %PATH%
```

PowerShell에서 사용하는 경우 다음을 사용하십시오.

```powershell
> echo $env:Path
```

Linux 및 macOS에서 사용하는 경우 다음을 사용하십시오.

```console
$ echo $PATH
```

이 모든 것이 올바르게 설정되어 있지만 Rust가 여전히 작동하지 않는 경우 도움을 받을 수 있는 곳이 많습니다. [커뮤니티 페이지][community]에서 다른 Rustacean(우리 자신을 부르는 어리석은 별명)과 연락하는 방법을 알아보십시오.

### 업데이트 및 제거

`rustup`을 통해 Rust를 설치한 후 새 버전으로 업데이트하는 것은 간단합니다. 셸에서 다음 업데이트 스크립트를 실행하십시오.

```console
$ rustup update
```

Rust와 `rustup`를 제거하려면 셸에서 다음 제거 스크립트를 실행하십시오.

```console
$ rustup self uninstall
```

### 로컬 문서

Rust의 설치에는 온라인에서 읽을 수 없는 로컬 문서 사본도 포함되어 있습니다. `rustup doc`를 실행하여 로컬 문서를 브라우저에서 열 수 있습니다.

표준 라이브러리에서 제공되는 어떤 유형이나 함수가 제공되더라도 확실하지 않은 경우, API 문서를 사용하여 확인하세요!

[다른 설치 방법]: https://forge.rust-lang.org/infra/other-installation-methods.html
[설치]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[커뮤니티]: https://www.rust-lang.org/community
