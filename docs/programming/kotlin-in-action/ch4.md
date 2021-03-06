---
sidebar_position: 4
title: 4. 클래스, 객체, 인터페이스
---

## 1. 클래스 계층 정의

- 코틀린 가시성/접근 변경자는 자바와 비슷하지만 아무것도 지정하지 않는 경우 기본 가시성은 다름
- `sealed`는 클래스 상속을 제한

### 코틀린 인터페이스

- 코틀린 인터페이스 안에 구현 메소드도 정의할 수 있음
- `interface`로 인터페이스 선언

```kotlin
interface Clickable {
    fun click()
}

class Button: Clickable {
    override fun click() = println("I was clicked")
}
```

- 자바에선 `extends`, `implements` 키워드를 붙이지만, 코틀린은 콜론(:) 뒤에 인터페이스명을 적음
- 자바에서 `@Override` 애노테이션과 비슷한 `override` 변경자가 있음
- 인터페이스 메소드도 디폴트 구현할 수 있음

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable")
}
```

- 디폴트 메소드를 구현하려는 클래스에선 재정의 해도 되고, 디폴트 메소드를 이용해도 됨

```kotlin
interface Focusable {
    fun setFocus(b: Boolean) =
        println("I ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm focusable")
}
```

- 만약 `Clickable`와 `Focusable`을 구현한다면 어떻게 될까?
    - `showOff()` 메소드를 직접 구현하지 않으면 컴파일러가 어느 인터페이스 구현체를 선택할 지 몰라서 컴파일러 에러를 밷음
    - 컴파일러는 하위 클래스에 두 메소드를 아우르는 구현을 강제화시킴

```kotlin
class Button: Clickable, Focusable {
    override fun click() = println("I was clicked")
    override fun showOff() {
        super<Clickable>.showOff()  // 꺽쇠 괄호에 상위 타입을 지정하여 메소드를 호출할 수 있음
        super<Focusable>.showOff()
    }
}
```
### open, final, abstract 변경자: 기본적으로 final

- Effective Java에선 **상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상속을 금지하라**라고 조언함
- 코틀린 클래스와 메소드도 기본적으로 `final`
  - 상속을 허용하려면 클래스 앞에 `open` 변경자 붙여야 함
```kotlin
open class RichButton: Clickable {
  fun disable() {}        // 이 함수는 파이널. 하위 클래스가 오버라이드 X
  open fun animate() {}   // 이 함수는 열려있음. 하위 클래스가 오버라이드 O
  override fun click() {} // 오버라이드 함수. 하위 클래스가 오버라이드 O
  // 만약 오버라이드 함수를 하위 클래스에서 오버라이드를 원치 않는다면 final을 붙임
  final override fun click() {}
}
```

- `abstract` 클래스는 인스턴스화 X
- `abstract` 내 추상 멤버는 항상 open이 기본

```kotlin
abstract class Animated {
  abstract fun animate()  // 추상 멤버는 구현 x, 하위 클래스에서 오버라이드 해야함

  open fun stopAnimating() {} // 비추상 함수는 기본적으로 final. 오버라이드를 원한다면 open 붙임

  fun animateTwice() {}
}
```

|변경자|이 변경자가 붙은 멤버|설명|
|---|---|---|
|final|오버라이드 X|클래스 멤버의 기본 변경자|
|open|오버라이드 O|반드시 open을 명시해야 오버라이드 할 수 있음|
|abstract|반드시 오버라이드 해야함|추상 클래스 멤버에만 적용 가능. 추상 멤버는 구현 X|
|override|상위 클래스, 인스턴스의 멤버를 오버라이드|오버라이드 멤버는 기본적으로 open. 하위 클래스에 오버라이드를 원치 않으면 final 붙임|

### 가시성 변경자: 기본적으로 공개

- 기존 가시성 변경자는 `public`
- 패키지 전용 가시성 변경자로 `internal`이 있음
  - 모듈 내부에서만 접근 가능
- 최상위 선언에 대해 `private` 가시성을 허용
  - 비공개 기사성인 최상위 선언은 그 선언이 들어있는 파일 내부에서만 사용 가능

|변경자|클래스 멤버|최상위 선언|
|---|---|---|
|public(기본 가시성)|모든 곳에서 볼 수 있음|모든 곳에서 볼 수 있음|
|internal|같은 모듈 안에서만 볼 수 있음|같은 모듈 안에서만 볼 수 있음|
|protected|하위 클래스 안에서만 볼 수 있음|(최상위 선언에 적용할 수 없음)|
|private|같은 클래스 안에서만 볼 수 있음|같은 파일 안에사만 볼 수 있음|

### 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

- 클래스 안에 다른 클래스를 선언할 수 있음
- 자바와의 차이는 **중첩 클래스**는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다는 점

```kotlin
interface State: Serializable

interface View {
  fun getCurrentState(): State
  fun restoreState(state: State)
}
```
- `Button` 클래스 상태를 저장하는 클래스는 `Button` 클래스 내부에 선언

```java
public class Button implements View {
  @Override
  public State getCurrentState() {
    return new ButtonState()
  }

  @Override
  public void restoreState(State state) { ... }
  
  public class ButtonState implements State {
    /**
    * ButtonState 클래스 정의
    */
  }
}
```
- `ButtonState`를 직렬화하면 `NotSerializableException` 오류 발생
  - 자바에서 클래스 안에 정의한 클래스는 내부 클래스로 취급
  - 바깥쪽 Button 클래스에 대한 참조를 묵시적으로 포함
  - 그 참조로 인해 `ButtonState`은 직렬화할 수 없음
- 자바에선 이 문제를 해결하기 위해 `static`으로 선언해야함

```kotlin
class Button: View {
  override fun getCurrentState(): State = ButtonState()

  override fun restoreState(state: State) { ... }

  class ButtonState: State {
    /**
    * ButtonState 클래스 정의
    */
  }
}
```
- 코틀린 중첩 클래스에 아무런 변경자가 없으면 자바 static 중첩 클래스와 동일
- 내부 클래스로 변경하여 바깥쪽 참조를 포함하고 싶으면 `inner` 변경자를 붙이면 됨

![image](https://user-images.githubusercontent.com/4207192/164894105-13d0c1d8-a585-4173-8fe9-3becca3ab335.png)

- 내부에서 바깥쪽 클래스를 참조하기 위해서는 `this@Outer`를 통해 참조할 수 있음

### 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

- [2.3.5절에 있는 클래스 계층](https://kimgs0725.github.io/til/docs/programming/kotlin-in-action/ch2#%EC%8A%A4%EB%A7%88%ED%8A%B8-%EC%BA%90%EC%8A%A4%ED%8A%B8-%ED%83%80%EC%9E%85-%EA%B2%80%EC%82%AC%EC%99%80-%ED%83%80%EC%9E%85-%EC%BA%90%EC%8A%A4%ED%8A%B8%EB%A5%BC-%EC%A1%B0%ED%95%A9)을 다시 가져와서
```kotlin
interface Expr
class Num(val value: Int): Expr
class Sum(val left: Expr, val right: Expr): Expr
```
- when 식에서 Num과 Sum이 아닌 경우 else 분기를 넣어야 함

```kotlin
fun eval(e: Expr) {
  when (e) {
    is Num -> e.val
    is Sum -> eval(e.right) + eval(e.left)
    else ->
      throw IllegalArgumentException("Unknown expression")
  }
}
```
- 코틀린은 이 문제를 해결하기 위해 `sealed` 클래스를 제공함
  - `sealed`는 상위 클래스를 상혹한 하위 클래스 정의를 제한시킬 수 있음
```kotlin
sealed class Expr {
  class Num(val value: Int): Expr()
  class Sum(val left: Expr, val right: Expr): Expr()
}

fun eval(e: Expr) {
  when (e) {  // when식에서 모든 하위 클래스를 검사하므로 별도의 else 분기가 없어도 됨
    is Expr.Num -> e.val
    is Expr.Sum -> eval(e.right) + eval(e.left)
  }
}
```

## 2. 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

- 코틀린에선 생성자를 주 생성자와 부 생성자로 구분
  - 초기화 블록을 통해 초기화 로직을 추가

### 클래스 초기화: 주 생성자와 초기화 블록

- 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 **주 생성자**라고 함
```kotlin
class User constructro (_nickname: String) {  // 파라미터가 하나만 있는 주 생성자
  val nickname: String
  init {    // 초기화 블록
    nickname = _nickname
  }
}
```

- `constructor`는 주 생성자와 부 생성자를 정의내릴 때 사용
- `init`은 초기화 블록을 시작
  - 클래스 안에 여러 초기화 블록을 선언할 수 있음
- 주 생성자 앞에 별다른 애노테이션이나 가시성 변경자가 없다면 `constructor`는 생략 가능

```kotlin
class User(_nickname: String) {
  val nickname = _nickname
}
```

- 주 생성자 파라미터 이름 앞에 `val`을 추가하면 프로퍼티 정의와 초기화를 간략화할 수 있음

```kotlin
class User(val nickname: String)
```

- 함수와 마찬가지로 디폴트 파라미터를 정의할 수 있음
- 인스턴스 생성은 `new` 키워드 없이 생성해야 함
- 상속 시, 부모 클래스로 넘겨야할 파라미터가 있다면 부모클래스 이름 뒤에 괄호치고 생성인자를 넘길 수 있음

```kotlin
open class User(val nickname: String) { ... }

class TwitterUser(nickname: String): User(nickname) { ... }
```

- 부모 클래스에 파라미터가 없더라도, 빈 괄호는 표시해야함
  - 단, 인터페이스의 경우 인스턴스로 생성될 수 없기에 괄호는 표시하지 않음
- 클래스 외부에서 인스턴스화하지 못하게 막고 싶으면 모든 생성자를 private으로 변경하면 됨

```kotlin
class Secretive private constructor() { ... }
```

### 부 생성자: 상위 클래스를 다른 방식으로 초기화

- 생성자가 많이 필요한 자바와 달리, 코틀린에선 디폴트 파라미터와 이름 붙은 인자 문법으로 오버로드를 해결할 수 있음
- 그럼에도 생성자가 여럿 필요한 경우
  - 프레임워크 클래스를 확장해야 하는데, 여러 가지 방법으로 인스턴스를 초기화할 수 있게 다양한 생성자를 지원해야하는 경우

```kotlin
open class View {
  constructor(ctx: Context) {
    ...
  }

  constructor(ctx: Context, attr: AttributeSet) {
    ...
  }
}
```

- 클래스를 확장하면서 똑같이 부 생성자를 정의할 수 있음

```kotlin
class MyButton : View {
  constructor(ctx: Context) : super(ctx) {
    ...
  }

  constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) {
    ...
  }
}
```

- `this`를 통해 클래스 자신의 다른 생성자 호출 가능

```kotlin
class MyButton : View {
  constructor(ctx: Context) : this(ctx, MY_STYLE) { // 이 클래스의 다른 생성자에게 위임
  }

  constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) {
    ...
  }
}

```

- 부 생성자가 필요한 이유는 자바 상호운용성 때문
- 그리고 다른 생성 방법이 여럿 존재하는 경우에도 부 생성자가 필요

### 인터페이스에 선언된 프로퍼터 구현

- 인터페이스에 추상 프로퍼티 선언을 넣을 수 있음

```kotlin
interface User {
  val nickname: String
}
```

- 이는 인터페이스를 구현하는 클래스에서 추상 프로퍼티를 구현해야함
- 그럼 클래스에서 이 프로퍼티를 어떻게 구현해야할까?

```kotlin
class PrivateUser(override val nickname: String): User  // 주 생성자에 있는 프로퍼티

class SubscribingUser(val email: String): User {
  override val nickname: String
    get() = email.substringBefore('@')    // 커스텀 게터
}

class FacebookUser(val accountId: Int): User {
  override val nickname = getFacebookName(accountId)    // 프로퍼티 초기화 식
}
```

- 인터페이스에는 추상 프로퍼티뿐만 아니라 게터와 세터가 잇는 프로퍼티를 선언할 수 있음
  - 단, 게터, 세터를 뒷받침할 필드를 참조할 수 없음

### 게터와 세터에서 뒷받침하는 필드에 접근

- 값을 저장하는 동시에 로직을 실행할 수 있게 하기 위해서는 접근자 안에서 프로퍼티를 뒷받침하는 필드에 접근할 수 있어야 하ㅣㅁ
- 프로퍼티에 저장된 값을 로그로 남기려는 경우를 생각

```kotlin
class User(val name: String) {
  var address: String = "unspecified"
    set(value: String) {
      println("""
        Address was changed for $name:
        "$field" -> "$value".""".trimIndent())
      )
      field = value
    }
}
```

- 접근자 본문에서 `field`라는 특별한 식별자를 통해 뒷받침하는 필드에 접근할 수 있음
- 컴파일러는 디폴트 접근자 구현을 사용하건 직접 게터나 세터를 정의하건 관계없이 게터나 세터에서 `field`를 사용하는 프로퍼티에 대해 뒷받침하는 필드를 생성함

### 접근자의 가시성 변경

- 기본적으로는 프로퍼티의 가시성과 같음
- 원한다면 `get`이나 `set` 앞에 가시성 변경자를 추가해서 가시성을 변경할 수 있음

```kotlin
class LengthCounter {
  var counter: Int = 0
    private set
  
  fun addWord(word: String) {
    counter += word.length
  }
}
```

## 3. 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임 

### 모든 클래스가 정의해야하는 메소드

- 코틀린도 `toString`, `equals`, `hashCode`등을 오버라이드 해야함
- 다음 클래스가 있다고 가정

```kotlin
class Client(val name: String, val postalCode: Int)
```

#### 문자열 표현: toString()
- 기본으로 제공되는 문자열 표현은 `Client@5e9f23b4`와 같이 표현
- 오버라이드를 통해 유용한 정보를 뿌리는 `toString`으로 재정의할 수 있음

```kotlin
class Client(val name: String, val postalCode: Int) {
  override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
```

#### 객체의 동등성: equals()

- 때때로 서로 다른 객체가 동일한 데이터를 포함한 경우, 그 둘을 동등한 객체로 간주해야하는 경우도 존재
  - 동등성 연산에는 `==`을 샤용
- 위 요구사항을 만족시키기 위해 `equals`을 구현

```kotlin
class Client(val name: String, val postalCode: Int) {
  override fun equals(other: Any?): Boolean {
    if (other == null || other !is Client) {
      return false
    }
    return name == other.name && postalCode == other.postalCode
  }

  override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
```

#### 해시 컨테이너: hashCode()

- `equals`를 오버라이드할 때 반드시 `hashCode`도 오버라이드해야 함
- JVM 언어에선 `hashCode`가 지켜야하는 제약이 존재

  > equals()가 true를 반환하는 두 객체는 반드시 같은 hashCode()를 반환해야한다

```kotlin
class Client(val name: String, val postalCode: Int) {
  override fun hashCode(): Int = name.hashCode() * 31 + postalCode

  override fun equals(other: Any?): Boolean {
    if (other == null || other !is Client) {
      return false
    }
    return name == other.name && postalCode == other.postalCode
  }

  override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
```

### 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성

- `data` 라는 변경자를 클래스 앞에 붙이면 컴파일러가 자동으로 `toString()`, `equals()`, `hashCode()`을 만들어줌

```kotlin
data class Client(val name: String, val postalCode: Int)
```

- `data` 클래스는 다음 메소드를 포함
  - 인스턴스가 비교를 위한 `equals`
  - HashMap과 같은 해시 기반 컨테이너에서 키로 사용할 수 있는 `hashCode`
  - 클래스의 각 필드를 선언 순서대로 표시하여 문자열로 표현하는 `toString`
- 위 메소드 말고도 `copy()` 메소드도 자동 구현해줌

### 클래스 위임: by 키워드 사용

- 대규모 설계 시, 구현 상속은 시스템을 취약하게 만듦
- 코틀린에선 기본적으로 클래스를 `final`로 두어 함부로 상속하지 못하게 함
- 대신 상속을 허용하지 않는 클래스에서 새로운 동작을 추가하려 할 때, 데코레이터 패턴을 사용
  - 새로운 클래스에 기능을 정의하고, 기존 클래스는 새로운 클래스로 메소드 요청을 전달
  - 하지만, 새로운 클래스를 만들어야 하기 때문에 복잡

```kotlin
class DelegatingCollection<T>: Collection<T> {
  private val innerList = arrayListOf<T>()
  override val size: Int get() = innerSize.size
  override fun isEmpty(): Boolean = innerList.isEmpty()
  override fun contains(element: T): Boolean = innerList.contains(element)
  override fun iterator(): Iterator<T> = innerList.iterator()
  override fun containsAll(elements: Collection<T>): Boolean = innerList.containsAll(elements)
}
```

- 코틀린에선 인터페이스를 구현할 때, `by` 키워드를 통해 인터페이스에 대한 구현을 다른 객체에 위임 중이란 것을 명시할 수 있음
- 메소드 중 일부 동작을 변경하고 싶은 메소드만 오버라이드하면 됨

```kotlin
class CountingSet<T>(
    private val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet {
    var objectsAdded = 0

    override fun add(element: T): Boolean {
        objectsAdded++
        return innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        objectsAdded += elements.size
        return innerSet.addAll(elements)
    }
}
>>> val cset = CountingSet<Int>()
>>> cset.addAll(listOf(1, 1, 2))
>>> println("${cset.objectsAdded} objects were added, ${cset.size} remain")
3 objects were added, 2 remain
```

## 4. object 키워드: 클래스 선언과 인스턴스 생성

- `object`는 선언과 동시에 인스턴스를 생성함
- `object`를 쓰는 케이스는 다음과 같음
  - **객체 선언**은 싱글턴을 정의하는 방법 중 하나
  - **동반 객체**는 인스턴스 메소드는 아니지만 어떤 클래스와 관련있는 메소드와 팩토리 메소드를 담을 때 쓰임
  - 객체 식은 자바의 **무명 내부 객체** 대신 쓰임

### 객체 선언: 싱글턴 쉽게 만들기

- 인스턴스가 하나만 필요한 클래스를 만들기 위해 [싱글턴 패턴](https://ko.wikipedia.org/wiki/%EC%8B%B1%EA%B8%80%ED%84%B4_%ED%8C%A8%ED%84%B4)을 자주 이용함
- 코틀린에선 **객체 선언** 기능을 이용해 싱글턴을 언어상에서 기본 지원

```kotlin
object Payroll {
  val allEmployees = arrayListOf<Person>()
  fun calculateSalary() {
    for (person in allEmployees) {
      ...
    }
  }
}

>>> Payroll.allEmployees.add(Person(...))
>>> Payroll.calculateSalary()
```

- 객체 선언 뒤에 .을 붙여 객체에 속한 프로퍼티나 메소드에 접근
- 객체 선언도 클래스나 인스턴스를 상속할 수 있음

```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
  override fun compare(file1: File, file2: File): Int {
    return file1.path.compareTo(file2.path, ignoreCase = true)
  }
}
```

> **싱글턴과 의존관계 주입**
> 
> 싱글턴 패턴과 마찬가지 이유로 대규모 소프트웨어 시스템에서는 객체 선언이 항상 적합하지는 않다.
> 의존 관계가 별로 많지 않은 소규모 소프트웨어에서는 싱글턴이나 객체 선언이 유용하지만,
> 시스템을 구현하는 다양한 구성 요소와 상호작용하는 대규모 컴포넌트에는 싱글턴이 적합하지 않다.
> 이유는 객체 생성을 제어할 방법이 없고 생성자 파라미터를 지정할 수 없어서이다.
>
> 생성을 제어할 수 없고 생성자 파라미터를 지정할 수 없으므로 단위 테스트를 하거나 소프트웨어
> 시스템의 설정이 달라질 때 객체를 대체하거나 객체를 의존관계를 바꿀 수 없다.
> 따라서 그런 기능이 필요하다면 자바와 마찬가지로 의존관계 주입 프레임워크와 코틀린 클래스를 함께 사용해야한다.

- 클래스 안에 `object`를 선언할 수 있음
  - 이 역시 한번만 생성됨

```kotlin
data class Person(val name: String) {
  object NameComparator : Comparator<Person> {
    override fun compare(p1: Person, p2: Person): Int = p1.name.compareTo(p2.name)
  }
}

>>> val persons = listOf(Person("Bob"), Person("Alice"))
>>> println(persons.sortedWith(Person.NameComparator))
[Person(name=Alice), Person(name=Bob)]
```

### 동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소

- 코틀린 클래스에선 `static`을 지원하지 않기 때문에 패키지 수준의 **최상위 함수**와 **객체 선언**을 주로 활용
  - 하지만 최상위 함수는 객체 내 private 멤버에 접근할 수 없음
- 클래스의 인스턴스와 관계없이 호출, but 클래스 내부 정보에 접근해야하는 함수가 필요 -> 중첩된 객체 선언을 멤버 함수로 정의
  - 팩토리 메소드를 예로 들 수 있음

  ![image](https://user-images.githubusercontent.com/4207192/165403569-04326497-3675-4d97-b4ae-56dcf7cf0c48.png)

- `companion` 키워드를 통해 클래스의 동반 객체를 만들 수 있음
- 동반 객체는 자신을 둘러싼 클래스의 모든 `private` 멤버에 접근 가능

```kotlin
// without 동반 객체
class User {
  val nickname: String

  constructor(email: String) {
    nickname = email.substringBefore('@')
  }

  constructor(facebookAccountId: Int) {
    nickname = getFacebookName(facebookAccountId)
  }
}

// with 동반 객체
class User private constructor(val nickname: String) {  // 주 생성자를 비공개로 만듦
  companion object {
    fun newSubscribingUser(email: String) =
      User(email.substringBefore('@'))

    fun newFacebookUser(accountId: Int) =
      User(getFacebookName(accountId))
  }
}
```

- 다만 클래스를 확장해야하는 경우, 하위 클래스에서 오버라이드 할 수 없으므로 여러 생성자를 사용하는게 좋음

### 동반 객체를 일반 객체처럼 사용

- 동반 객체도 일반 객체
  - 일반 객체처럼 이름을 붙이거나
  - 인터페이스를 상속
  - 확장 함수와 프로퍼티를 정의할 수 있음

#### 동반 객체에 이름 부여

```kotlin
class Person(val name: String) {
  companion object Loader {
    fun fromJSON(jsonText: String): Person = ...
  }
}

>>> Person.Loader.fromJSON(...)
>>> Person.fromJSON(...)
```

#### 동반 객체에서 인터페이스 구현

- `object`처럼 인터페이스를 구현할 수 있음

```kotlin
interface JSONFactory<T> {
  fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
  companion object : JSONFactory<Person> {
    override fun fromJSON(jsonText: String): Person = ...
  }
}
```

#### 동반 객체 확장

- 동반 객체 안에 함수를 정의함으로써 클래스에 대해 호출할 수 있는 확장함수를 만들 수 있음
  - `C`라는 클래스 안에 `C.Companion` 동반 객체 안에 `func`를 정의하면 외부에서 `func`를 `C.func`로 호출할 수 있음

```kotlin
class Person(val firstName:  String, val lastName: String) {
  companion object {
  }
}

fun Person.Companion.fromJSON(json: String): Person {
  ...
}

val p = Person.fromJSON(json)
```

- 여기서 동반객체에 대한 확장 함수를 작성하려면, 원래 클래스에 동반 객체를 꼭 선언해야함(빈 내용이라도)

### 객체 식: 무명 내부 클래스를 다른 방식으로 작성

- 무명 객체를 정의할 때에도 `object` 키워드를 씀

```kotlin
// 무명 클래스로 이벤트 리스너 구현
window.addMouseListener(
  object: MouseAdapter() {   // MouseAdapter를 확장하는 무명 객체 선언
    override fun mouseClicked(e: MouseEvent) {
      ...
    }

    override fun mouseEntered(e: MouseEvent) {
      ...
    }
  }
)
```

- 객체 선언과 비슷하지만, 이름이 없다는 것이 특징
- 객체 선언과 달리 무명 객체는 싱글턴이 아님. 식이 쓰일 때마다 인스턴스가 생성
- 무명 객체 식 안에 로컬 변수 접근 가능

```kotlin

fun countClicks(window: Window) {
  var clickCount = 0
  window.addMouseListener(object: MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
      clickCount++
    }
  })
}
```

## 5. 요약
- 코틀린 인터페이스는 디폴트 메소드와 프로퍼티를 포함
- 모든 코틀린 선언은 기본 `public final`임
- `final`을 안 붙이려면 `open` 키워드 사용
- `internal`은 같은 모듈에서만 접근 가능
- 중첩 클래스는 내부 클래스가 아님. `inner` 키워드를 통해 중첩 클래스 내에서 바깥 클래스에 대한 참조 가능
- `sealed` 클래스를 사용하는 클래스를 정의하려면 반드시 부모 클래스 정의 안에 중첩 클래스로 정의
- 초기화 블록과 부 생성자를 통해 인스턴스를 유연하게 초기화
- `data class`를 사용하면 컴파일러가 `equals`, `hashCode`, `toString`, `copy` 등의 메소드를 자동으로 생성
- 클래스 위임을 사용해 위임에 작성되는 보일러 플레이트 코드들을 예방
- 객체 선언을 통해 싱글턴 클래스를 정의