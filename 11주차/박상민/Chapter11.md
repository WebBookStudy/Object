# 📕 [Object] 11장. 합성과 유연한 설계

상속 관계는 `is-a` 관계라고 부르고 합성 관계는 `has-a` 관계라고 부른다.

상속은 부모 클래스 안에 구현된 코드 자체를 재사용 하지만 합성은 포함되는 객체의 퍼블릭 인터페이스를 재사용한다. 따라서 상속 대신 합성을 사용하면 구현에 대한 의존성을 인터페이스에 대한 의존성으로 변경할 수 있다. 다시 말해서 클래스 사이의 높은 결합도를 객체 사이의 낮은 결합도로 대체할 수 있는 것이다.

상속을 받으면 부모 클래스의 내부가 자식 클래스에 공개 되기 때문에 화이트박스 재사용으로 부른다. 합성은 객체의 내부는 공개 되지 않고 인터페이스를 통해서만 재사용 되기 때문에 블랙박스 재사용으로 부른다.

# ⭐️ 상속을 합성으로 변경하기
이전에 코드 재사용을 위해 상속을 남용 했을 때 직면할 수 있는 세 가지 문제점을 살펴봤다.

- 불필요한 인터페이스 상속 문제 : java.util.Properties와 java.util.Stack

- 메서드 오버라이딩의 오작용 문제 : java.util.HashSet을 상속받은 InstumentedHashSet

- 부모 클래스와 자식 클래스의 동시 수정 문제 : Playlist를 상속받은 PersonalPlaylist

## 📌 상속으로 인한 조합의 폭발적인 증가
상속으로 인해 결합도가 높아지면 다음과 같은 두 가지 문제점이 발생한다.

- 하나의 기능을 추가하거나 수정하기 위해 불필요하게 많은 수의 클래스를 추가하거나 수정해야 한다.
- 단일 상속만 지원하는 언어에서는 상속으로 인해 오히려 중복 코드의 양이 늘어날 수 있다.

이처럼 상속의 남용으로 하나의 기능을 추가하기 위해 필요 이상으로 많은 수의 클래스를 추가해야 하는 경우를 가리켜 클래스 폭발(class explosion) 문제 또는 조합의 폭발(combinational explosion) 문제라고 부른다.

상속 관계는 컴파일 타임에 결정되고 고정되기 때문에 코드를 실행하는 도중에는 변경할 수 없다. 따라서 여러 기능을 조합해야 하는 설계에 상속을 이용하면 모든 조합 가능한 경우 별로 클래스를 추가 해야 한다.

## 📌 합성 관계로 변경하기
클래스 폭발 문제를 해결하기 위해 합성을 사용하는 이유는 런타임에 객체 사이의 의존성을 자유롭게 변경할 수 있기 때문이다. 합성을 사용하면 구현 시점에 정책들의 관계를 고정시킬 필요가 없으며 실행 시점에 정책들의 관계를 유연하게 변경할 수 있게 된다.

Phone 클래스를 생성한다.
```java
public class Phone {
    private RatePolicy ratepolicy;
    private List<Call> calls = new ArrayList<>();

    public Phone(RatePolicy ratePolicy) {
        this.ratePolicy = ratePolicy;
    }

    public List<Call> getCalls() {
        return Collections.unmodifiableList(calls);
    }

    public Money calculateFee() {
        return ratePolicy.calculateFee(this);
    }
}
```
Phone 클래스에서 사용하는 RatePolicy 타입을 인터페이스로 정의한다.

```java
public interface RatePolicy {
    Money calculateFee(Phone phone);
}
```
RatePolicy를 구현한 추상 클래스를 생성한다.

```java
public abstract class BasicRatePolicy implements RatePolicy {
    @Override
    public Money calculateFee(Phone phone) {
        Money result = Money.ZERO;

        for(Call call : phone.getCalls()) {
            result.plus(calculateCallFee(call));
        }

        return result;
    }

    abstract protected Money calculateCallFee(Call call);
}
```
추상 클래스를 상속받은 구현 클래스를 정의한다.

```java
public class RegularPolicy extends BasicRatePolicy {
    private Money amout;
    private Duration seconds;

    public RegularPolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```
이번엔 부가 연산을 구현할 추상클래스를 생성해보자

```java
public abstract class AdditionalRatePolicy implements RatePolicy {
    private RatePolicy next;

    public AdditionalRatePolicy(RatePolicy next) {
        this.next = next;
    }

    @Override
    public Money calculateFee(Phone phone) {
        Money fee = next.calculateFee(phone);

        return afterCalculated(fee);
    }

    abstract protected Money afterCalculated(Call call);
}
```
실제로 클라이언트에서는 런타임시에 다양한 조합이 가능해진다.

```java
Phone phone = new Phone( new TexablePolicy(0.05,
                           new RateDiscountablePolicy(Money.wons(1000),
                             new RegularPolicy(...)));
```
# ⭐️ 믹스인
믹스인(mixin)은 객체를 생성할 때 코드 일부를 클래스 안에 섞어 넣어 재사용하는 기법을 가리키는 용어다. 합성이 실행 시점에 객체를 조합하는 재사용 방법이라면 믹스인은 컴파일 시점에 필요한 코드 조각을 조합하는 재사용 방법이다.

## 📌 믹스인은 상속과 다르다.
여기까지 설명을 듣고 나면 믹스인과 상속이 유사한 것처럼 보이겠지만 믹스인은 상속과는 다르다. 상속의 진정한 목적은 자식 클래스를 부모 클래스와 동일한 개념적인 범주로 묶어 is-a 관계를 만들기 위한 것이다. 반면 믹스인은 말 그대로 코드를 다른 코드 안에 섞어 넣기 위한 방법이다.


- - -
**출처**<br>
[오브젝트 - 코드로 이해하는 객체지향 설계](https://smartstore.naver.com/aladinstores/products/7815204109?NaPm=ct%3Dlsvyemdc%7Cci%3Da772aefc1b71a76e7f3412247dc9108aec75e0f6%7Ctr%3Dboksl%7Csn%3D4399901%7Chk%3D0c624b3c7c58fc3c0cc29d777797044a631fb294)
https://github.com/eternity-oop/object