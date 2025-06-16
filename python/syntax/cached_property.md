## 🔖 “Python 타입 검사기 지원과 런타임 동작을 분리해서 @cached_property를 사용하는 고급 패턴”

> 전체적으로 **cached_property를 타입 검사기에서는 정적으로 해석할 수 있도록 도와주면서**, 실제 런타임에서는 Python 내장 functools.cached_property를 사용하도록 구성되어 있습니다.



### ✅ cached_property란?

- Python 3.8+에서 도입된 **@property의 일종**으로, **값을 한 번만 계산하고 캐싱**하는 프로퍼티입니다.
- 첫 접근 시 메서드를 호출하고, **결과를 저장**해 이후 접근은 메서드 대신 저장된 값을 반환합니다.

```python
from functools import cached_property

class User:
    @cached_property
    def profile(self):
        print("calculating profile...")
        return {"name": "sigi"}

u = User()
print(u.profile)  # 계산됨
print(u.profile)  # 캐시된 값 반환됨
```



### ✅ 이 코드의 목적

- **타입 검사기 (mypy, pyright)가 cached_property를 정확히 이해하지 못할 수 있기에,** TYPE_CHECKING일 때는 **별도의 typed_cached_property 클래스**를 정의해서 정확한 타입 정보를 제공
- **실제 런타임에서는 functools.cached_property를 그대로 사용**



### 🔍 코드 구조 설명

**1. TYPE_CHECKING 분기**

```python
if TYPE_CHECKING:
    cached_property = property  # 타입 검사기에게는 그냥 property로 인식
```

- TYPE_CHECKING은 typing 모듈에서 제공
- mypy나 pyright가 타입 검사할 때만 True
- 런타임에는 False → 실행되지 않음

**2. typed_cached_property 클래스 정의**

```python
class typed_cached_property(Generic[_T]):
    ...
```

- @cached_property처럼 동작하지만,
- 타입 검사기에게는 **정확한 타입 해석**을 제공하기 위한 정의
- \__get\__, \__set\__, \__set_name\__을 정의해서 **데스크립터처럼 동작하는 시그니처를 제공**

**3. else 블럭 (실제 런타임)**

```python
else:
    from functools import cached_property as cached_property
    typed_cached_property = cached_property
```

- 실질적인 실행 환경에서는 **표준 라이브러리의 cached_property를 사용**
- 타입 검사기 외에는 위의 typed_cached_property 정의는 무시됨



### ⭐️ Usecase

```python
@cached_property
def completions(self) -> Completions:
    from .resources.completions import Completions
    return Completions(self)
```

- completions는 첫 호출 시 Completions(self)를 생성하고,
- 이후에는 해당 값을 **캐시하여 재계산 없이 반환**함



### 🧠 요약

| **항목**              | **설명**                                                     |
| --------------------- | ------------------------------------------------------------ |
| @cached_property      | 속성값을 한 번만 계산 후 캐시하는 데코레이터                 |
| TYPE_CHECKING         | 타입 검사기 전용 코드 분기                                   |
| typed_cached_property | 타입 시스템에서 cached_property를 더 잘 해석하게 하는 헬퍼 클래스 |
| 용도                  | **정적 타입 검사 정확성 보장 + 런타임 캐시 기능 제공**       |



---

> 쉬운버전으로 각색



## 1️⃣ @cached_property가 뭔지부터 이해하자



### ✅ 먼저 @property 복습

```python
class User:
    @property
    def name(self):
        print("계산 중...")
        return "신강식"

u = User()
print(u.name)  # "계산 중..." 출력됨
print(u.name)  # 또 "계산 중..." 출력됨
```

- @property는 **메서드를 속성처럼 사용할 수 있게 해주는 것**.
- 하지만 매번 계산됨 → 느릴 수 있음.



### ✅ @cached_property는 뭐가 다를까?

```Python
from functools import cached_property

class User:
    @cached_property
    def name(self):
        print("계산 중...")
        return "신강식"

u = User()
print(u.name)  # 계산됨
print(u.name)  # 캐시된 값 반환됨! 계산 안 됨!
```

- @cached_property는 **처음에만 계산하고**, 그 뒤에는 **값을 저장**해서 재사용합니다.



### 2️⃣ 그럼 너가 본 코드는 왜 복잡하냐?



```python
if TYPE_CHECKING:
    # 타입 검사기(mypy, pyright)가 볼 때만 이 코드를 씀
    cached_property = property

    class typed_cached_property:
        ...
else:
    # 실제 실행 시에는 진짜 cached_property를 씀
    from functools import cached_property
    typed_cached_property = cached_property
```

**🎯 문제 상황**

- 타입 검사기 (mypy, pyright)는 @cached_property를 정확히 이해하지 못할 수 있어요.
- 그래서 다음 같은 일이 생깁니다:

```python
@cached_property
def name(self) -> str:
    ...
print(self.name.upper())  # ← mypy가 여기서 에러낼 수도 있어
```

- 😵 mypy는 self.name이 str이라는 걸 잘 모름
- 왜냐하면 @cached_property가 어떤 건지 **정확히 해석 못해서**!



### ✅ 해결 방법

**1. 타입 검사기만 볼 수 있는 코드**

```python
if TYPE_CHECKING:
    cached_property = property
```

- 타입 검사기가 보면 "아~ 그냥 property구나"라고 이해하도록 속이는 거예요.

**2. 내부적으로는 typed_cached_property라는 클래스를 만들어서**

- 타입 검사기가 __get__이나 __set__ 같은 것까지 **정확히 이해하도록 도와주는 타입 정의**를 합니다.



### ✅ 실제 실행 시에는?

```python
else:
    from functools import cached_property
    typed_cached_property = cached_property
```

- 타입 검사기가 아닌 **실제 코드 실행에서는** functools.cached_property를 그대로 사용합니다.
- 즉, **캐싱 기능은 실제로 잘 동작함!**

