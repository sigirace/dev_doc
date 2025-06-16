## 🎯 왜 @overload를 쓸까?



### 🧩 배경: Python의 특성과 한계

**✅ Python은 “런타임 다형성”을 지원**

- Python은 동적 언어입니다. 즉, 아래 함수는 add(1, 2)도 되고, add("a", "b")도 됩니다.

```python
def add(x, y):
    return x + y
```

- 👉 **런타임에 동작이 결정**되고, 반환 타입은 상황에 따라 달라질 수 있습니다.



**❌ 하지만 “정적 타입 시스템”에서는 문제 발생**

- 정적 타입 검사기(mypy, pyright)는 **코드 실행 없이 코드만 보고 판단**해야 합니다.

```python
def request(stream: bool) -> str | Generator[str, None, None]:
    if stream:
        return (s for s in ["hello", "world"])  # generator
    return "hello"
```

- 👉 정적 분석기는 request()의 반환 타입을 정확하게 **조건별로** 파악하기 어렵습니다.
  - stream=True일 때 → Generator
  - stream=False일 때 → str
- 이런 조건별 타입을 명확하게 **표현**하는 방법이 바로 @overload입니다.



### ✅ @overload가 해결하는 것

```Python
from typing import overload, Generator

@overload
def request(stream: Literal[False]) -> str: ...
@overload
def request(stream: Literal[True]) -> Generator[str, None, None]: ...

def request(stream: bool):
    if stream:
        return (s for s in ["hello", "world"])
    return "hello"
```

- request(False)라고 하면 정적 분석기(mypy, Pyright)는 **반환 타입이 str이라는 걸 정확히 알게 됨**
- request(True)는 Generator 반환



**🔍 실제 효과 예시 (mypy / IDE 인식)**

```Python
s = request(False)
reveal_type(s)  # Revealed type is "builtins.str"

g = request(True)
reveal_type(g)  # Revealed type is "Generator[str, None, None]"
```

- IDE 자동완성도 그에 따라 .split()을 제안하거나, .send()를 제안하게 됨
- 📌 즉, **IDE 자동완성과 타입 추론이 정확하게 작동**함



### 🔒 핵심 요약

| **구분**         | **설명**                                                |
| ---------------- | ------------------------------------------------------- |
| Python 런타임    | 함수 하나가 다양한 타입을 반환 가능                     |
| 정적 타입 분석기 | 조건에 따라 타입을 다르게 해석하지 못함                 |
| @overload        | 조건별 타입을 명시해 정적 분석기가 올바르게 추론하게 함 |
| 혜택             | IDE 자동완성 향상, 타입 안전성 강화, 버그 예방          |



### ⭐️ Usecase

| **상황**                                                | @overload **필요 여부** |
| ------------------------------------------------------- | ----------------------- |
| 입력 파라미터에 따라 반환 타입이 바뀜                   | ✅ 반드시 필요           |
| 내부 구현 하나로 다양한 반환을 할 때                    | ✅ 필요                  |
| API SDK나 Wrapper 등에서 타입 안정성을 강화하고 싶을 때 | ✅ 적극 권장             |

