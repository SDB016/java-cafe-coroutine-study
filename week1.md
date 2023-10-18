# 1주차
- 변수, 상속, 연산자
- 조건문, 반복문
- 함수, 익명함수, 람다 표현식, 클로저


## 변수

코틀린에서는 기본적으로 아래 형식처럼 변수를 선언한다.

`val/var name: Type` 

`val`로 선언한 변수는 자바에서 `final` 키워드와 함께 선언하는 것처럼 추후에 할당한 참조의 변경을 막아야 할 때 사용하고, `var`키워드는 추후 객체 변경이 필요할 때 사용한다. 

`val`은 변경 불가능한 참조를 저장하는 키워드이다. 즉, 참조 자체는 불변이지만 그 참조가 가리키는 객체의 내부 값은 변경될 수 있다.

`val → value`, `var → variable`

```kotlin
val languages = arrayListOf("Java")
languages.add("Kotlin")
```

`var`키워드를 사용하면 할당된 참조를 변경할 수 있지만, 변수의 타입은 고정돼 바뀌지 않는다.

```kotlin
var answer = 42
answer = "this is error"  // 컴파일 오류 발생
```

- 프로퍼티
    
    ### 프로퍼티는 동작이 아니라 상태를 나타내야 한다.
    
    여기서 변수를 지칭하는 용어를 살펴보면 자바에서는 **필드(field)**라는 용어를 사용하지만, 코틀린에서는 `val` , `var`로 선언한 변수를 **프로퍼티(property)**라고 부른다. 
    
    그 이유는 변수 선언 시 자동으로 `getter`, `setter`(`var` 로 변수 선언시에만) 내장 함수가 생성되기 때문인데, 이 부분을 더 자세히 살펴보자. 
    
    ```kotlin
    // Kotlin Code
    val name = "Java"
    val surname = "Cafe"
    val fullName: String
        get() = "$name $surname"
        
    // Java Decompile
    public final class MainKt {
       @NotNull
       private static final String name = "Java";
       @NotNull
       private static final String surname = "Cafe";
    
       @NotNull
       public static final String getName() {
          return name;
       }
    
       @NotNull
       public static final String getSurname() {
          return surname;
       }
    
       @NotNull
       public static final String getFullName() {
          return name + ' ' + surname;
       }
    }
    ```
    
    위 코드에서는 `fullName`변수는 읽기 전용 프로퍼티로써 초기화 하지 않고 `getter`만 별도로 정의했다. 이 코드를 자바로 디컴파일 해보면 `name`과 `surname`과는 다르게 은 필드가 존재하지 않는 것을 확인할 수 있다. 
    
    많은 글에서 코틀린의 프로퍼티를 설명할 때 **`필드 + getter + setter`**라고 정의하지만, 엄밀히 말하면 프로퍼티는 필드가 필요하지 않고 **`val 접근자에서는 getter`, `var 접근자에서 getter + setter`**를 나타내는 것이라 할 수 있다.
    
    위처럼 프로퍼티가 캡슐화되어 있다는 점을 이용할 수 있다. 예를 들어 `Date` 타입의 프로퍼티를 사용하고 있었는데 직렬화 등의 문제로 객체의 타입을 바꾸어야 한다는 상황이라 가정하자. 이미 이 프로퍼티는 프로젝트의 여러 곳에서 사용하고 있어 직접적인 변경이 어려울 때 아래와 같이 기존 프로퍼티를 wrap/unwrap 하여 해결할 수 있다.
    
    ```kotlin
    var date: Date
    	get() = Date(millis)
      set(value) {
      	millis = value.time
      }
    ```
    
    프로퍼티는 필드가 필요하지 않다는 점, 즉 프로퍼티는 본질적으로 함수라는 점에서 자바의 필드와 비교해 아래의 추가적인 차이점들도 가지게 된다.
    
    - 인터페이스에 프로퍼티 정의 가능
        
        ```kotlin
        interface Book {
        	val name: String
        }
        ```
        
    - 오버라이드 가능
        
        ```kotlin
        open class Developer {
        	open val language: String = "C++"
        }
        class BackendDeveloper : Developer() {
        	override val language: String = "Kotlin"
        }
        ```
        
    - 위임 가능
        
        ```kotlin
        val db: Database by lazy { connectToDb() }
        ```
        
    - 확장 프로퍼티 가능
        
        ```kotlin
        val Context.preferences: SharedPreferences
        	get() = PreferenceManager.getDefaultSharedPreferences(this)
        
        val Context.inflater: LayoutInflater
        	get() = getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
        ```
        
    
    위처럼 프로퍼티는 본질적으로는 함수이다. 하지만 원칙적으로 프로퍼티를 함수의 대체로 사용해서는 안되며 **상태**를 나타내거나 **설정**하기 위한 목적으로만 사용해야 한다. 관습적으로 `getter`에는 많은 계산량이 필요하지 않다고 생각하기 때문이다. 만약 계산량이 많은 경우에는 함수로 선언해야 사용자가 캐싱 등을 고려할 수 있다.
    
    아래는 프로퍼티 대신 함수로 선언해야 하는 경우이다.
    
    - 복잡도가 O(1)보다 높은 경우
    - 비즈니스 로직을 포함하는 경우
    - 동작마다 다른 결과가 나오는 경우
    - 변환의 경우
    - getter에서 프로퍼티의 상태 변경이 일어나는 경우
    
    반대로 상태를 추출하거나 설정하는 경우에는 함수가 아닌 프로퍼티를 사용해야 한다.
    
    ```kotlin
    // Incorrect example
    class UserIncorrect {
    	private var name: String = ""
        
        fun getName() = name
        
        fun setName(name: String) {
        	this.name = name
        }
    }
    
    // Correct example
    class UserCorrect {
    	var name: String = ""
    }
    ```
    
    ```kotlin
    // Incorrect example
    val Tree<Int>.sum: Int
        get() = when (this) {
            is Leaf -> value
            is Node -> left.sum + right.sum
        }
    
    // Correct example
    fun Tree<Int>.sum(): Int = when (this) {
        is Leaf -> value
        is Node -> left.sum() + right.sum()
    }
    ```
    
    이런 점에서 프로퍼티는 **상태**를 나타내고, 함수는 **동작**을 나타낸다.
    

## 상속

상속이란 상위 클래스 멤버(함수, 프로퍼티)를 하위 클래스에서 자신의 멤버처럼 사용할 수 있게 하는 기능이다.

```kotlin
open class Base(p: Int) {
    fun testFunction() {}
}

class Derived(p: Int) : Base(p)
```

자바에서 최상위 클래스가 Object인 것처럼 코틀린에서는 최상위 클래스가 Any이다. 즉, 모든 코틀린 클래스는 [Any 클래스](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/)의 상속을 받는다. 

```kotlin
class Example // Implicitly inherits from Any
```

거의 대부분이 자바와 동일한 상속 개념을 갖고 있지만 큰 차이점이 하나 있다. 코틀린은 자바와 달리 모든 클래스 및 멤버가 기본적으로 `final` 선언이 되어 있는데 이는 상속을 불가능하게 한다. 만약 상속이 가능하게 만드려면 별도로 `open` 키워드를 사용해야 한다. 

```kotlin
open class Shape {
		open val vertexCount: Int = 0
    
		open fun draw() { /*...*/ }
    fun fill() { /*...*/ }
}

class Circle() : Shape() {
		override val vertexCount = 4

    final override fun draw() { /*...*/ }
}
```

위의 코드를 예시로 보면 `shape` 클래스와 `draw` 함수는 `open`키워드를 명시했기 때문에 상속 및 오버라이드가 가능하지만 `fill`함수와 같이 `open` 키워드가 없을 경우 override가 불가능하다. 또한 `Circle.draw`처럼 override한 함수에 `final` 선언을 하면, `Circle`을 상속하는 다른 클래스에서 이 함수를 override하지 못하도록 막을 수 있다. 

`val`로 선언한 프로퍼티도 getter/setter 함수를 오버라이드할 수 있으므로 재정의가 가능하다. 여기서 주의할 점은 `val` 프로퍼티를 `var`로 재정의는 가능하지만, `var` 프로퍼티를 `val` 프로퍼티로 재정의 하는 것은 불가능하다. 기존에 존재하던 getter를 재정의하고 setter를 추가 선언하기 때문이다. 반대의 경우, setter를 제거해야 하는데 이는 상속으로 불가능하다.

### 코틀린은 왜 default로 final을 채택하였나

첫번째 이유로 코틀린은 **가변을 사용했을 때 발생하는 문제를 최대한 방지**하기 위해 최대한 불변을 기본으로 하고 있다.

두번째 이유는 **상속**에 있는데 더 자세히 알아보자.

자바에서도 상속, 오버라이딩과 관련해 다음 권장사항이 존재했다.

- 오버라이딩 가능한 메소드는 반드시 문서화하고, 상황에 따라 다르게 구현될 수 있음을 자세히 명시해야 한다. (오버라이드로 인해 본래의 의도와는 다른 동작을 하도록 바뀔 수 있는 위험이 존재하기 때문)
- 오버라이딩 가능한 메소드는 하위 클래스를 만들어 반드시 테스트한다.
- 생성자에서 오버라이딩 가능 메소드를 호출하면 원하지 않는 동작을 할 수 있다.
- 위와 같은 이유로 `clone`과 `readObject`에서 오버라이딩 가능 메소드를 호출하면 원치 않는 동작을 할 수 있다.

상속을 사용시 개발자들의 실수로 인해 위와 같은 부작용을 일으킬 가능성이 있다. 따라서 기본으로 `final`을 선언하여 예측 가능성을 높여 안정성을 증가시켰다.

세번째 이유는 **컴파일 시점의 차이**가 존재한다는 것이다. 코틀린의 `final` 클래스 및 멤버 함수는 컴파일 시점에 정적 바인딩을 이용하기 때문에 런타임에 다형성 체크를 하지 않는다. (`open`키워드를 사용하면 동적 바인딩이 이뤄져 런타임에 다형성 체크)

이는 가상호출의 오버헤드를 줄이고, 최적화를 가능하게 하여 성능 향상에 도움을 준다. 

## 연산자

자바와 동일하게 산술 연산자(`+`, `-`, `*`, `/`, `%`), 관계 연산자(`>`, `<`, `≥`, `≤`, `==`, `≠`), 대입 연산자(`+=`, `-=`, `*=`, `/=`, `%=`), 단항 연산자(`++`, `--`), 논리 연산자(`&&`, `||`, `!`), 비트 연산자(`shl`, `shr`, `ushr`, `and`, `or`, `xor`, `inv`)등이 있다.

추가적으로 코틀린은 고유의 연산자로 엘비스 연산자(elvis operator)를 갖고 있다.(`?:`)

엘비스 연산자의 좌측 항이 null일 경우 엘비스 연산자 오른쪽 값을 반환하고, 그렇지 않을 경우 좌측 항을 그대로 반환하는 연산자이다.

```kotlin
var name: String? = null
var notNullName: String = name ?: "new name"
// var notNullName: String = if (name != null) name else "new name"

print(notNullName)  // "new name"
```

 엘비스 연산자를 통해 null-safe한 프로퍼티를 보장할 수 있다.

아래처럼 엘비스 연산자를 활용도 가능하다.

1. `return` 활용한 Null 체크
    
    ```kotlin
    data class Student(val name: String?)
    
    fun checkNullName(student: Student) {
    	val studentName: String = student.name ?: return
    	println(studentName)
    }
    
    fun main() {
    	checkNullName(Student(null))
    }
    ```
    
2. null 체크 후 `throw`
    
    ```kotlin
    getResult() ?: throw IllegalStateException()
    ```
    
3. null 체크 후 동작
    
    ```kotlin
    var name: String? = null
    var notNullName: String = name ?: run {
    	println("previous name is null")
    	return
    }
    
    print(notNullName)  // previous name is null
    ```
    
4. boolean 값 주입

```kotlin
suspend fun test() {
	val result = withTimeoutOrNull(300L) {
		delay(1000)
		println("true")
		true
	} ?: false  // 지양
}
```

위 예시 이외에도 다양하게 활용이 가능하기 때문에 null이 예상되는 상황에 적절하게 활용하는 것을 권장한다.

## 조건문

### if

코틀린은 기본적으로 자바의 조건문과 같은 형식을 가진다.

```kotlin
fun test(input: Int) {
	if (input < 0) println("음수")
	else if (0 < input) println("양수")
	else println("0")
} 
```

자바와의 차이점은 코틀린에서의 if문은 값을 바로 반환할 수 있다는 것이다.

```kotlin
val a = 10
val b = 5

val maxNum = if (a > b) {
	println("a가 더 큼")
	a
} else {
	println("b가 더 큼")
	b
}
```

if 문으로 값을 바로 반환할 때 조건문의 내용이 여러 줄이면 마지막 값을 반환한다.

### when

코틀린에서 when은 자바에서 switch와 동일한 동작을 한다.

```kotlin
val name = "tom"

when (name) {
	"tom" -> println("name is tom")
	"paul" -> println("name is paul")
	else -> println("stranger")
}
```

`case`는 `->`로 표현하고, `default`는 `else`로 표현한다. 

when도 if와 마찬가지로 바로 값을 반환할 수 있다.

```kotlin
val flag: Boolean? = null

val value = when (flag) {
	true -> 1
	false -> 0
	else -> -1
}
```

이때에는 반드시 모든 경우의 조건에 대해 명시해주어야 한다.

when문은 표현식, 범위를 조건식으로 가질 수도 있다.

```kotlin
val testParam = Student("tom", 27)

when (testParam) {
	is Student -> println("testParam type is Student")
	else -> println("I don't know testParam type")
}

val age = when (testParam.age) {
	in 10 .. 20 -> 15
	in 20 .. 30 -> 25
	else -> 50
}
```

when은 다음과 같은 장점을 가진다

- 가독성 및 유지 보수성을 높여줌
- 변수 스코프를 제한해 다른 곳에서 사용되지 못하도록 방지

#### 장점 1. 가독성 및 유지 보수성

when은 if에 비해 가독성을 높여주기 때문에 만약 여러개의 조건을 비교해 값을 반환해야 한다면 when을 더 권장한다.

```kotlin
fun isPromotionTarget(addToCart: Boolean, rates: Int): Boolean {
    if (rates < 2) { return false }
    if (rates > 3) { return false }
    if (rates == 3) { return true }
    return addToCart && rates == 2
}

fun isPromotionTarget(addToCart: Boolean, rates: Int) = when {
    rates < 2 -> false
    rates > 3 -> false
    rates == 3 -> true
    else -> addToCart && rates == 2
}
```

위 코드는 동일한 동작을 하는 함수를 if문과 when으로 표현한 코드이다. when을 사용했을 때 가독성이 증가하는 장점도 있지만 또다른 장점을 가진다. when 표현식이 사용될 때 컴파일러는 표현식이 가능한 모든 경우에 대해 값을 입력하는지 검증한다. 이는 컴파일 시점에 이루어지기 때문에 오류를 줄이는 데 큰 기여를 해준다.

만약 아래처럼 인자가 가질 수 있는 경우가 많을 때 개발자가 실수를 할 수 있는데 이를 방지해준다.

```kotlin
fun whatToDo(dayOfWeek: Any) = when (dayOfWeek) {
    "Saturday", "Sunday" -> "Relax"
    in listOf("Monday", "Tuesday", "Wednesday", "Thursday") -> "Work hard"
    in 2..4 -> "Work hard"
    "Friday" -> "Party"
    is String -> "What?"
    else -> "No Clue"
}
```

> when문이 명령어로 사용될 때(Unit 반환)는 else가 없어도 관계 없음
> 

이는 Enum class에서 더 큰 장점을 가진다.

```kotlin
enum class Color{
    RED,ORANGE,YELLOW,GREEN,BLUE,INDIGO
}

fun getMnemoic(color:Color) = when (color) {
    Color.RED,Color.ORANGE->"Richard"
    Color.YELLOW -> "York"
    Color.GREEN -> "Gave"
    Color.BLUE -> "Battle"
    Color.INDIGO -> "In"
		else -> "None"
}
```

위와 같은 코드가 있을 때 만약 Color에 새로운 값이 추가 되더라도 else에서 처리되어 에러를 방지할 수 있다.

#### 장점 2. 스마트 캐스트

코틀린에서는 `is`를 사용해 변수 타입을 검사한다. 자바의 `instanceof`와 다른 점은 타입을 확인한 후 컴파일러가 그 변수를 자동으로 캐스팅 해준다. 이를 스마트 캐스트라고 부른다.

when에서 스마트 캐스트를 이용하면 본문에서 좀 더 편한 코드 작성이 가능하다.

```kotlin
fun withSmartCast(param: Any) {
	when (param) {
		is String -> println("param's length : ${param.length()}") // String으로 캐스팅
		is Int -> println("2 + $param = ${2 + param}") // Int로 캐스팅
		else -> println("param is another type")
	}
}
```

```kotlin
fun eval(e: Expression): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

#### 장점 3. 변수 스코프 제한

```kotlin
fun systemInfo(): String {
    val numberOfCores = Runtime.getRuntime().availableProcessors()
    return when (numberOfCores) {
        1 -> "1 core, packing this one to the museum"
        in 2..16 -> "You hav $numberOfCores cores"
        else -> "$numberOfCores cores!, I want your machine"
    }
}
```

위 코드에서 `numberOfCores`는 when 안에서만 사용됨에도 불구하고 `systemInfo` 함수에서 계속 사용될 수 있다.

```kotlin
fun systemInfo(): String =
    when (val numberOfCores = Runtime.getRuntime().availableProcessors()) {
        1 -> "1 core, packing this one to the museum"
        in 2..16 -> "You hav $numberOfCores cores"
        else -> "$numberOfCores cores!, I want your machine"
    }
```

만약 위처럼 리펙토링한다면 `numberOfCores`는 when에서만 사용되도록 제한되어 안전한 코드를 작성할 수 있다.

위의 장점들에서 볼 수 있듯 when은 단순히 switch문을 대체하는 것에서 끝나는 게 아니라 코틀린의 가독성, 유지 보수성, 안정성을 높여주는 기능을 한다.

## 반복문

코틀린에서 for문은 다양한 방식으로 작성될 수 있다.

```kotlin
fun main(args:Array<String>) {
    for(i: Int in 1..10)
        print("$i ")    //output : 1, 2, 3, 4, 5 ... 10

    val len: Int = 5
    for(i in 1..len)
        print("$i ")    //output : 1, 2, 3, 4, 5

    for(i in 1 until len)
        print("$i ")    //output : 1, 2, 3, 4

// step 키워드로 증가값 변경
		for(i: Int in 1..10 step(2))
        print("$i ")    //output : 1, 3, 5, 7, 9

// step은 인자로 음수를 지원하지 않음
    for(i in 10..1 step(-1))    //error    
        print("$i ")

// 음수 대신 downTo 사용
		for(i in 10 downTo 1)    //output : 10, 9, 8, 7 ... 1
        print("$i ")

    for(i in 10 downTo 1 step(2))    //output : 10, 8, 6, 4, 2
        print("$i ")

// 배열을 for로 출력
		val arr: IntArray = intArrayOf(10, 20, 30, 40, 50)
    for(i in arr)
        print("$i ")    //output : 10, 20, 30, 40, 50

    val list = listOf<String>("korea", "salmon", "T_T")
    for(i in list)
        print("$i ")    //output : korea, salmon, T_T

    for(i in 0 until list.count())
        print("${list[i]} ")    //output : korea, salmon, T_T

		val someList = mutableListOf<Int>(1,2,3)
		for( (index,item) in someList.withIndex()){ //index와 item에 각각 index와 value값 들어감
        print("[index: " + index + " item: "+item + "]") //[index: 0 item: 1][index: 1 item: 2][index: 2 item: 3]
    }

		someList.forEach { //it에 someList의 원소가 들어감
        print(""+it+" ") //1 2 3 
    }
}
```

[Conditions and loops | Kotlin](https://kotlinlang.org/docs/control-flow.html#for-loops)

코틀린의 모든 표현식은 label을 추가할 수 있는데, 이 라벨을 사용해 break나 continue를 특정 지을 수 있다.

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

라벨이 부여된 break는 해당 라벨로 표시된 루프 직후에 실행 지점으로 점프한다. continue는 해당 루프의 다음 반복으로 진행된다. 이는 return도 마찬가지로 적용하여 사용할 수 있다.

[Returns and jumps | Kotlin](https://kotlinlang.org/docs/returns.html#return-to-labels)

## 함수, 익명 함수, 람다 표현식

코틀린은 아래와 같이 함수를 선언한다.

```kotlin
fun 함수명(변수): Unit {
}

// or

fun 함수명(변수): 리턴타입 {
	return 리턴 값
}
```

위에서 리턴 타입이 추론 가능하다면 스마트 캐스트로 인해 생략이 가능하다.

코틀린에서 함수는 값으로 표현될 수 있기 때문에 아래처럼 표현도 가능하다.

```kotlin
fun sum(a: Int, b: Int) = a + b
```

### 코틀린 1급 함수

프로그래밍에서 1급 시민이란 다음 조건을 만족하는 것을 말한다.

- 변수에 담을 수 있음
- 함수의 인자로 전달할 수 있음
- 함수의 반환 값으로 전달할 수 있음

1급 객체는 1급 시민을 충족하는 객체를 말하며, 1급 함수는 1급 객체이면서 아래 조건을 추가로 만족하는 함수이다.

- 런타임에 생성 가능
- 익명으로 생성 가능

코틀린의 함수는 1급 함수이다. 즉, 자바와 다르게 코틀린의 함수는 함수 자체가 하나의 변수가 될 수 있다. 변수에 할당할 수 있으며 다른 고차 함수의 인자로 전달되고 반환될 수 있다.

```kotlin
// 1. 변수나 데이터 구조에 할당
val function = { println("first-class test") }

// 2. 함수의 인자로 전달
fun function2(f: () -> Unit) {
    f.invoke()
}

// 3. 함수의 반환값으로 전달
fun function3() : () -> Unit {
    return function
}

fun main() {
    val a = function

    a.invoke()

    function2(a)

    function3().invoke()
}
```

```kotlin
fun add(a: Int, b: Int) = a + b
fun subtract(a: Int, b: Int) = a - b
val functions = mutableListOf(::add, ::subtract) // MutableList에 함수 할당

fun main() {
    println("add : ${functions[0](10, 2)}, sub : ${functions[1](15, 5)}")
    // add : 12, sub : 10
}
```

코틀린은 함수 타입을 가진다. 이는 정수, 실수와 같은 타입을 말하는데 예를 들어 `(Int) -> String`과 같이 사용할 수 있다. 

```kotlin
// 정수를 받아 홀수 인지 확인하는 단일 표현식(Single-expression) 함수
var isOdd(x: Int) : Boolean = x % 2 != 0

// 정수를 받아서 불리언 값을 반환하는 함수 타입 변수를 선언하고,
// 위의 함수를 호출 가능한 참조(Callable reference)(::)를 사용해서 할당
var predicate: (Int) -> Boolean = ::isOdd

// 함수 타입 변수를 사용해서 함수를 호출
println( predicate(1) )
```

### 함수 타입

함수 타입은 아래와 같은 경우가 존재한다.

- `(A, B) -> C`
- `( ) -> C`
- `( ) -> Unit`
- `A.(B) -> C`
    
    ```kotlin
    var x: Int = 2
    
    println( x.plus(4) )
    
    // literal도 자신의 타입이 가진 함수를 사용 가능
    println( 2.plus(4) )
    
    // 수신자를 가지는 함수 타입의 변수 sum을 함수 본체는 함수 타입의 리터럴인 람다식으로 생성
    val sum: Int.(Int) -> Int = { x -> plus(x) }
    println( 2.sum(2) )
    ```
    
- `suspend () -> Unit` 또는 `suspend A.(B) -> C`
- `((Int, Int) -> Int)?`: 함수 타입을 Nullable로 선언
- `(Int) -> ((Int) -> Unit)`: 소괄호를 사용해서 결합 우선 순위를 지정(`(Int) -> (Int) -> Unit`와 같음)
- `typealias`: 반환 타입에 별칭 지정
    
    ```kotlin
    typealias ClickHandler = (Button, ClickEvent) -> Unit
    ```
    

실제로 함수를 인스턴스화 하는 여러가지 예시를 아래에서 보인다.

- 람다 표현식, 익명 함수, 수신자가 있는 함수 리터럴로 할당

```kotlin
var subtract: (Int, Int) -> Int = { a, b -> a - b }
// println(subtract(10, 7))

// 컴파일러가 함수 타입을 추론할 수 있으므로 아래와 같음
var subtract = { a: Int, b: Int -> a - b }

// 익명 함수를 사용해 할당
var subtract = fun(a: Int, b: Int): Int = a - b

// 수신자가 있는 함수 리터럴 사용하여 할당
var subtract: Int.(Int) -> Int = { x -> minus(x) }
// println(15.subtract(9))
```

- 최상위, 멤버 또는 확장 프로퍼티의 호출 가능한 참조를 사용하여 할당

```kotlin
// 최상위 함수
fun isOdd(x: Int) : Boolean {
    return x % 2 != 0
}

// 최상위 함수의 호출가능한 참조를 사용하여 할당
var predicate: (Int) -> Boolean = ::isOdd

// 클래스의 멤버를 할당
var toInt: (String) -> Int  = String::toInt
```

```kotlin
val strs = listOf("a", "b", "c")

val prop = List<String>::size

// Reflection 사용
println(prop.get(strs))
```

- 생성자를 이용한 선언

```kotlin
class Foo

var con = ::Foo

var x: Foo = con()
```

### 람다 표현식

람다 표현식, 익명 함수는 함수 리터럴이다. 즉, 선언되지 않았으나 표현식처럼 즉시 전달되는 함수이다.

```kotlin
max(string, {a, b -> a.length < b.length})
```

람다 표현식은 다음 특징을 가진다.

- 중괄호로 둘러싸여 있음
- 함수 본체는 `->` 다음에 위치
- 람다의 추론된 반환 타입이 `Unit`이 아니면 본문 마지막 표현식이 반환 값으로 처리

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }

// 추론 가능한 반환 타입 생략
val sum = { x: Int, y: Int -> x + y }

// 추론 가능한 매개 변수의 타입 생략
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

람다 표현식은 매개변수로 전달할 때 후행 람다(tailing lambdas)로 전달할 수 있는데, 함수의 마지막 매개변수가 함수라면 이 매개변수는 소괄호 뒤에 위치가 가능하다.

```kotlin
val product = items.fold(1, { acc, e -> acc * e })

// 마지막 매개변수가 함수일 경우
val product = items.fold(1) { acc, e -> acc * e }
```

또한 람다가 유일한 매개변수이면 소괄호도 생략 가능하다.

```kotlin
run({println("java cafe")})

run { println("java cafe") }
```

람다 표현식이 하나의 매개변수만 갖는 경우, 컴파일러가 매개변수 없이 구문 분석이 가능할 때는 매개변수를 선언할 필요가 없고 `->`도 생략 가능하다. 이때 매개변수는 `it`으로 암시적 선언이 된다.

```kotlin
ints.filter {it: Int -> it > 0}

ints.filter { it > 0 }
```

람다에서 return을 사용해 명시적으로 값을 반환할 수 있고, 지정하지 않을 시 마지막 표현식의 값이 반환된다.

```kotlin
ints.filter {
	val filterValue = it > 0
	filterValue
}

// 위 코드와 동일
ints.filter {
	val filterValue = it > 0
	return@filter filterValue
}
```

만약 람다의 매개변수가 사용되지 않을 경우에는 매개변수 이름 대신 underscope(`_`)를 사용할 수 있다

```kotlin
map.forEach { _, value -> println("$value") }
```

람다에서 매개변수는 구조 분해도 가능하다.

```kotlin
map.mapValues { entry: Map.Entry -> "${entry.value}!" }

// Map.Entry의 맴버인 key, value로 분해
map.mapValues { (key, value) -> "$value!" }
```

### 익명 함수

람다 표현식에는 함수의 반환 타입을 지정하는 부분이 빠져있다. 만약 명시적으로 반환 타입을 적어야 할 때 익명 함수를 사용할 수 있다.

```kotlin
fun(a: Int, b: Int): Int = a - b

// or

fun(a: Int, b: Int): Int {
	a - b
}
```

익명 함수에서 함수 본문이 표현식일 경우, 일반 함수와 동일하게 컴파일러가 매개변수 타입을 컨텍스트에서 추론할 수 있다면 생략이 가능하다. 하지만 함수 본문이 블록으로 선언될 경우 명시적으로 선언해야 한다.

### 익명 함수와 람다 표현식 차이

- 익명 함수를 매개변수로 보낼 때 항상 소괄호 안에 위치시켜야 함
- 익명 함수 내의 return은 익명 함수만 반환한다.
    
    ```kotlin
    fun main() {
        val numbers = listOf(1, 2, 3, 4, 5)
        numbers.forEach {
            if (it == 3) return
            println(it)
        }
        println("This line will not be printed.")
    }
    ```
    
    ```kotlin
    fun main() {
        val numbers = listOf(1, 2, 3, 4, 5)
        numbers.forEach(fun(number) {
            if (number == 3) return
            println(number)
        })
        println("This line will be printed.")
    }
    ```
    
    ```kotlin
    fun main() {
        val numbers = listOf(1, 2, 3, 4, 5)
        numbers.forEach label@{
            if (it == 3) return@label
            println(it)
        }
        println("This line will be printed.")
    }
    ```
    
    - 라벨이 없는 `return`은 항상 `fun` 키워드로 선언된 함수로부터 반환한다. 즉, 람다 표현식 내의 `return`은 람다를 둘러싼 함수를 반환하고 익명 함수 내의 `return`은 익명 함수만 반환한다.
    - 만약 람다만 반환하려면 `return@라벨` 을 사용한다.

### 클로저

클로저는 상위 함수의 영역에 있는 변수에 접근할 수 있는 함수를 말한다. 즉, 다른 함수에 자신의 매개변수와 변수를 사용할 수 있게 하는 함수이다.

코틀린은 클로저를 지원하므로 익명 함수는 함수 밖에 정의된 변수에 접근이 가능하다.

```kotlin
fun counter(): () -> Int {
    var i = 0
    
    return { i ++ }
}

fun main() {
    val next = counter()
    
    print(next())
    print(next())
    print(next())
}
// 012
```

코틀린은 클로저를 이용해 `let`, `also`, `apply`, `run`, `with`와 같은 키워드를 제공한다.

```kotlin
val peter = Person().apply {
    name = "Peter"
    age = 18
}
```

예를 들어 `apply`의 경우 확장함수로 Person()을 this로 받아 클로저를 사용해 객체의 프로퍼티를 변경한 뒤 원본 객체에 반영한다.

#### 자바에서는 안되던 기능이 어떻게 되는걸까?

위에서 살펴본 코틀린 코드(`counter` 함수)를 자바로 바꾸면 아래와 같다.

```kotlin
public final class Counter {

    public Function0<Integer> counter() {
        final int i = 0;

        return new Function0<Integer>() {
            @Override
            public Integer invoke() {
                return iΩΩΩΩΩ; // error here
            }
        };
    }
}
```

자바로 작성한 위 코드를 실행하면 동일한 결과를 예상하지만, 그와 달리 에러가 발생한다.

```bash
javac Counter.java
Counter.java:9: error: local variables referenced from an inner class must be final or effectively final
                return i++;
                       ^
1 error
```

왜 이런 일이 발생하는지 먼저 알아보자.

자바에서도 버전 8 이후부터 `final` 혹은 effectively final인 외부 변수는 하위 함수에서 사용할 수 있도록(변경이 아닌 참조만) 기능이 추가되었다.  위 코드에서 캡쳐링을 시도한 변수는 `final` 또는 effectively final이 아니기 때문에 해당에러가 발생한다.

메서드가 실행되면 그 메서드의 지역 변수는 스택에 할당된다. 메서드가 종료되면 이 변수들은 스택에서 삭제되어 접근이 불가능해진다. 하지만 메서드 내부에 정의된 로컬 클래스, 익명 클래스의 인스턴스는 힙 메모리에 할당되어 메서드가 종료된 후에도 힙에 계속 존재하며 객체 참조가 가능한 동안에는 GC에 의해 회수되지 않는다.

익명 클래스나 람다 인스턴스가 외부 함수의 지역 변수에 접근하려 할 때 만약 함수가 종료된 후라면 그 변수가 존재하지 않기 떄문에 문제가 생긴다. 이를 해결하기 위해 자바에서는 지역 클래스나 익명 클래스가 외부 지역 변수를 참조할 때, 그 값을 자신의 인스턴스에 복사하여 함수가 종료된 후에도 계속 사용할 수 있도록 했다. (Variable capture)

하지만 이 지역 변수가 캡쳐 후에 변경될 경우 값이 불일치하기 때문에 문제가 발생할 수 있다. 이를 방지하기 위해 자바는 외부 지역 변수를 `final` 혹은 effectively final로 강제한 것이다.

자바에서 불가능한 이유를 알아 보았다면 다음으로 코틀린에서는 어떻게 가능한지 파악해야 한다.

위 `counter` 함수를 인텔리제이로 디컴파일링하면 아래와 같다.

```java
@NotNull
public static final Function0 counter() {
    final Ref.IntRef i = new Ref.IntRef();
    i.element = 0;
    return (Function0)(new Function0() {
        public Object invoke() {
            return this.invoke();
        }

        public final int invoke() {
            Ref.IntRef var10000 = i;
            int var1;
            var10000.element = (var1 = var10000.element) + 1;
            return var1;
        }
    });
}
```

`kotlin.jvm.internal.Ref.IntRef`는 `int` 를 포함하는 레퍼런스 타입이다. 참조하는 변수가 `final`인데도 불구하고 클로저 내부에서 참조 유형의 상태가 변경될 수 있는 이유는 클로저가 객체에 대한 참조를 캡처하기 때문이다. 따라서 클로저 내부의 참조를 통해 변경된 내용은 참조하는 원래 변수에 직접적인 영향을 미친다. 

그렇다면 클래스에 선언되어 있는 멤버 프로퍼티를 캡쳐링 하는 경우는 어떨까?

```kotlin
class Counter {
    private var i = 0

    fun counter(): () -> Int {
        return { i++ }
    }
}
```

위 코드를 자바로 디컴파일하면 아래와 같다.

```java
public final class Counter {
   private int i;

   @NotNull
   public final Function0 counter() {
      return (Function0)(new Function0() {
         
         public Object invoke() {
            return this.invoke();
         }

         public final int invoke() {
            Counter var10000 = Counter.this;
            int var1;
            var10000.i = (var1 = var10000.i) + 1;
            return var1;
         }
      });
   }
}
```

`Ref.IntRef`를 정의해 캡쳐링 하던 이전과 다르게 Counter 클래스에 대한 참조가 캡쳐된다. `i`를 변경하면 캡쳐된 인스턴스에 보존되므로 `final` 선언을 할 필요가 없어진다. 하지만 이 경우 `invoke()`로 생성한 인스턴스가 동작중이라면 `Counter` 인스턴스가 GC에 의해 삭제되지 않아 메모리 누출이 될 수 있기 때문에 주의해야 한다.