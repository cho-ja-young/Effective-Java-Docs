---
sidebar: auto
lang: ko-KR
title: effective-java-chapter3
---

# 3장 모든 객체의 공통 메서드

## ITEM 14. Comparable을 구현할지 고려하라

### compareTo 메서드

* Comparable 인터페이스의 유일무이한 메서드

* Object의 메서드가 아니다. Object의 equals와 두 가지의 성격 차이가 있다. 

* 단순 동치성 비교 + 순서까지 비교 + 제네릭

* 'Comparable을 구현했다'  ==  '그 클래스의 인스턴스들에게는 자연적인 순서(natural order)가 있다'

> Array.sort(a);

> 검색, 극단값 계산, 자동 정렬 컬렉션 관리 또한 쉽게 가능

```
// 명령줄 인수들을 중복 제거 후, 알파벳순으로 출력
// String 이 Comparable 을 구현한 덕분

public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```
* 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입(아이템34)이 Comparable을 구현 

* 알파벳, 숫자. 연대 값이 순서가 명확한 값 클래스를 작성한다면, 반드시 Comparable 인터페이스를 구현하라

```
public interface Comparable<T> {
    int compareTo(T t);
}
```

<br>

### compareTo 메서드의 일반 규약
---
#### 이 객체와 주어진 객체의 순서를 비교
- 이 객체 < 주어진 객체 : 음의 정수로 반환
- 이 객체 == 주어진 객체 : 0으로 반환
- 이 객체 > 주어진 객체 : 양의 정수로 반환한다
- 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException 을 던진다.

> 다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, <br>
> 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의

- equals 메서드 : 모든 객체에 대해 전역 동치관계를 부여

- compareTo 메서드 : 타입이 다른 객체는 신경쓰지 않음 -> `ClassCastException` 반환

- 단, 아래의 규약에서는 다른 타입 사이의 비교도 허용
-> 비교할 객체들이 구현한 공통 인터페이스를 매개로 이루어짐

- compareTo 규약을 지키지 못하면, 비교 활용 클래스와 어울리지 못한다.
    - EX1 : 정렬된 컬렉션인 TreeSet과 TreeMap, 
    - EX2 : 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스인 Collections와 Arrays

<br>

### 첫 번째 규약 - 반사성
### : 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.

```
* Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compartTo(y)) == -sgn(y.compareTo(x))여야 한다. 

-> 따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다.
```
<br>

### 두 번째 규약 - 추이성
### : 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다.

```
* Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
```
<br>

### 세 번째 규약 - 대칭성 
### : 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.

```
* Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))다.
```
<br>

### 이상의 세 규약은 compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 '반사성, 대칭성, 추이성' 을 충족해야 함을 뜻한다.

#### ** 주의사항
1) 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면, compareTo 규약을 지킬 수 없다.

2) Comparable 을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들고, 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 두자.

3) 그 후 내부 인스턴스를 반환하는 '뷰' 메서드를 제공하자.

> 바깥 클래스에 우리가 원하는 compareTo 메서드 구현이 가능 <br> 
> 클라이언트는 필요에 따라 바깥 클래스의 인스턴스를 필드 안에 담긴 원래 클래스의 인스턴스로 다룰 수도 있게 된다.

<br>

### 네 번째 규약
### : compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.

> 필수는 아니지만, 지키는 것이 좋음
```
* (x.compareTo(y) == 0) == (x.equals(y))여야 한다. 
* Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 

"주의 : 이 클래스의 순서는 equals 메서드와 일관되지 않다." // 명시 예시
```

- 이를 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 일관되게 된다.
- compareTo의 순서와 equals의 결과가 일관되지 않은 클래스도 여전히 동작은 한다.
- 단, 이 클래스의 객체를 정렬된 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스(Collection, Set, 혹은 Map)에 정의된 동작과 엇박자를 낼 것이다.
    - 이 인터페이스들은 equals 메서드의 규약을 따른다고 되어 있지만, 정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 compareTo를 사용하기 때문이다.

```
BigDecimal 클래스

@Test
void test() {
    BigDecimal number1 = new BigDecimal("1.0");
    BigDecimal number2 = new BigDecimal("1.00");

    Set<BigDecimal> set1 = new HashSet<>();
    set1.add(number1);
    set1.add(number2);

    Set<BigDecimal> set2 = new TreeSet<>();
    set2.add(number1);
    set2.add(number2);

    assertThat(set1).hasSize(2);
    assertThat(set2).hasSize(1);
}
```

<br>

### compareTo 메서드 작성 요령
---
#### 1. Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일타임에 정해진다.
- 입력 인수의 타입을 확인하거나 형변환할 필요가 없다

#### 2. compareTo 메서드는 각 필드가 동치인지를 비교하는게 아니라 그 순서를 비교한다. 
- 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출
- Comparable 을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용

#### 3. compareTo 메서드에서 관계 연산자 <, >를 사용하는 이전 방식은 거추장스럽게 오류를 유발하니, 객체의 compare 정적 메서드를 이용하자.
- 자바 7부에서부터 변한 내용

#### 4. 핵심 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해진다. 가장 핵심적인 필드를 먼저 비교하자.

```
// 코드 14-2 기본 타입 필드가 여럿일 때의 비교자

public class PhoneNumber implements Comparable<PhoneNumber> {
    private Short areaCode;
    private Short prefix;
    private Short lineNum;

    @Override
    public int compareTo(PhoneNumber phoneNumber) {
        int result = Short.compare(areaCode, phoneNumber.areaCode); // 가장 중요한 필드
        if (result == 0) {
            result = Short.compare(prefix, phoneNumber.prefix); // 두 번째로 중요한 필드
            if (result == 0) {
                result = Short.compare(lineNum, phoneNumber.lineNum); // 세 번째로 중요한 필드
            }
        }
        return result;
    }
}
```

<br>

### Comparator 인터페이스로 비교자 생성
---
* 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.
* 약간의 성능 저하가 뒤따르게 된다.
* 정적 임포트 기능을 이용하면 정적 비교자 생성 메서드들을 그 이름만으로 사용할 수 있어 코드가 훨씬 깔끔해진다.

#### 1. 비교자 생성 메서드를 활용한 비교자
```
public class PhoneNumber implements Comparable<PhoneNumber> {
    private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber phoneNumber) -> phoneNumber.areaCode)
            .thenComparingInt(phoneNumber -> phoneNumber.prefix) // 프리픽스
            .thenComparingInt(phoneNumber -> phoneNumber.lineNum); // 가입자번호

    private Short areaCode;
    private Short prefix;
    private Short lineNum;


    public int compareTo(PhoneNumber phoneNumber) {
        return COMPARATOR.compare(this, phoneNumber);
    }
}
```
- 이 코드는 클래스를 초기화할 때 비교자 생성 메서드 2개를 이용해 비교자를 생성한다.
- comparingInt : 객체 참조를 int 타입 키에 매핑하는 키 추출 함수(key extractor function)를 인수로 받아, 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드
- thenComparingInt : Comparator의 인스턴스 메서드로, int 키 추출자 함수를 입력받아 다시 비교자를 반환 (이 비교자는 첫 번째 비교자를 적용한 다음 새로 추출한 키로 추가 비교를 수행한다). thenComparingInt는 원하는 만큼 연달아 호출할 수 있다.
> 자바의 타입 추론 능력이 최초의 comparingInt를 호출할 때에는 타입을 알아낼 만큼 강력하지 않기 때문에 입력 인수의 타입인 PhoneNumber를 명시해줬지만, 그 이후에 thenComparingInt를 호출할 때는 충분히 타입 추론이 가능하므로 명시하지 않았다.

<br>

#### 2. Comparator와 보조 생성 메서드 : 기본 타입
- Comparator는 long과 double용으로는 comparingInt와 thenComparingInt의 변형 메서드를 갖고 있다.
- short처럼 더 작은 정수 타입에는 int용 버전을 사용하면 된다. 
- 마찬가지로 float는 double용을 이용해 수행한다.

<br>

#### 3. Comparator와 보조 생성 메서드 : 객체 참조 타입
> 객체 참조용 비교자 생성 메서드도 있다. 
- comparing이라는 정적 메서드 2개가 다중정의되어 있다.
    - 첫 번째는 키 추출자를 받아서 그 키의 자연적 순서를 사용한다.
    - 두 번째는 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받는다.

- thenComparing이란 인스턴스 메서드가 3개 다중정의되어 있다.
    - 첫 번째는 비교자 하나만 인수로 받아 그 비교자로 부차 순서를 정한다.
    - 두 번째는 키 추출자를 인수로 받아 그 키의 자연적 순서로 보조 순서를 정한다.
    - 세 번째는 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받는다.

<br>

#### 4. 해시코드 값의 차를 기준으로 하는 비교자 - 추이성을 위배 
> '값의 차'를 기준으로 하는 경우 <br>

> 첫 번째 값이 두 번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫 번째 값이 크면 양수를 반환하는 compareTo나 compare 메서드와 마주할 것이다.

```
// 사용하면 안 되는 방식의 예

public class HashCodeOrderComparator {
    static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(Object o1, Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
    };
}
```
- 이 방식은 정수 오버플로를 일으키거나 IEEE 754 부동소수점 계산 방식에 따른 오류를 낼 수 있다.
- 따라서 위 방식 대신 다음의 두 방식 중 하나를 사용하자.

<br>

#### 5. 정적 compare 메서드를 활용한 비교자
```
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

<br>

#### 6. 비교자 생성 메서드를 활용한 비교자
```
static Comparator<Object> hashCodeOrder = 
            Comparator.comparingInt(o -> o.hashCode());
```

<br><br>

# 정리
---
- 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 `Comparable 인터페이스`를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다. 

- compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰지 말아야 한다. 그 대신 박싱된 기본 타입 클래스에서 제공하는 `정적 compare 메서드`나 Comparator 인터페이스가 제공하는 `비교자 생성 메서드`를 사용하자.