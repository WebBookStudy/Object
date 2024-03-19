# 📕 [Object] 10장. 상속과 코드 재사용

# 상속과 중복 코드
## DRY 원칙
**중복 코드는 변경을 방해한다.**
이것이 중복 코드를 제거해야 하는 가장 큰 이유다. 중복 코드가 가지는 가장 큰 문제는 코드를 수정하는 데 필요한 노력을 몇 배로 증가 시킨다는 것이다. 중복 여부를 판단하는 기준은 **변경**이다. **요구사항이 변경 됐을 때 두 코드를 함께 수정**해야 한다면 이 코드는 중복이다.

**DRY 원칙**
DRY는 '반복하지 마라'라는 뜻의 Don't Repeat Yourself의 첫 글자를 모아 만든 용어로 간단히 말해 **동일한 지식을 중복하지 말라는 것**이다.
DRY 원칙은 한번, 단 한번(Once and Only One) 원칙 또는 단일 지점 제어(Single-Point Control) 원칙이라고도 부른다.

## 중복과 변경
상속을 이용해 코드를 재사용하기 위해서는 **부모 클래스의 개발자가 세웠던 가정이나 추론 과정을 정확하게 이해**해야 한다. 이것은 자식 클래스의 작성자가 부모 클래스의 구현 방법에 대한 정확한 지식을 가져야 한다는 것을 의미한다.
따라서 **상속은 결합도를 높인다**고 이야기할 수 있다.

## 강하게 결합된 Phone과 NightlyDiscountPhone
부모 클래스와 자식 클래스 사이의 결합이 문제인 이유를 살펴보자.

`NightlyDiscountPhone`은 부모 클래스인 `Phone`의 calculateFee 메서드를 오버라이딩한다. 또한 메서드 안에서 super 참조를 이용해 부모 클래스의 메서드를 호출한다. NightlyDiscountPhone의 calculateFee 메서드는 자신이 오버라이딩한 Phone의 calculateFee 메서드가 모든 통화에 대한 요금의 총합을 반환한다는 사실에 기반하고 있다.

하지만 세금을 부과하는 요구사항이 추가된다면 어떻게 될까? calculateFee 메서드에서 값을 반환할 때 taxRate를 이용해 세금을 부과해야 한다.

```java
public class Phone {
    private double taxRate; // 세금 부과 요구사항으로 인한 변수 추가
    public Phone(Money amount, Duration seconds, double taxRate) {
        ...
        this.taxTate = taxRate;
    }

    public calculateFee() {
        ...
        return result.plus(result.times(taxRate));
    }

    public double getTaxRate() {
        return this.taxRate;
    }
}
```
NigthlyDiscountPhone은 생성자에서 전달받은 taxRate를 부모 클래스인 Phone의 생성자로 전달해야 한다. 또한 Phone과 동일하게 값을 반환할 때 taxRate를 이용해 세금을 부과해야 한다.

```java
public class NightlyDiscountPhone extends PHone {
    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        super(regularAmount, seconds, taxRate);
        ...
    }

    @Override
    public Money calculateFee() {
        ...
        return result.minus(nightlyFee.plus(nightlyFee.times(getTaxRate()));
    }
}
```

NightlyDiscountPhone을 Phone의 자식 클래스로 만든 이유는 **Phone의 코드를 재사용하고 중복 코드를 제거**하기 위해서다. 하지만 세금을 부과하는 로직을 추가하기 위해 Phone을 수정할 때 유사한 코드를 NightlyDiscountPhone에도 추가해야 했다.
> 다시 말해서 **코드 중복을 제거하기 위해 상속을 사용했음에도 세금을 계산하는 로직을 추가하기 위해 새로운 중복 코드를 만들어야 하는 것**이다.

이것은 NightlyDiscountPhone이 Phone의 구현에 너무 강하게 결합돼 있기 때문에 발생하는 문제다. 이처럼 상속 관계로 연결된 자식 클래스가 부모 클래스의 변경에 취약해지는 현상을 가리켜 **취약한 기반 클래스 문제**라고 부른다.

# 취약한 기반 클래스 문제
부모 클래스의 작은 변경에도 자식 클래스는 컴파일 오류와 실행 에러라는 고통에 시달려야 할 수도 있다. 구현을 상속한 경우(extends를 사용한 경우) 파생 클래스는 기반 클래스에 강하게 결합되며, 이 둘 사이의 밀접한 연결은 바람직하지 않다.

상속은 자식 클래스를 점진적으로 추가해서 기능을 확장하는 데는 용이하지만 높은 결합도로 인해 부모 클래스를 점진적으로 개선하는 것은 어렵게 만든다. 취약한 기반 클래스 문제는 캡슐화를 약화시키고 결합도를 높인다.

객체를 사용하는 이유는 구현과 관련된 세부사항을 퍼블릭 인터페이스 뒤로 캡슐화할 수 있기 때문이다. 캡슐화는 변경에 의한 파급효과를 제어할 수 있기 때문에 가치가 있다. 객체는 변경 될지도 모르는 불안정한 요소를 캡슐화함으로써 파급효과를 걱정하지 않고도 자유롭게 내부를 변경할 수 있다.

상속은 코드의 재사용을 위해 캡슐화의 장점을 희석시키고 구현에 대한 결합도를 높임으로써 객체지향이 가진 강력함을 반감시킨다.

## 불필요한 인터페이스 상속 문제
java.util.Properties와 java.util.Stack은 대표적인 잘못된 상속의 예이다.
![](https://velog.velcdn.com/images/pp8817/post/cc0f40dd-265b-49b7-b932-feb2bc043cd2/image.png)
자바의 초기 컬렉션 프레임워크 개발자들은 요소의 추가, 삭제 오퍼레이션을 제공하는 Vector를 재사용하기 위해 Stack을 Vector의 자식 클래스로 구현했다.

Vector는 임의의 위치(index)에서 요소를 조회하고, 추가하고, 삭제할 수 있는 get, add, remove 오퍼레이션을 제공한다. 이에 비해 Stack은 맨 마지막 위치 에서만 요소를 추가하거나 제거할 수 있는 push, pop 오퍼레이션을 제공한다.

```java
Stack<String> stack = new Stack<>();
stack.push("1st");
stack.push("2nd");
stack.push("3rd");

stack.add(0, "4th");
assertEquals("4th", stack.pop()); // 에러!
```

위 코드에서 Stack에 마지막으로 추가한 값은 "4th"지만 pop 메서드의 반환값은 "3rd"다. 그 이유는 Vector의 add 메서드를 이용해서 스택의 맨 앞에 "4th"를 추가했기 때문이다.

## 메서드 오버라이딩의 오작용 문제
조슈아 블로치는 클래스가 상속되기를 원한다면 상속을 위해 클래스를 설계하고 문서화해야 하며, 그렇지 않은 경우에는 상속을 금지시켜야 한다고 주장한다.

> 그러나 잘된 API 문서는 메서드가 무슨 일(what)을 하는지를 기술해야 하고, 어떻게 하는지(how)를 설명해서는 안 된다는 통념을 어기는 것은 아닐까?
어기는 것이다. 이 또한 상속이 캡슐화를 위반함으로써 초래된 불행인 것이다.

설계는 트레이드오프 활동이라는 사실을 기억해야 한다. 상속은 코드 재사용을 위해 캡슐화를 희생한다. 완벽한 캡슐화를 원한다면 코드 재사용을 포기하거나 상속 이외의 다른 방법을 사용해야 한다.

# Phone 다시 살펴보기
## 추상화에 의존하자
이 문제를 해결하는 가장 일반적인 방법은 자식 클래스가 부모 클래스의 구현이 아닌 추상화에 의존하도록 만드는 것이다. 정확하게 말하면 부모 클래스와 자식 클래스 모두 추상화에 의존하도록 수정해야 한다. 개인적으로 코드 중복을 제거하기 위해 상속을 도입할 때 따르는 두 가지 원칙이 있다.

- 두 메서드가 유사하게 보인다면 차이점을 메서드로 추출하라.
    - 메서드 추출을 통해 두 메서드를 동일한 형태로 보이도록 만들 수 있다.

- 부모 클래스의 코드를 하위로 내리지 말고 자식 클래스의 코드를 상위로 올려라.
    - 부모 클래스의 구체적인 메서드를 자식 클래스로 내리는 것보다 자식 클래스의 추상적인 메서드를 부모 클래스로 올리는 것이 재사용성과 응집도 측면에서 더 뛰어난 결과를 얻을 수 있다.

# 중복 코드를 부모 클래스로 올려라
부모 클래스를 추가하자. 목표는 모든 클래스들이 추상화에 의존하도록 만드는 것이기 때문에 이 클래스는 추상 클래스로 구현하는 것이 적합할 것이다.
![](https://velog.velcdn.com/images/pp8817/post/5df0964b-e3dc-4e66-b689-cc00bf526723/image.png)

>모든 하위 클래스가 이 행동을 할 수 있게 만들려면 여러 개의 중복 코드를 양산하거나 이 행동을 상위 클래스로 올리는 수밖에 없다.

## 차이에 의한 프로그래밍
시간이 흐르고 객체지향에 대한 이해가 깊어지면서 사람들은 코드를 재사용하기 위해 맹목적으로 상속을 사용하는 것이 위험하다는 사실을 깨닫기 시작했다. 상속이 코드 재사용이라는 측면에서 매우 강력한 도구인 것은 사실이지만 강력한 만큼 잘못 사용할 경우에 돌아오는 피해 역시 크다는 사실을 뼈저리게 경험한 것이다.
>상속의 오용과 남용은 애플리케이션을 이해하고 확장하기 어렵게 만든다.

- - -
**출처** <br>
[오브젝트 - 코드로 이해하는 객체지향 설계](https://smartstore.naver.com/aladinstores/products/7815204109?NaPm=ct%3Dlsvyemdc%7Cci%3Da772aefc1b71a76e7f3412247dc9108aec75e0f6%7Ctr%3Dboksl%7Csn%3D4399901%7Chk%3D0c624b3c7c58fc3c0cc29d777797044a631fb294)
https://github.com/eternity-oop/object
