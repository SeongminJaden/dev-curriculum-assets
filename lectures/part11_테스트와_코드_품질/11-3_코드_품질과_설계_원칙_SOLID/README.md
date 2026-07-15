# Part 11. 테스트와 코드 품질

## 11-3. 코드 품질과 설계 원칙(SOLID) — 원칙을 지키는 코드는 왜 변경에 더 강한가

> **학습 목표**
> - SOLID 다섯 원칙 각각의 정확한 정의를 말할 수 있다.
> - 각 원칙을 위반한 코드가 요구사항 변경 앞에서 구체적으로 어떻게 취약해지는지 설명할 수 있다.
> - 원칙을 지키는 코드가 왜 결과적으로 테스트하기도 더 쉬워지는지, 11-1·11-2의 내용과 연결해서 설명할 수 있다.

> **개요** 좋은 코드와 나쁜 코드를 가르는 기준은 "지금 당장 동작하는가"가 아니라 "요구사항이 바뀌었을 때 얼마나 적은 부분만 고치면 되는가"입니다. SOLID는 이 기준을 다섯 개의 구체적인 원칙으로 정리한 것입니다.

---

## 1. SOLID란 무엇인가

> **개요** SOLID라는 이름의 출처와, 다섯 원칙이 공통으로 겨냥하는 목표를 살펴봅니다.

SOLID는 로버트 마틴(Robert C. Martin, 흔히 "Uncle Bob"이라 불림)이 2000년대 초 자신의 논문과 저서에서 정리한 다섯 가지 객체지향 설계 원칙의 앞글자를 딴 이름입니다. 개방-폐쇄 원칙은 버트런드 마이어(Bertrand Meyer)가, 리스코프 치환 원칙은 바바라 리스코프(Barbara Liskov)가 먼저 제시한 개념이지만, 로버트 마틴이 이 둘을 포함한 다섯 원칙을 하나의 체계로 묶어 SOLID라는 이름을 붙였습니다.

- **S** — 단일 책임 원칙 (Single Responsibility Principle)
- **O** — 개방-폐쇄 원칙 (Open-Closed Principle)
- **L** — 리스코프 치환 원칙 (Liskov Substitution Principle)
- **I** — 인터페이스 분리 원칙 (Interface Segregation Principle)
- **D** — 의존성 역전 원칙 (Dependency Inversion Principle)

![SOLID 다섯 원칙 개요 다이어그램](https://raw.githubusercontent.com/SeongminJaden/dev-curriculum-assets/main/lectures/part11_테스트와_코드_품질/11-3_코드_품질과_설계_원칙_SOLID/assets/diagram1_solid_overview.svg)

> **Q.** 다섯 원칙이 다루는 대상이 제각각인데, 이들을 하나로 묶는 공통점은 무엇일까요?
> **A.** 다섯 원칙 모두 <u>**"요구사항 변경이 코드에 미치는 파급 범위를 최소화한다"**</u>는 하나의 목표를 서로 다른 각도에서 겨냥합니다. 단일 책임 원칙은 "한 가지 이유로만 바뀌게 하라"는 관점에서, 개방-폐쇄 원칙은 "기존 코드를 건드리지 말고 확장하라"는 관점에서, 의존성 역전 원칙은 "구체적인 구현이 아니라 추상에 의존하라"는 관점에서 같은 문제를 다룹니다.

---

## 2. 단일 책임 원칙 (SRP)

> **개요** "클래스는 변경의 이유를 단 하나만 가져야 한다"는 원칙을 예시로 확인합니다.

> **원칙** 단일 책임 원칙: <u>**하나의 클래스(또는 모듈)는 변경되어야 할 이유를 오직 하나만 가져야 한다.**</u> 여기서 "책임"은 "이 클래스가 누구를 위해, 어떤 이유로 바뀌는가"를 뜻합니다.

### 2.1 위반 코드 (의사코드)

```
class Report:
    def calculate_total(self, items):
        ...  # 합계 계산 로직

    def save_to_database(self, report):
        ...  # DB 저장 로직

    def format_as_pdf(self, report):
        ...  # PDF 출력 형식 로직
```

> **해석** 이 `Report` 클래스는 세 가지 서로 다른 이유로 바뀔 수 있습니다. 합계 계산 방식이 바뀔 때, 데이터베이스 종류가 바뀔 때, PDF 양식이 바뀔 때 모두 이 클래스를 수정해야 합니다. 문제는 "PDF 양식만 바꾸려던" 담당자가 같은 클래스 안의 계산 로직까지 실수로 건드릴 위험에 노출된다는 점입니다.

### 2.2 준수 코드 (의사코드)

```
class ReportCalculator:
    def calculate_total(self, items):
        ...

class ReportRepository:
    def save_to_database(self, report):
        ...

class ReportPdfFormatter:
    def format_as_pdf(self, report):
        ...
```

> **해석** 계산, 저장, 출력 형식이라는 세 가지 변경 이유를 세 개의 클래스로 분리했습니다. PDF 양식이 바뀌어도 `ReportPdfFormatter`만 수정하면 되고, 이 변경이 계산 로직에 영향을 줄 가능성 자체가 사라집니다.

> **핵심** SRP를 지키면 <u>**하나의 변경이 영향을 미치는 범위가 클래스 하나로 좁혀집니다.**</u> 이는 11-1에서 다룬 단위 테스트 작성에도 직결됩니다. 책임이 하나뿐인 클래스는 테스트할 때 준비해야 할 조건이 적고, 테스트 하나가 실패했을 때 원인을 좁히기도 쉽습니다.

---

## 3. 개방-폐쇄 원칙 (OCP)

> **개요** "확장에는 열려 있고, 변경에는 닫혀 있어야 한다"는 원칙을 결제 수단 추가 예시로 살펴봅니다.

> **원칙** 개방-폐쇄 원칙: <u>**소프트웨어 요소(클래스, 모듈, 함수)는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.**</u> 새로운 기능을 추가할 때 기존에 이미 검증된 코드를 수정하지 않고, 새로운 코드를 덧붙이는 것만으로 확장할 수 있어야 한다는 뜻입니다.

![OCP 위반과 준수 비교 다이어그램](https://raw.githubusercontent.com/SeongminJaden/dev-curriculum-assets/main/lectures/part11_테스트와_코드_품질/11-3_코드_품질과_설계_원칙_SOLID/assets/diagram2_ocp.svg)

### 3.1 위반 코드 (의사코드)

```
class PaymentProcessor:
    def process(self, method, amount):
        if method == "card":
            ...  # 카드 결제 로직
        elif method == "bank_transfer":
            ...  # 계좌이체 로직
        # 새 결제 수단이 생길 때마다 이 함수 내부를 계속 수정해야 함
```

> **해석** "간편 결제"라는 새 결제 수단이 추가될 때마다 이미 동작하고 있던 `process` 함수 내부에 `elif`를 추가해야 합니다. 이 함수를 수정할 때마다 카드 결제, 계좌이체처럼 이미 검증되어 운영 중인 분기까지 함께 재검토 대상이 됩니다.

### 3.2 준수 코드 (의사코드)

```
class PaymentMethod:
    def process(self, amount):
        raise NotImplementedError

class CardPayment(PaymentMethod):
    def process(self, amount):
        ...

class BankTransferPayment(PaymentMethod):
    def process(self, amount):
        ...

class SimplePayPayment(PaymentMethod):  # 새 결제 수단은 클래스 추가만으로 확장
    def process(self, amount):
        ...
```

> **해석** 공통 인터페이스(`PaymentMethod`)를 만들고, 결제 수단마다 별도 클래스로 구현하도록 바꾸면 새 결제 수단은 새 클래스를 하나 추가하는 것으로 끝납니다. 기존의 `CardPayment`, `BankTransferPayment`는 단 한 줄도 건드릴 필요가 없습니다.

> **핵심** OCP의 핵심은 <u>**"이미 테스트를 통과하고 운영 중인 코드를 다시 건드리지 않아도 되게 만드는 것"**</u>입니다. 기존 코드를 건드리지 않으면, 그 코드에 대해 이미 작성된 테스트도 다시 실패할 위험이 없습니다.

---

## 4. 리스코프 치환 원칙 (LSP)

> **개요** "하위 타입은 상위 타입을 대체할 수 있어야 한다"는 원칙을 정사각형-직사각형 문제로 살펴봅니다.

> **원칙** 리스코프 치환 원칙: <u>**상위 타입의 객체를 하위 타입의 객체로 교체해도 프로그램의 정확성이 깨지지 않아야 한다.**</u> 하위 클래스는 상위 클래스가 약속한 동작 규약을 어겨서는 안 됩니다.

### 4.1 위반 코드 (의사코드) — 정사각형은 직사각형인가

```
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h
    def area(self): return self.width * self.height

class Square(Rectangle):
    def set_width(self, w):
        self.width = w
        self.height = w   # 정사각형이므로 높이도 같이 바뀜
    def set_height(self, h):
        self.width = h
        self.height = h
```

> **해석** 수학적으로 정사각형은 직사각형의 한 종류이므로 `Square`가 `Rectangle`을 상속하는 것이 자연스러워 보입니다. 하지만 "너비를 5로, 높이를 10으로 각각 설정하면 넓이는 50이어야 한다"는 `Rectangle`의 규약을 `Square`는 지킬 수 없습니다. `set_height`를 호출하는 순간 너비까지 같이 바뀌기 때문입니다. `Rectangle`을 사용하도록 작성된 코드에 `Square`를 넣으면 예상과 다른 결과가 나옵니다.

### 4.2 준수 방향 (의사코드)

```
class Shape:
    def area(self): raise NotImplementedError

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    def area(self): return self.width * self.height

class Square(Shape):
    def __init__(self, side):
        self.side = side
    def area(self): return self.side * self.side
```

> **해석** `Square`가 `Rectangle`을 상속하는 대신, 둘 다 공통 상위 타입 `Shape`을 각자 독립적으로 구현하도록 바꿨습니다. 이제 `Square`는 `Rectangle`이 지켜야 할 "너비와 높이를 독립적으로 바꿀 수 있다"는 규약을 애초에 약속하지 않으므로, 규약 위반 자체가 발생하지 않습니다.

> **핵심** LSP를 위반하면 <u>**"타입만 보고 안심하고 코드를 재사용할 수 없게" 됩니다.**</u> 상위 타입을 다루는 코드를 작성할 때마다 "혹시 이 하위 타입에서는 다르게 동작하지 않을까"를 매번 의심해야 한다면, 상속 구조 자체가 코드를 이해하기 더 어렵게 만드는 셈입니다.

---

## 5. 인터페이스 분리 원칙 (ISP)

> **개요** "클라이언트는 자신이 사용하지 않는 메서드에 의존하도록 강요받아서는 안 된다"는 원칙을 다중기능 인터페이스 예시로 살펴봅니다.

> **원칙** 인터페이스 분리 원칙: <u>**하나의 거대한 인터페이스보다, 클라이언트별로 필요한 만큼만 나눈 여러 개의 작은 인터페이스가 낫다.**</u>

### 5.1 위반 코드 (의사코드)

```
class Worker:
    def work(self): raise NotImplementedError
    def eat(self): raise NotImplementedError
    def sleep(self): raise NotImplementedError

class RobotWorker(Worker):
    def work(self): ...
    def eat(self):
        raise Exception("로봇은 식사하지 않음")   # 억지로 구현해야 함
    def sleep(self):
        raise Exception("로봇은 수면하지 않음")   # 억지로 구현해야 함
```

> **해석** `Worker` 인터페이스가 "일하기", "식사하기", "수면하기"를 모두 강제하기 때문에, 이 세 가지 중 일부만 해당하는 `RobotWorker`도 나머지 메서드를 억지로 구현해야 합니다. 이렇게 사용하지도 않을 메서드를 예외 처리로 채워 넣는 코드가 늘어나면, 인터페이스만 보고는 이 클래스가 실제로 무엇을 하는지 파악하기 어려워집니다.

### 5.2 준수 코드 (의사코드)

```
class Workable:
    def work(self): raise NotImplementedError

class Eatable:
    def eat(self): raise NotImplementedError

class Sleepable:
    def sleep(self): raise NotImplementedError

class RobotWorker(Workable):
    def work(self): ...

class HumanWorker(Workable, Eatable, Sleepable):
    def work(self): ...
    def eat(self): ...
    def sleep(self): ...
```

> **해석** 인터페이스를 기능 단위로 잘게 나누면, `RobotWorker`는 자신에게 실제로 필요한 `Workable`만 구현하면 됩니다. 억지 구현이나 예외 처리가 사라지고, 어떤 클래스가 `Eatable`을 구현했는지만 봐도 그 클래스가 무엇을 할 수 있는지 정확히 드러납니다.

> **핵심** ISP는 <u>**"인터페이스가 크면 클수록 그 인터페이스에 의존하는 모든 클라이언트가 불필요한 부분까지 떠안게 된다"**</u>는 문제를 지적합니다. 인터페이스를 작게 나누면 변경의 영향 범위도 그만큼 좁아집니다.

---

## 6. 의존성 역전 원칙 (DIP)

> **개요** "상위 모듈은 하위 모듈에 의존해서는 안 되며, 둘 다 추상에 의존해야 한다"는 원칙을 11-2의 의존성 주입과 연결해서 살펴봅니다.

> **원칙** 의존성 역전 원칙: <u>**상위 수준 모듈은 하위 수준 모듈에 의존해서는 안 되며, 둘 다 추상(인터페이스)에 의존해야 한다. 추상은 세부 구현에 의존해서는 안 되며, 세부 구현이 추상에 의존해야 한다.**</u>

### 6.1 위반 코드 (의사코드)

```
class MySQLDatabase:
    def save(self, data): ...

class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()   # 구체적인 구현에 직접 의존
    def place_order(self, order):
        self.db.save(order)
```

> **해석** 상위 수준의 정책을 다루는 `OrderService`가, 하위 수준의 세부 구현인 `MySQLDatabase`를 직접 알고 있습니다. 나중에 데이터베이스를 PostgreSQL로 바꾸거나 테스트용 가짜 저장소로 바꾸려면 `OrderService`의 코드 자체를 수정해야 합니다.

### 6.2 준수 코드 (의사코드)

```
class Database:                       # 추상(인터페이스)
    def save(self, data): raise NotImplementedError

class MySQLDatabase(Database):
    def save(self, data): ...

class OrderService:
    def __init__(self, db: Database):  # 추상에 의존
        self.db = db
    def place_order(self, order):
        self.db.save(order)
```

> **해석** `OrderService`는 이제 `Database`라는 추상에만 의존하고, 그 추상을 실제로 구현한 것이 `MySQLDatabase`인지 테스트용 가짜 객체인지는 알 필요가 없습니다. 이 구조가 바로 11-2에서 다룬 의존성 주입이 실현하는 목표이며, DIP는 그 목표를 원칙 수준에서 정의한 것입니다.

> **참고** OCP가 "확장 시 기존 코드를 건드리지 않는다"에 초점을 둔다면, DIP는 <u>**"상위 정책과 하위 구현 사이의 방향을 뒤집어, 둘 다 추상을 바라보게 한다"**</u>는 더 구조적인 해법을 제공합니다. 두 원칙은 서로 다른 각도에서 같은 문제(구체적인 것에 얽매이지 않기)를 해결합니다.

> **핵심** DIP를 지키면 <u>**하위 구현을 교체해도 상위 모듈은 전혀 알아채지 못합니다.**</u> 이는 11-1에서 다룬 통합 테스트·단위 테스트를 가르는 기준, 즉 "실제 의존성을 얼마나 걷어낼 수 있는가"와 정확히 같은 문제입니다.

---

## 7. 실생활 예시 — SOLID 위반 코드가 요구사항 변경 앞에서 취약해지는 과정

> **개요** 다섯 원칙을 모두 위반한 하나의 결제 시스템이 새로운 결제 수단 추가라는 흔한 요구사항 변경 앞에서 어떻게 무너지는지 정리합니다.

| 위반 원칙 | "새로운 결제 수단(간편 결제)을 추가해 달라"는 요청 앞에서 벌어지는 일 |
|---|---|
| SRP 위반 | 결제 로직과 DB 저장, 알림 발송이 한 클래스에 섞여 있어, 결제 로직만 고치려 해도 클래스 전체를 다시 검토해야 함 |
| OCP 위반 | 결제 수단이 `if-elif`로 나열되어 있어, 새 수단을 추가할 때마다 기존에 검증된 카드·계좌이체 분기까지 함께 있는 함수를 수정해야 함 |
| LSP 위반 | 특정 결제 수단 클래스가 상위 타입의 규약을 지키지 않아, 공통 코드에서 결제 수단을 교체할 때 예상치 못한 오류가 발생 |
| ISP 위반 | 모든 결제 수단이 "환불", "정기 결제 등록" 같은 자신과 무관한 메서드까지 구현해야 해서, 새 결제 수단을 추가할 때 불필요한 코드를 계속 채워 넣어야 함 |
| DIP 위반 | 결제 서비스가 특정 PG사 SDK에 직접 의존하고 있어, 새 PG사로 확장하려면 서비스 코드 자체를 뜯어고쳐야 함 |

> **핵심** 다섯 원칙은 각각 독립적인 규칙이 아니라, <u>**"이 코드가 다음 요구사항 변경에도 견딜 수 있는가"라는 하나의 질문을 다섯 개의 구체적인 체크리스트로 나눈 것**</u>입니다. 원칙을 지키는 코드는 변경 자체를 막지 않습니다. 다만 변경이 필요한 부분만 정확히 좁혀서, 이미 검증된 나머지 코드를 다시 건드리지 않게 해줍니다.

---

## 요약

> **핵심**
> - SOLID는 로버트 마틴이 정리한 다섯 가지 객체지향 설계 원칙(단일 책임, 개방-폐쇄, 리스코프 치환, 인터페이스 분리, 의존성 역전)의 앞글자를 딴 이름입니다.
> - 다섯 원칙은 모두 <u>**"요구사항 변경이 코드에 미치는 파급 범위를 최소화한다"**</u>는 하나의 목표를 서로 다른 각도에서 겨냥합니다.
> - 원칙을 위반한 코드는 작은 요구사항 변경에도 이미 검증된 코드 전체를 다시 건드려야 하지만, 원칙을 지키는 코드는 <u>**변경이 필요한 범위를 정확히 좁혀 이미 검증된 부분을 그대로 보존합니다.**</u>
