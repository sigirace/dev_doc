## 🔖 “클래스 변수에 대한 타입 힌트 (Class Variable Type Annotation)”

> **클래스 본문 안에 타입만 선언하고 값을 할당하지 않은 코드**



### ✅ 인스턴스 속성에 대한 타입 힌트 (Instance Attribute Type Annotation)

**📌 정리: 이 코드는 무엇인가?**

| **항목**     | **설명**                                                 |
| ------------ | -------------------------------------------------------- |
| 공식 명칭    | **PEP 526-style Variable Annotation**                    |
| 대상         | 클래스 내부의 **인스턴스 변수에 대한 타입 힌트**         |
| 사용 목적    | 타입 검사기, IDE 지원, 문서화                            |
| 실행 시 의미 | 별도 초기화 코드가 없다면, **런타임에는 아무 영향 없음** |
| 예시 용도    | self.api_key가 어떤 타입인지 외부에 명시적으로 드러냄    |



**🧠 관련 용어 정리**

| **용어**                                  | **설명**                                                     |
| ----------------------------------------- | ------------------------------------------------------------ |
| **PEP 526**                               | Python 3.6에서 변수 주석 문법(type annotation)을 도입한 공식 제안 |
| **type hint**                             | 정적 타입 검사기나 IDE를 위한 힌트이며 런타임 동작과 무관    |
| **variable annotation**                   | x: int처럼 변수에 타입만 명시                                |
| **class attribute vs instance attribute** | 해당 힌트가 실제로 클래스 속성이냐, 인스턴스 속성이냐는 **할당 위치와 생성자 내용**에 따라 다름 |



**🔍 예시 비교**

```python
class A:
    x: int  # 타입 힌트만 있음. 아직 클래스 속성도, 인스턴스 속성도 아님

    def __init__(self):
        self.x = 123  # 이 시점에서 x는 인스턴스 속성

print(A.__annotations__)  # {'x': <class 'int'>}
```

