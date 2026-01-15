# 0) Linux Kernel Coding Style (v5.8)

다음 이미지는 리눅스 커널 코딩 스타일 페이지의 모습이다.

![Linux Kernel Coding Style](./assets/images/linux%20kernel%20style%20overview.png)

[여기](https://www.kernel.org/doc/html/v5.8/process/coding-style.html)에서 확인할 수 있다.

문서의 개정이 이뤄질수도 있으니, 원문도 같이 보는것을 추천드린다.

필자는 원문을 같이 보고 번역을 하면서 공부하였다. 다소 의역이 있을 것이다.

리눅스 커널 코딩 스타일은 단순히 규칙을 너머, 전 세계 개발자들이 거대한 시스템을 함께 유지보수하기 위한 약속과도 같다. 그러니 잘 알아두면 좋을 것이다.

# 소개
이 문서는 리눅스 커널에서 선호되는 coding style을 설명하는 짧은 문서입니다. Coding style은 매우 개인적인 영역이며, 저는 제 방식을 다른 이에게 **강요**하고 싶지 않습니다. 하지만 제가 유지보수해야 하는 코드에 대해서는 이야기가 다르며, 다른 많은 커널 유지보수자(maintainer)들의 경우도 마찬가지입니다. 여러분이 작성한 patch가 수락되기를 바란다면, 이 규칙들을 따라주시기 바랍니다.

Coding style의 주된 목적은 가독성과 유지보수성입니다. 코드가 어떻게 작성되어야 하는지에 대한 절대적인 정답은 없지만, 커널 커뮤니티가 공유하는 공통의 스타일을 따르는 것은 코드를 읽는 사람들이 로직 자체에 집중할 수 있게 도와줍니다.

# 1) Indentation (들여쓰기)
`Tab`은 8자(character)이며, 따라서 indentation 또한 8자입니다. 어떤 사람들은 indentation을 4자(혹은 2자!)로 만드는 것을 선호하며, 이는 마치 $sin(x)$의 값을 2로 정의하려는 시도와 같습니다.

Indentation의 핵심 아이디어는 control block이 어디서 시작하고 끝나는지를 명확하게 정의하는 것입니다. 특히 당신이 20시간 연속으로 깨어있을 때, indentation이 8자로 크게 설정되어 있으면 소스 코드를 훨씬 쉽게 파악할 수 있습니다.

이제 어떤 사람들은 8자의 indentation이 indentation이 코드를 너무 오른쪽으로 밀어버려서 80자 column 제한 내에 코드를 작성하기 어렵게 만든다고 주장합니다. 그에 대한 답은 간단합니다. 만약 당신에게 3단계 이상의 indentation이 필요하다면, 당신의 코드는 엉망인 상태이므로 해당 function을 수정해야 합니다.

요약하자면, `tab`은 8자여야 하며, indentation 또한 `tab`을 사용해야 합니다. Indentation을 위해 `space`를 사용하지 마십시오.

# 2) Breaking Long Lines and Strings
Coding style은 가독성에 관한 것이며, 이를 위해 tool을 사용하기도 합니다. 한 line의 길이는 가급적 80 column으로 제한하는 것이 좋습니다. 이는 오랫동안 유지되어 온 관습이며, 많은 편집기나 터미널의 기본 설정과 일치합니다.

만약 한 statement가 80 column을 넘어간다면, 이를 적절하게 나누어야 합니다. 나뉜 line은 원래 line보다 더 깊게 indent되어야 하며, 호출되는 function의 parameter 리스트 중간에서 나뉘는 경우 이전 line의 여는 괄호 위치에 맞추는 것이 좋습니다.

하지만, user-visible strings인 `printk` 메세지 등은 중간에 나누지 마십시오. 왜냐하면 문자열을 나누게 되면 해당 메시지를 `grep`으로 찾을 수 없게 되기 때문입니다. 비록 80 column을 조금 넘더라도, 문자열은 한 line에 온전하게 두는 것이 좋습니다.

# 3) Placing Braces and Spaces
## 3-1) Braces (중괄호)
커널 스타일에서 권장하는 brace 배치는 Kernighan & Ritchie(K&R) 스타일입니다. non-function block(`if, switch, for, while` 등)의 경우, 여는 brace는 statement와 같은 line에 둡니다.

```C
if (x is true) {
        we do y
}
```

그러나 function 정의의 경우, 여는 brace는 다음 line의 시작 부분에 둡니다.

```C
int function(int x) 
{
        body of function
}
```

닫는 brace는 그 자체로 한 line을 차지하는 것이 기본이지만, 같은 statement의 연속인 경우에는 예외입니다. 예를 들어 `do` statement의 `while`이나 `if` statement의 `else`는 닫는 brace와 같은 line에 위치합니다.

```C
if (x == y) {
        ...
} else if (x > y) {
        ...
} else {
        ...
}

do {
        ...
} while (condition);
```

또한, 단일 statement만 포함하는 경우에는 brace를 생략하는 것이 원칙입니다.

```C
if (condition)
        do_this();
```

단, `if`나 `else` 중 하나라도 복수 statement를가져서 brace를 써야 한다면, 나머지 조건문에서도 가독성을 위해 brace를 함께 써주는 것이 좋습니다.

## 3-2) Spaces (공백)
리눅스 커널에서 `space`의 사용은 주로 function과 keyword를 구분하는 데 쓰입니다. `if`, `switch`, `case`, `for`, `do`, `while` 같은 keyword 뒤에는 `space`를 사용하십시오. 반면 function 호출 뒤에는 space를 붙이지 않습니다.

```C
// Good
if (condition)
        foo();

// Bad
if(condition)
        foo ();
```
대부분의 binary 또는 ternary operator  주위에는 space를 하나씩 둡니다. `=`, `+`, `-`, `<`, `>`, `*`, `/`, `%`, `|`, `&`, `^`, `&&`, `||`, `?`, `:`, 등이 해당됩니다.

하지만 unary operator 뒤에는 space를 두지 않습니다. `&`(address-of), `*`(dereference), `+`, `-`, `~`, `!`, `sizeof`, `typeof`, `alingnof`, `__attribute__`, `defined` 등이 이에 해당합니다.

# 4) Naming
C는 Spartan 언어이며, 여러분의 naming 또한 그래야 합니다. Modula-2나 Pascal 프로그래머들과 달리, C 프로그래머들은 `ThisVariableIsATemporaryCounter` 같은 이름을 쓰지 않습니다. C 프로그래머는 그 variable을 `tmp`라고 부를 것입니다.

이름이 길다고 해서 가독성이 좋아지는 것은 아닙니다. 오히려 짧고 명확한 이름이 코드의 흐름을 파악하기 좋게 만듭니다.
- **Local Variable:** 최대한 간결하게 짓습니다. 루프 인덱스는 `i`, `j`, `k`면 충분합니다. 임식 변수는 `tmp`면 됩니다.
- **Global Variable:** 전역적으로 사용되는 만큼, 좀 더 서술적인 이름이 필요합니다. `count` 같은 모호한 이름 대신 `total_cpu_count`처럼 의미를 담아야 합니다.
- **CamelCase:** 리눅스 커널에서는 CamelCase(예: `MyVariable`)를 절대 사용하지 않습니다. 대신 **snake_case**(예: `my_variable`)를 사용하십시오.

# References
- [Official] Linux Kernel Coding Style 
- [Blog] mythos.log: Linux Tutorial #2 (코딩 스타일 1장)