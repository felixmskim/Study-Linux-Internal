# 10) Kconfig configuration files
모든 Kconfig configuration 파일들의 indentation은 다소 다릅니다. `config` 정의 아래의 line들은 한 개의 `tab`으로 들여씁니다. 반면, `help` 텍스트는 추가로 `space` 2개를 더하여 들여씁니다.

```bash
config AUDIT
        bool "Auditing support"
        depends on NET
        help
          Enable auditing infrastructure that can be used with another
          kernel subsystem, such as SELinux (which requires this for
          logging of avc messages).
```
위험한 기능(예: 특정 파일 시스템에 대한 쓰기 지원)은 해당 configuration 옵션의 prompt 문자열에 명확하게 **(EXPERIMENTAL)** 또는 **(DANGEROUS)** 라고 표기해야 합니다.

# 11) Data structures
자체적인 참조 카운트(reference count)가 없는 구조체를 가리키는 포인터가 해당 구조체가 할당 해제될 수 있는 single-threaded 외부 환경에 공개된다면, 이는 항상 버그입니다. 리눅스 커널은 참조 카운팅을 위해 refcount_t를 사용하며, 이를 적극적으로 활용해야 합니다.

모든 데이터 구조에는 이를 보호하기 위한 locking 전략이 있어야 합니다. 코드를 읽는 사람이 이 구조체가 어떤 락에 의해 보호되는지 알 수 있도록 주석을 달거나, 구조체 멤버 이름에 명시하십시오.

# 12) Macros, Enums and RTL (Runtime Library)
상수를 정의하는 macro 이름과 `enum`의 열거형 이름은 모두 대문자(CAPS)여야 합니다.

```C
#define CONSTANT 0x12345
```

서로 연관된 여러 상수를 정의할 때는 `enum`을 사용하는 것이 좋습니다.

Function처럼 동작하는 매크로의 이름은 소문자여야 합니다. 일반적으로 매크로보다는 inline function이 선호됩니다.

여러 개의 statement를 포함하는 매크로는 `do { ... } while (0)` 블록으로 감싸야 합니다.

```C
#define macrofun(a, b, c)			
        do {					
                if (a == 5)			
                        do_this(b, c);		
        } while (0)
```

매크로를 사용할 때 피해야 할 사항들은 다음과 같습니다:

1. 제어 흐름(control flow)에 영향을 주는 매크로:

```C
#define FOO(x)					
        do {					
                if (blah(x))			
                        return 1;		
        } while (0)
```

이것은 매우 나쁜 아이디어입니다. 호출하는 쪽에서 return이 발생한다는 사실을 알기 어렵게 만듭니다.

2. Local variable에 의존하는 매크로:

```C
#define FOO(val) bar(index, val)
```

이 매크로는 호출되는 문맥에 `index`라는 변수가 반드시 있어야 하므로 가독성을 해칩니다.

3. Parameter를 l-value로 사용하는 매크로: `FOO(x) = y;` 와 같은 방식은 매크로가 struct인 것처럼 오해를 불러일으킬 수 있습니다.

4. 연산 우선순위 문제: 매크로의 모든 argument는 괄호로 감싸야 합니다.

```C
#define CONSTANT 0x4000
#define ADDR (CONSTANT | 0x0001)
```

# 13) Printing kernel messages

커널 개발자들은 메시지가 가독성 있게 출력되기를 바랍니다. 메시지는 가급적 문장 형태로 작성하며, 마침표(period)로 끝내지 마십시오. "메모리 할당 실패" 보다는 "failed to allocate memory"가 낫습니다.

printk를 직접 쓰기보다는 `pr_info()`, `pr_debug()`, `pr_err()` 등과 같은 helper 함수를 사용하십시오. 특히 장치 드라이버 내에서는 `dev_info()`, `dev_err()` 등을 사용하여 메시지가 어떤 장치에서 발생했는지 명확히 해야 합니다.

# 14) Allocating memory

커널은 메모리를 할당하는 다양한 방법을 제공합니다. 구조체의 크기를 계산할 때는 항상 다음과 같은 형식을 권장합니다.

```C
p = kmalloc(sizeof(*p), ...);
```

구조체의 이름(`sizeof(struct name)`)을 직접 쓰는 것은 가독성을 해칠 뿐만 아니라, 나중에 포인터 `p`의 타입이 변경되었을 때 버그를 유발할 수 있습니다.

`void` 포인터인 할당 결과값을 명시적으로 형변환(casting)하는 것은 C 언어에서 불필요하며 에러를 숨길 수 있으므로 하지 마십시오.

15) The inline disease
`inline` 키워드는 컴파일러에게 해당 함수를 호출 지점에 직접 삽입하라고 제안하는 것이지만, 과용하면 안 됩니다. `inline`은 함수가 매우 짧고(3줄 이하) 성능에 아주 민감한 경우에만 사용하십시오. 함수가 커지면 커널 이미지 크기만 늘리고 성능상 이점은 사라집니다.

16) Function return values and names
함수의 반환값은 그 의미가 명확해야 합니다.

1. **성공/실패 여부:** 0은 성공을 의미하며, 음수 에러 코드(예: -EFAULT, -ENOMEM)는 실패를 의미합니다.

2. **Boolean:** 함수 이름이 질문 형태(예: is_valid())라면 1(true) 또는 0(false)을 반환해야 합니다.

# 17) Don't re-invent the kernel macros
커널 헤더 파일 `include/linux/kernel.h` 등에는 이미 검증된 수많은 매크로가 있습니다. 배열의 크기를 구할 때 직접 계산하지 말고 `ARRAY_SIZE()` 매크로를 사용하십시오. 구조체의 멤버 주소로 구조체 본체의 주소를 찾을 때는 `container_of()`를 사용하십시오.

# 18) Editor modelines and other cruft
소스 파일에 에디터 설정(Vim의 modeline이나 Emacs의 Local Variables)을 넣지 마십시오. 이는 다른 사람의 에디터 설정을 망칠 수 있습니다.

# 19) Inline assembly
인라인 어셈블리를 사용할 때는 C 함수로 감싸서 추상화하십시오. 어셈블리 코드 내에서도 C 언어처럼 가독성을 위해 적절한 주석을 달아야 합니다.

# 20) Conditional compilation
가급적 소스 코드(.c 파일) 내에서 `#ifdef`를 사용하는 것을 피하십시오. 대신 헤더 파일(.h 파일)에서 빈 함수(stub)를 정의하거나, `if (IS_ENABLED(CONFIG_FOO))` 형식을 사용하십시오. 컴파일러가 최적화 과정에서 실행되지 않는 코드를 제거해 줄 것입니다.

```C
// Bad
#ifdef CONFIG_QUOTA
	    do_quota_stuff();
#endif

// Better
if (IS_ENABLED(CONFIG_QUOTA))
	    do_quota_stuff();
```

# 21) Conclusion
이것으로 리눅스 커널 코딩 스타일 가이드를 마칩니다. 다시 한번 강조하지만, 이 가이드의 목적은 **가독성**과 **유지보수성**입니다. 이 규칙들을 숙지하고 적용한다면 여러분의 코드는 커널 커뮤니티에서 더 환영받을 것입니다.

# References
- [Official] Linux Kernel Coding Style 
- [Blog] mythos.log: Linux Tutorial #4 (코딩 스타일 3장)