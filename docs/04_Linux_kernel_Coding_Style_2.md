# 5) Typedefs (타입 정의)
제발 `vps_t`와 같은 방식을 사용하지 마십시오. `struct`나 `pointer`에 `typedef`를 사용하는 것은 **실수**입니다. 다음과 같은 경우에만 `typedef`를 사용하십시오.

**1. 완전히 불투명한 타입(Totally opaque objects):** 해당 타입의 accessor function을 통하지 않고는 내용에 접근할 수 없는 경우입니다. (e.g. `sigset_t`, `atomic_t`)

**2. 명확한 정수 타입:** 추상화가 나중에 `int`에서 `long`으로 바뀔 가능성이 있는 있는 경우입니다. (e.g. `sector_t`)

**3. sparse를 이용한 타입 체크:** `__le32`처럼 endian 체크 등을 위해 새로운 타입을 만드는 경우입니다.


**4. 표준 C99 타입과 동일한 타입:** 특정 상황에서 이식성을 위해 정의된 타입들입니다.

기본적으로, 어떤 variable의 요소에 직접 접근할 수 있다면(Member Access), 그 타입은 `typedef`여서는 안됩니다. `typedef`는 정보를 숨기기 위한 용도이지, 단순히 타이핑을 줄이기 위한 용도가 아니기 때문입니다. `struct`를 그대로 쓰면 코드를 읽는 사람이 "아, 이건 구조체구나"라고 바로 알 수 있지만, `typedef`는 그 실체를 확인하기 위해 정의를 다시 찾아보게 만듭니다.

# 6) Functions
Function은 짧고 간결해야 하며, **오직 한 가지 일**만 수행해야 합니다. Function은 코드 작성자의 모니터에 한두 화면 정도 분량에 들어와야 하며(ISO/ANSI 표준 스크린 크기인 80x24 기준), 한 가지 일을 제대로 해내야 합니다.

만약 function이 너무 길다면, 그 안에서 서로 다른 수많은 일을 하고 있을 가능성이 큽니다. 이럴 때는 보조 function(helper function)으로 분리하십시오. 비록 그 helper function이 단 한 곳에서만 호출되더라도, 로직을 분리하는 것이 가독성에 큰 도움이 됩니다.

또한, function 내의 local variable 개수는 5~10개를 넘지 않는 것이 좋습니다. 이를 넘긴다면 당신은 무언가 잘못하고 있는 것입니다. function을 다시 쪼개십시오.

# 7) Centralized Exiting of Functions (함수의 집중화된 종료)
많은 사람들에게 교육된 것과는 반대로, `goto` statement는 커널에서 매우 유용하게 쓰입니다. 특히 여러 리소스를 할당받았다가 종료 시점에 해제해야 하는 function에서 `goto`는 빛을 발합니다.

`goto`는 다음과 같은 상황에서 주로 쓰입니다.
- 무조건적인 cleanup이 필요한 경우
- 중첩된 `if`문을 제거하여 indentation 레벨을 낮추고 싶은 경우
- 공통된 종료 로직을 한데 모으고 싶은 경우

```C
int fun(int a)
{
        int result = 0;
        char * buffer;

        buffer = kmalloc(SIZE, CFP_KERNEL);
        if (!buffer)
                return -ENOMEM;
        
        if (condition1) {
                while (loop1) {
                        ...
                }
                result = 1;
                goto out_free_buffer;
        }
        ...
out_free_buffer:
        kfree(buffer);
        return result;
}
```

Error handling을 할 때, 단 하나의 `err:` label만 사용했을 때 발생할 수 있는 버그가 있습니다 (One err bugs)

```C
// The Problematic Code
err:
        kfree(foo->bar);
        kfree(foo);
        return ret;
```
- 어떤 에러가 발생하든 이 `err:` label로 jump해서 리소스를 해제하려고 합니다.
- 만약 `foo`라는 구조체를 할당받기도 전에(즉, `foo`가 **NULL**인 상태에서) 에러가 발생해서 이 `err:`로 jump하게 되면 어떻게 될까요?
- `kfree(foo->bar)`를 실행하는 순간, **NULL pointer dereference**(NULL인 주소의 멤버에 접근)가 발생해서 커널이 크래시(panic)가 나버립니다.

이를 해결하기 위해 리소스가 할당된 역순으로 여러 개의 label을 만들어주는 방식을 사용합니다.
```C
// The Fix: Multiple Lables (순차적 해제)
err_free_bar:
        kfree(foo->bar);
err_free_foo:
        kfree(foo);
        return ret;
```
1. `foo` 할당 실패 시 -> 그냥 `return`하거나, 아무것도 해제하지 않는 label로 이동
2. `foo`는 성공했지만 `foo->bar` 할당 실패 시 -> `err_free_foo:`로 jump해서 `foo`만 해제
3. 모든 할당 후 이후 로직에서 실패 시 -> `err_free_bar:`로 jump. 그러면 **순차적으로** `foo->bar`를 해제하고, 바로 아래에 있는 `err_free_foo:`까지 타고 내려가서 `foo`까지 안전하게 해제합니다.

즉, 리소스를 A -> B -> C 순서로 할당했다면, 에러 처리는 C -> B -> A 순서로 해제하도록 label을 배치하는 게 국룰입니다.


이 방식은 error path에서 발생하기 쉬운 resource leak를 방지해 줍니다. Label의 이름은 해당 `goto`가 **무엇을 하는지** 또는 **왜 그곳으로 가는지**를 나타내야 합니다. (e.g. `out_free_buffer:`, `out_unlock:`)


# 8) Commenting
주석은 좋은 것이지만, 과도한 주석은 오히려 해롭습니다. 주석으로 코드가 **어떻게** 작동하는지 설명하려 하지 마십시오. 대신 코드가 **왜** 그렇게 작성되었는지를 설명하십시오.

코드가 너무 복잡해서 설명이 필요하다면, 주석을 달기 전에 코드를 다시 작성하여 이해하기 쉽게 만드십시오. 일반적으로 주석은 function의 서두에 데이터를 목적이나 사용법을 적는 데 사용하며, function 내부에는 정말 까다로운 로직이 아닌 이상 주석을 남발하지 마십시오.

커널은 **C89 style** 주석을 선호합니다. `//`보다는 `/* ... */`를 사용하십시오.
```C
/*
 * This is the preferred style for multi-line
 * comments in the Linux kernel source code.
 * Please use it consistently.
 *
 * Description:  A column of asterisks on the left side,
 * with beginning and ending almost-blank lines.
 */
```

`net/`, `drivers/net/` 등의 디렉토리들에 있는 파일들은 일반적인 커널 주석 스타일과 조금 다릅니다. (네트워킹 코드 스타일)


```C
/* The preferred comment style for files in net/ and drivers/net
 * looks like this.
 *
 * It is nearly the same as the generally preferred comment style,
 * but there is no initial almost-blank line.
 */
```

보통은 첫 줄을 `/*`만 쓰고 비우지만, 여기서는 바로 설명을 시작합니다.

변수(data)를 선언할 때도 주석이 중요합니다. 한 line에 여러 개를 선언하지 말고(comma 사용 금지), **한 line당 딱 하나의 data만 선언**해야 합니다. 그래야 각 아이템 옆에 그 변수가 무엇을 하는지 짧은 comment를 달 수 있는 공간이 생기기 때문입니다.

데이터 구조(struct 등)를 설명할 때는 `kernel-doc`형식을 따라야 합니다. 자세한 내용은 `Documentation/doc-guide/`를 참조하십시오.

# 9) You've made a mess of it
괜찮습니다. 우리 모두 그런 실수를 하니까요. 아마 오랫동안 Unix를 사용해 온 지인으로부터 **GNU emacs**가 C 소스 포맷팅을 자동으로 해준다는 말을 들었을 것이고, 실제로 그렇다는 것도 확인했을 것입니다.하지만 emacs의 기본 설정은 기대 이하입니다. (사실, 아무렇게나 타이핑하는 것보다 못합니다. 무한히 많은 원숭이가 GNU emacs로 타이핑을 한다 해도 절대 좋은 프로그램을 만들 수 없을 정도입니다.)

따라서 GNU emacs를 버리거나, 아니면 더 합리적인 값(saner values)을 사용하도록 설정을 바꿔야 합니다. 후자를 선택하겠다면, `.emacs` 파일에 다음과 같은 내용을 넣으십시오.

```코드 스니펫
(defun c-lineup-arglist-tabs-only (ignored)
  "Line up argument lists by tabs, not spaces"
  (let* ((anchor (c-langelem-pos c-syntactic-element))
         (column (c-langelem-2nd-pos c-syntactic-element))
         (offset (- (1+ column) anchor))
         (steps (floor offset c-basic-offset)))
    (* (max steps 1)
       c-basic-offset)))

(dir-locals-set-class-variables
 'linux-kernel
 '((c-mode . (
        (c-basic-offset . 8)
        (c-label-minimum-indentation . 0)
        (c-offsets-alist . (
                (arglist-close         . c-lineup-arglist-tabs-only)
                (arglist-cont-nonempty .
                    (c-lineup-gcc-asm-reg c-lineup-arglist-tabs-only))
                (arglist-intro         . +)
                (brace-list-intro      . +)
                (c                     . c-lineup-C-comments)
                (case-label            . 0)
                (comment-intro         . c-lineup-comment)
                (cpp-define-intro      . +)
                (cpp-macro             . -1000)
                (cpp-macro-cont        . +)
                (defun-block-intro     . +)
                (else-clause           . 0)
                (func-decl-cont        . +)
                (inclass               . +)
                (inher-cont            . c-lineup-multi-inher)
                (knr-argdecl-intro     . 0)
                (label                 . -1000)
                (statement             . 0)
                (statement-block-intro . +)
                (statement-case-intro  . +)
                (statement-cont        . +)
                (substatement          . +)
                ))
        (indent-tabs-mode . t)
        (show-trailing-whitespace . t)
        ))))

(dir-locals-set-directory-class
 (expand-file-name "~/src/linux-trees")
 'linux-kernel)
```

이 설정은 `~/src/linux-tress` 아래에 있는 C 파일들이 커널 코딩 스타일에 잘 맞도록 emacs를 작동하게 해 줄 것입니다.

설령 emacs를 제대로 설정하는 데 실패하더라도 모든 희망이 사라진 것은 아닙니다. `indent`를 사용하십시오.

여기서도 마찬가지로, GNU `indent`는 GNU emacs와 똑같이 멍청한(brain-dead) 기본 설정을 가지고 있으므로 몇 가지 커맨드 라인 옵션(command line options)을 주어야 합니다. 하지만 너무 걱정하지 마십시오. GNU `indent` 제작자들도 K&R의 권위는 인정하고 있으니까요. (GNU 사람들이 약해서가 아니라, 단지 이 문제에 있어서 심하게 오도된 것뿐입니다.) 그저 `indent`에 `-kr -i8`옵션(K&R 스타일, 8자 indent)을 주거나, 최신 스타일로 indent를 맞춰주는 `scripts/Lindet` 스크립트를 사용하면 됩니다.

`indent`에는 수많은 옵션이 있으며, 특히 comment의 포맷을 다시 맞출 때는 `man page`를 살펴보고 싶을 것입니다. 하지만 기억하십시오: **`indent`는 bad programming을 고쳐주는 해결책이 아닙니다.**

참고로, 이러한 규칙들을 적용하기 위해 `clang-format` 도구를 사용할 수도 있습니다. 이 도구는 코드의 일부를 자동으로 빠르게 재포맷하거나, 파일 전체를 검토하여 코딩 스타일 실수, 오타 및 개선 가능한 부분을 찾아내는 데 유용합니다. 또한 `#include`문을 정렬하거나 변수/매크로를 맞추고, 텍스트 흐름을 조정하는 등의 작업에도 편리합니다. 더 자세한 내용은 `Documentation/process/clang-format.rst` 파일을 확인하십시오.

# References
- [Official] Linux Kernel Coding Style 
- [Blog] mythos.log: Linux Tutorial #3 (코딩 스타일 2장)