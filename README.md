# Logic Minimizer using Quine–McCluskey Algorithm

<p align="center">
  <img src="https://github.com/HigherIdeal/Logic-minimizer-Quine-McCluskey/releases/latest/download/<GIF_FILE_NAME>.gif" width="720">
</p>

10변수 이상의 Boolean logic minimization을 목표로 제작한 C++ 기반 논리최소화 프로그램입니다. 4변수 이하의 논리식은 Karnaugh map으로 비교적 쉽게 처리할 수 있고, 8변수 수준까지는 일부 무료 도구로도 접근할 수 있습니다. 그러나 변수 수가 증가할수록 계산 시간이 길어지거나 결과 검증이 어려운 경우가 많으며, 9변수 이상부터는 유료 프로그램을 사용해야 하는 경우도 많습니다.

본 프로젝트는 대학생, 연구실 동료, 디지털 논리설계를 학습하는 사용자가 다변수 논리최소화를 무료로 수행할 수 있도록 제작되었습니다. 프로그램은 truth table을 입력받아 Quine–McCluskey 알고리즘 기반으로 Prime Implicant를 생성하고, PI chart를 이용해 간략화된 Boolean expression을 출력합니다.

---

## Quine–McCluskey 알고리즘이란?

Quine–McCluskey 알고리즘은 Boolean function을 최소화하기 위한 표 기반 논리최소화 알고리즘입니다. Karnaugh map과 동일하게 인접한 minterm을 묶어 변수를 제거하는 원리를 사용하지만, 이를 사람이 직접 그림으로 처리하는 대신 표와 반복 연산으로 일반화합니다. 따라서 4변수 이상처럼 Karnaugh map이 복잡해지는 경우에도 알고리즘적으로 논리식을 최소화할 수 있습니다.

이 알고리즘은 먼저 출력이 1인 minterm과 don't-care 항을 모은 뒤, 한 비트만 다른 항들을 서로 병합합니다. 병합된 위치는 0과 1 중 어느 값이어도 상관없는 don't-care 위치가 됩니다. 이 과정을 더 이상 병합 가능한 항이 없을 때까지 반복하면 Prime Implicant가 생성됩니다. 이후 Prime Implicant들이 실제 minterm을 어떻게 커버하는지 PI chart를 구성하고, 필요한 implicant를 선택하여 최종 논리식을 만듭니다.

Quine–McCluskey 알고리즘의 장점은 절차가 명확하고 체계적이라는 점입니다. 반면 변수 수와 minterm 수가 증가하면 병합 후보 수가 빠르게 증가하므로 계산량과 메모리 사용량이 커집니다. 따라서 단순한 구현으로는 10변수 이상의 입력에서 실행 시간이 길어질 수 있으며, 구현 방식에 따라 성능 차이가 크게 발생합니다.

예를 들어 다음 두 minterm을 생각할 수 있습니다.

```text
10010
10011
```

두 항은 마지막 비트만 다르므로 하나의 implicant로 병합할 수 있습니다.

```text
1001-
```

여기서 `-`는 해당 위치가 0이어도 되고 1이어도 되는 don't-care 위치를 의미합니다. 즉, 두 개의 minterm을 하나의 항으로 묶으면서 최종 논리식에서 하나의 변수를 제거할 수 있습니다.

---

## 구현 사항

본 구현의 핵심은 문자열 비교 대신 정수 기반 비트 연산을 사용했다는 점입니다. 일반적인 설명에서는 minterm을 `10010`, `10011`과 같은 문자열로 표현하지만, 문자열 방식은 각 자리 문자를 반복적으로 비교해야 합니다. 본 프로그램은 minterm을 정수로 저장하고, 두 항의 차이를 XOR 연산으로 계산하여 병합 가능 여부를 빠르게 판단합니다.

```cpp
diff = a ^ b;
```

두 implicant가 정확히 한 비트만 다르고, 기존 don't-care mask가 동일하면 병합 가능한 항으로 처리합니다. 병합된 항은 value와 mask 형태로 관리됩니다. value는 실제 비트값을 의미하고, mask는 don't-care 처리된 비트 위치를 의미합니다. 이 구조를 사용하면 문자열을 직접 수정하거나 비교하지 않고도 implicant의 병합, 중복 검사, minterm coverage 판정을 수행할 수 있습니다.

또한 중간 후보 배열이 과도하게 증가하는 것을 줄이기 위해 현재 단계와 다음 단계의 후보군을 분리하여 관리합니다. 병합 과정에서는 현재 단계의 implicant를 비교해 다음 단계의 후보만 생성하고, 병합되지 않은 항만 Prime Implicant로 저장합니다. 즉, 모든 가능한 문자열 조합을 한 번에 전개하는 방식이 아니라, 단계별 후보군을 갱신하면서 필요한 Prime Implicant만 남기는 구조입니다.

최종 단계에서는 PI chart를 구성하여 각 Prime Implicant가 어떤 minterm을 커버하는지 확인합니다. 먼저 특정 minterm을 유일하게 커버하는 Essential Prime Implicant를 선택하고, 이후 남은 minterm에 대해서는 가장 많은 항을 커버하는 implicant를 반복적으로 선택합니다. 따라서 본 프로그램은 Quine–McCluskey 기반의 Prime Implicant 생성을 수행하지만, 최종 선택 단계에서 Petrick’s method를 사용하지는 않습니다.

---

## 사용법

Release 페이지에서 실행 파일을 다운로드할 수 있습니다.

[Download from GitHub Releases](https://github.com/HigherIdeal/Logic-minimizer-Quine-McCluskey/releases)

프로그램을 실행하면 먼저 변수 개수를 입력합니다.

```text
How many variables are used?:
```

예를 들어 5변수 Boolean function을 최소화하려면 다음과 같이 입력합니다.

```text
5
```

5변수 함수는 총 2^5 = 32개의 truth table 출력값을 입력해야 합니다. 입력은 minterm index 순서대로 넣으면 됩니다.

```text
00010110x0010110010x1110000101x0
```

입력값의 의미는 다음과 같습니다.

| 입력     | 의미                        |
| ------ | ------------------------- |
| `0`    | 출력 0, 커버하지 않는 항           |
| `1`    | 출력 1, 반드시 커버해야 하는 minterm |
| 그 외 문자 | don't-care 항              |

예를 들어 `x`, `d`, `-` 등은 모두 don't-care 항으로 처리됩니다.

입력값을 콘솔에 하나씩 직접 입력할 필요는 없습니다. Excel, 메모장, truth table 생성기 등에서 출력 열을 만든 뒤, 해당 값을 그대로 복사하여 콘솔에 붙여넣으면 됩니다. 예를 들어 Excel에서 32개 이상의 출력값을 한 열로 만든 뒤 복사해 붙여넣어도 사용할 수 있습니다.

계산이 완료되면 다음과 같은 형태로 최소화된 Boolean expression이 출력됩니다.

```text
minimized boolean expression : A'B + AC
```

변수 이름은 `A`, `B`, `C`, ... 순서로 출력되며, 보수는 `'` 기호로 표시됩니다.

본 레포지토리의 프로그램은 Windows 콘솔 환경에서 동작하도록 구현되었습니다. 개발용 Linux 환경에서 사용하고 싶은 경우 별도의 수정이 필요하므로, 필요하다면 아래 연락처로 문의해 주세요.

---

## Contact

문의 사항은 아래 이메일로 연락해 주세요.

```text
kjwhigherideal[at]gmail.com
```

김정우
Jung-Woo Kim
