---
title: Kotlin - 고차 함수와 inline (2/2)
author: yoonmin
date: 2024-03-19 00:00:00 +0900
categories: [CS, 프로그래밍 언어]
tags: [Kotlin, Android, 함수형 프로그래밍, 람다, 익명 함수, inline]
render_with_liquid: true
---

![slava-auchynnikau-N1u1b-YKXN0-unsplash 1](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/0f11549b-1681-4805-993f-744a6d804670)

## Intro

저번 고차 함수와 인라인 1편에서 코틀린 함수가 왜 고차 함수인지 함수 타입과 함수 리터럴을 통해 알아봤습니다. 코틀린은 함수에 대한 타입을 제공하고 이를 통해서 함수를 인자로 활용하거나 반환값으로 활용할 수 있습니다. 특히 인자로 활용하는 과정에서 함수 리터럴을 활용하면 가독성이 뛰어난 코드 구현이 가능합니다. 

이런 뛰어난 장점을 가진 고차 함수도 단점은 존재합니다. 이 때문에 코틀린은 고차 함수의 단점을 보완하기 위해 `inline` 이라는 키워드를 제공하여 고차 함수의 단점을 보완하는 데 도움을 제공합니다.

따라서 코틀린이 제공하는 함수형 프로그래밍을 제대로 활용하려면 고차 함수의 장점과 단점이 무엇인지 파악하고 사용해야 합니다. 이번 글에서는 고차 함수가 가지는 장점과 단점이 무엇이며 단점을 해결하기 위한 `inline` 키워드에 대해 알아보겠습니다.

​		

## 고차 함수를 활용해야 하는 이유

#### **1. 코드 재사용성 및 모듈성**

특정 기능을 개별 함수로 분리하여 전역의 여러 부분에서 재사용이 가능합니다. 애플리케이션 전역에서 재사용이 가능하다는 점 덕분에 베이스 코드의 유지 관리가 용이하고 중복성을 줄일 수 있습니다.

전역의 여러 부분에서 사용하는 확장 함수를 보면 이해가 쉽습니다. 우리가 자주 사용하는 고차 함수 특징을 활용한 `forEach` , `map` 등의 확장 함수 베이스 코드는 `Collections.kt` 파일에서 관리합니다.

```kotlin
/* _Collections.kt */

public inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R> {
    return mapTo(ArrayList<R>(collectionSizeOrDefault(10)), transform)
}

public inline fun <T> Iterable<T>.forEach(action: (T) -> Unit): Unit {
    for (element in this) action(element)
}
```

`forEach` , `map` 함수는 둘 다 모든 원소에 대해서 파라미터로 받는 함수 내 정의된 동작을 적용합니다. 따라서 `forEach` , `map` 을 호출하는 지점에서 우리는 각 원소에 적용할 동작 코드만 정의하면 됩니다.

```kotlin
listOf(1,2,3,4,5,6).map { it+1 } // ex) 각 원소에 1을 더하기 위한 map 사용
listOf(1,2,3,4,5,6).forEach { println(it) } // ex) 각 원소를 출력하기 위한 forEach 사용
```

이런 식으로 공통 코드(베이스 코드)를 정의해놓고 필요한 동작만 함수 파라미터로 우리가 작성해서 전달하면 베이스 코드와 필요한 동작 코드가 분리되어 유지 관리가 편리하고 재사용하기 좋습니다.

​		

#### **2. 추상화와 캡슐화를 통한 분리**

함수의 구현 세부사항을 숨기고 필요한 기능만 노출하는 추상화를 가능하게 합니다. 이러한 캡슐화는 보안을 강화하고 복잡한 기능을 단순화합니다. 그리고 관심사를 분리하여 쉬운 유지보수가 가능합니다.

`forEach` , `map` 을 사용하기 위해 필요한 것은 오직 각 원소에 적용할 동작을 정의하는 것입니다. `_Collections.kt` 파일에 구현 세부사항을 정의하고 사용자 정의가 필요한 부분만 함수 파라미터로 받기 때문에 `forEach` , `map` 을 사용하는 우리는 필요한 동작만 정의해서 전달하면 됩니다.

```kotlin
/* _Collections.kt

public inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R> {
    return mapTo(ArrayList<R>(collectionSizeOrDefault(10)), transform)
}

*/

fun main() {
    listOf(1,2,3,4,5,6).map { it+1 }
}
```

​		

#### **3. 가독성 향상**

고차 함수의 람다 표현식을 통해서 보다 표현력이 풍부하고 간결한 코드를 작성할 수 있습니다. 특히 코틀린이 주장하는 코틀린의 강력한 장점중 하나인 함수형 프로그래밍을 위해서 고차 함수의 좋은 가독성은 필수입니다.

코틀린 공식 홈페이지를 보면 코틀린의 장점 다섯 가지를 확인할 수 있는데 그 중 하나가 함수형 프로그래밍 구현입니다. 고차 함수를 활용하면 다음과 같은 간결하고 명료한 코드 작성이 가능합니다.

```kotlin
fun main() {
    // Who sent the most messages?
    val frequentSender = messages
        .groupBy(Message::sender)
        .maxByOrNull { (_, messages) -> messages.size }
        ?.key                                                 // Get their names
    println(frequentSender) // [Ma]

    // Who are the senders?
    val senders = messages
        .asSequence()                                         // Make operations lazy (for a long call chain)
        .filter { it.body.isNotBlank() && !it.isRead }        // Use lambdas...
        .map(Message::sender)                                 // ...or member references
        .distinct()
        .sorted()
        .toList()                                             // Convert sequence back to a list to get a result
    println(senders) // [Adam, Ma]
}

data class Message(                                           // Create a data class
    val sender: String,
    val body: String,
    val isRead: Boolean = false,                              // Provide a default value for the argument
)

val messages = listOf(                                        // Create a list
    Message("Ma", "Hey! Where are you?"),
    Message("Adam", "Everything going according to plan today?"),
    Message("Ma", "Please reply. I've lost you!"),
)
```

실제로 위와 같은 고차 함수를 활용한 메서드 체이닝 기법을 사용하면 무슨 동작을 할 것인지에 대해서만 코드를 작성할 수 있기 때문에 코드 가독성 개선에 좋습니다.

​		

## 고차 함수 활용시 주의사항

> *"Using higher-order functions imposes certain runtime penalties: each function is an object, and it captures a closure. A closure is a scope of variables that can be accessed in the body of the function. Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead."*
>
> **Kotlin docs**

공식 문서에서 말하는 고차 함수 사용시 발생할 수 있는 부작용입니다. 고차 함수를 사용하면 **특정 런타임 페널티**가 부과되는데,  고차 함수가 객체로 전환되며 **클로저**를 이용합니다. 클로저는 함수 본문에서 접근할 수 있는 변수의 범위입니다. **메모리 할당**(함수 객체와 클래스 모두)과 가상 호출은 런타임 오버헤드를 유발합니다.

​		

### 코틀린 클로저?

> *"A lambda expression or anonymous function (as well as a local function and an object expression can access its closure, which includes the variables declared in the outer scope. The variables captured in the closure can be modified in the lambda"*
>
> **Kotlin docs**
>
> *"A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function's scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time."*
>
> **MDN - JavaScript**

코틀린이 설명하는 클로저는 람다 표현식 또는 익명 함수(로컬 함수 및 객체 표현식도 마찬가지)를 사용할 때 외부 범위에서 선언된 변수에 액세스할 수 있는 것을 말합니다.

자바스크립트에서 설명하는 클로저는 함수를 주변 상태(어휘 환경)에 대한 참조와 함께 묶음 결합한 것입니다. 다시 말해, 클로저를 사용하면 내부 함수에서 외부 함수의 범위에 접근할 수 있습니다.

즉, 종합하면 클로저는 외부 범위에 접근할 수 있는 기능이라고 할 수 있습니다. 한 번 생각해 봅시다. 코틀린 반복문인 `forEach` 을 사용하기 위해  함수 리터럴로 동작 코드를 정의할 때 자연스럽게 외부 변수를 가져다 사용한 경험이 있지 않나요? 다음 예시처럼요.

```kotlin
var sum = 0
listOf(1,2,3,4).forEach {
    sum += it
}
println(sum)
```

알고보면 우리는 코틀린을 사용할 때 클로저라는 기능을 지금까지 자연스럽게 사용해 왔습니다. 그렇다면 우린 다음과 같은 생각을 할 수 있습니다.

{: .prompt-tip}

> "그럼 코틀린과 밀접한 자바도 외부 변수에 접근하고 수정도 가능하겠네?"

​		

### 자바 클로저는 코틀린과 같을까?

> *"In Android Java world, you know that when you declare a lambda in a function, you can access the parameters of that function as well as the local variables declared before the lambda. These variables are said to be captured by the lambda, but the variables must be final or effectively final to be captured by lambda. In Kotlin, this constraint has been removed."*
>
> *"안드로이드 자바 세계에서는 함수에서 람다를 선언하면 해당 함수의 매개변수와 람다 앞에 선언된 로컬 변수에 액세스할 수 있다는 것을 알고 계실 겁니다. 이러한 변수는 람다에 의해 캡처된다고 하지만, 람다가 캡처하려면 변수가 `final`이거나 `effectively final`이어야 합니다. Kotlin에서는 이 제약 조건이 제거되었습니다."*
>
> [**How Kotlin lambda capture variable**](https://medium.com/@yangweigbh/how-kotlin-lambda-capture-variable-ef90e11e531d)

자바 클로저는 `enclosing scope` 지역 변수에 접근하려면 해당 변수가 `final` 혹은 `effectively final` 이어야 하는 조건이 있습니다. 람다 표현식이나 익명 함수에서 외부 지역 변수를 사용할 경우, 해당 변수를 **캡쳐해서** 람다 인스턴스 내부에서 사용할 수 있습니다.

```java
String prefix = "prefix";
Executor executor = Executors.newSingleThreadExecutor();
executor.execute(() -> System.out.println(prefix + "hello world"));
```

```java
// $FF: synthetic class
public final class -$$Lambda$Foo$SO0lUY_bGsXMRCTRPJJFcOU0yQM implements Runnable {
   // $FF: synthetic field
   public final String f$0;

   // $FF: synthetic method
   public _$$Lambda$Foo$SO0lUY_bGsXMRCTRPJJFcOU0yQM/* $FF was: -$$Lambda$Foo$SO0lUY_bGsXMRCTRPJJFcOU0yQM*/(String var1) {
      this.f$0 = var1;
   }

   public final void run() {
      System.out.println(this.f$0 + "hello world");
   }
}
```

위 코드를 보면 `f$0`  멤버 변수가 외부에서 캡쳐한 값을 저장합니다. 그 밑을 보면 외부에서 캡쳐한 정보를 `var1` 로 가져와서 `f$0` 에 저장하는 것을 볼 수 있습니다. 굳이 람다 인스턴스 내에서 멤버 변수를 따로 만들어서 외부 지역 변수 값을 복사하는 이유가 뭘까요? 

람다 인스턴스는 힙에서 관리가 되지만 외부 지역 변수를 포함한 메서드는 스택에서 관리됩니다. 메모리 관리 특성상 메서드가 끝난 이후에도 힙은 계속 남아 있을 수 있기 때문에 람다 내에서 변수를 호출했을 때 변수가 존재하지 않을 가능성이 있습니다. 그래서 람다 인스턴스 내에 따로 멤버 변수를 둬서 해당 문제를 방지하는 것입니다.

그런데 인스턴스 내부에 변수를 두고 사용하는 부분에서 위험한 점이 존재합니다. 위에서 람다 인스턴스 내부에서 사용하는 변수는 복사하려는 외부 지역 **변수를 캡쳐한 것**이라고 언급했습니다. 

**캡쳐한 시점의 값을 사용하고 있는데 외부 지역 변수의 값이 변경된다면 값이 일치하지 않는 시점이 생깁니다.** 이 부분은 개발자에게 큰 혼란을 야기하기 때문에 자바는 이 상황에서 외부 지역 변수를 `final` 혹은 `effectively final` 로 강제해서 값이 다른 상황을 방지합니다.

하지만 코틀린은 자바와 다르게 외부 지역 변수 수정이 가능합니다. 코틀린은 함수형 프로그래밍 구현을 강력하게 지원하기 때문에 가능하면 람다나 익명 함수를 이용한 유연한 코드 처리를 지원합니다.

​		

- `val` 키워드를 이용하여 불변 변수를 람다 표현식에 이용할 때

  ```kotlin
  val prefix = "Hello"
  val threadPool = Executors.newSingleThreadExecutor()
  threadPool.execute {
      println("$prefix world")
  }
  ```

  ```java
  final String prefix = "Hello";
  ExecutorService threadPool = Executors.newSingleThreadExecutor();
  threadPool.execute((Runnable)(new Runnable() {
     public final void run() {
        String var1 = prefix + " world";
        System.out.println(var1);
     }
  }));
  ```

- `var` 키워드를 이용하여 가변 변수를 람다 표현식에 이용할 때

  ```kotlin
  var prefix = "Hello"
  val threadPool = Executors.newSingleThreadExecutor()
  threadPool.execute {
      prefix += " my"
      println("$prefix world")
  }
  ```

  ```java
  final Ref.ObjectRef prefix = new Ref.ObjectRef();
  prefix.element = "Hello";
  ExecutorService threadPool = Executors.newSingleThreadExecutor();
  threadPool.execute((Runnable)(new Runnable() {
     public final void run() {
        Ref.ObjectRef var10000 = prefix;
        String var10001 = (String)var10000.element;
        var10000.element = var10001 + " my";
        String var1 = (String)prefix.element + " world";
        System.out.println(var1);
     }
  }));
  
  /* Hello my world */
  ```

​		

불변 변수를 이용할 때는 `final` 키워드가 붙는 자바 방식으로 처리가 되지만 가변 변수를 사용해서 람다 표현식 내에 수정 작업을 하면 `final` 이 붙지 않고 `Ref` 라는 객체 참조 클래스를 이용하여 외부 지역 변수와 연결합니다. 이 덕분에 람다 표현식 내에서도 외부 지역 변수 수정이 가능합니다.

```java
public class Ref {
    private Ref() {}

    public static final class ObjectRef<T> implements Serializable {
        public T element;

        @Override
        public String toString() {
            return String.valueOf(element);
        }
    }

    public static final class ByteRef implements Serializable {
        public byte element;

        @Override
        public String toString() {
            return String.valueOf(element);
        }
    }

    public static final class ShortRef implements Serializable {
        public short element;

        @Override
        public String toString() {
            return String.valueOf(element);
        }
    }

    public static final class IntRef implements Serializable {
        public int element;

        @Override
        public String toString() {
            return String.valueOf(element);
        }
    }

    public static final class LongRef implements Serializable {
        public long element;

        @Override
        public String toString() {
            return String.valueOf(element);
        }
    }

    public static final class FloatRef implements Serializable {
        public float element;

        @Override
        public String toString() {
            return String.valueOf(element);
        }
    }

    public static final class DoubleRef implements Serializable {
        public double element;

        @Override
        public String toString() {
            return String.valueOf(element);
        }
    }

    public static final class CharRef implements Serializable {
        public char element;

        @Override
        public String toString() {
            return String.valueOf(element);
        }
    }

    public static final class BooleanRef implements Serializable {
        public boolean element;

        @Override
        public String toString() {
            return String.valueOf(element);
        }
    }
}
```

​		

### 고차 함수는 객체 전환 비용이 발생한다

클로저 다음으로 알아야 할 것은 고차 함수의 객체 전환 비용입니다. 코틀린은 고차 함수가 사용될 경우, 인자나 반환값으로 활용되는 함수를 객체로 변환합니다. 그래서 고차 함수를 활용하면 필연적으로 메모리 할당에 대한 오버헤드가 따라올 수밖에 없습니다.

말로만 들으면 이해가 잘 되지 않으니 예시로 첫 번째로 함수를 인자로 활용하는 경우를 보겠습니다. 다음과 같이 `func` 이름의 함수가 함수를 인자로 받아서 이를 내부에서 실행하는 코드가 있습니다.

```kotlin
fun func(lambda: () -> Unit) {
    lambda()
}
```

그리고 이 함수를 다음과 같이 `main` 영역에서 활용합니다. 이를 실행하면 예상대로 `sum` 은 `1` 이 되어 출력됩니다.

```kotlin
fun main() {
    var sum = 0
    func {
        sum += 1
        println(sum)
    }
}
```

코틀린 코드는 위와 같습니다. 그렇다면 코틀린에서 자바 코드로 변환될 때는 코드가 어떻게 될까요? 디컴파일을 하면 다음과 같습니다.

```java
public static final void main() {    
		final Ref.IntRef sum = new Ref.IntRef();
    sum.element = 0;
    func((Function0)(new Function0() {
       // $FF: synthetic method
       // $FF: bridge method
       public Object invoke() {
          this.invoke();
          return Unit.INSTANCE;
       }

       public final void invoke() {
          ++sum.element;
          int var1 = sum.element;
          System.out.println(var1);
       }
    }));
}

public static final void func(@NotNull Function0 lambda) {
    Intrinsics.checkNotNullParameter(lambda, "lambda");
    lambda.invoke();
}
```

고차 함수인 `func` 의 파라미터(함수를 인자로 받는)가 `Function0` 이라는 객체로 전환된 것을 알 수 있습니다. 실제로 메인 영역에서 람다 표현식을 전달할 때 `new Function0()` 코드로 인스턴스를 생성합니다.

이번에는 함수를 반환값으로 활용하는 경우를 보겠습니다. 함수를 인자로 활용하는 예시와 마찬가지로 함수를 반환하는 `func` 함수를 정의하고 메인 영역에서 호출합니다. 결과는 예상대로 `2` 가 출력됩니다.

```kotlin
fun main() {
		val adder = func()
    println(adder(1))
}

fun func() = { n: Int -> n + 1  }
```

이를 자바 코드로 변환하면 다음과 같습니다. `func` 의 반환되는 함수가 `Function1` 로 전환이 됐고 해당 인스턴스를 반환합니다. 해당 인스턴스를 받은 메인 영역은 `invoke` 메서드를 통해 정의했던 함수 - `{ n: Int -> n + 1  }` 를 실행합니다.

```java
public static final void main() {
  Function1 adder = func();
  int var1 = ((Number)adder.invoke(1)).intValue();
  System.out.println(var1);
}

@NotNull
public static final Function1 func() {
  return (Function1)null.INSTANCE;
}
```

위의 두 가지 예시를 통해서 우리는 이제 코틀린 공식 문서가 말하는 고차 함수의 런타임 오버헤드가 무엇인지 알 수 있습니다. 고차 함수를 활용하면 인자나 반환값으로 활용되는 함수를 객체로 전환하고 심지어 외부 변수 사용에 대한 클로저도 객체를 사용합니다.

​		

## 인라인 함수

> *"But it appears that in many cases this kind of overhead can be eliminated by inlining the lambda expressions."*
>
> **Kotlin docs**

코틀린은 이러한 문제점을 해결하기 위해 전달되는 인자를 포함해서 고차 함수의 코드를 호출 지점에 인라이닝할 수 있는 기능을 제공합니다. 이것이 `inline` 키워드입니다.

 `inline` 키워드는 `fun` 앞에 붙일 수 있으며 인라인으로 지정한 함수는 호출 지점에 본인의 코드를 붙여넣습니다. 이를 인라이닝이라고 합니다.

​		

### 인라이닝

위에서 예시로 사용했던 `func` 함수를 다시 보겠습니다. 다음과 같이 함수를 인자로 받고 내부에서 인자로 받은 함수를 실행하는 코드를 가지고 있습니다.

```kotlin
fun main() {
    var sum = 0
    func {
        sum += 1
        println(sum)
    }
}

fun func(lambda: () -> Unit) {
    lambda()
}
```

이것을 `inline` 함수로 바꾸고 몇 개의 출력문 코드를 추가하겠습니다.

```kotlin
inline fun func(lambda: () -> Unit) {
    println("before lambda()")
    lambda()
    println("after lambda()")
}
```

이제 `func` 함수는 인라인 함수가 됐습니다. 자바 코드로 디컴파일 했을 때 코드가 어떻게 변했는지 보겠습니다.

```java
/* inline 키워드를 사용한 경우 */
public static final void main() {
  int sum = 0;
  int $i$f$func = false;
  String var2 = "before lambda()";
  System.out.println(var2);
  int var3 = false;
  ++sum;
  System.out.println(sum);
  var2 = "after lambda()";
  System.out.println(var2);
}

/* inline 키워드를 사용하지 않은 경우 */
public static final void main() {
  final Ref.IntRef sum = new Ref.IntRef();
  sum.element = 0;
  func((Function0)(new Function0() {
     // $FF: synthetic method
     // $FF: bridge method
     public Object invoke() {
        this.invoke();
        return Unit.INSTANCE;
     }

     public final void invoke() {
        ++sum.element;
        int var1 = sum.element;
        System.out.println(var1);
     }
  }));
}
```

인라인 키워드를 사용하게 되면 함수 표현식을 객체에 담아서 전달할 필요 없이 오히려 표현식을 인자로 받는 `func` 의 코드를 호출 지점으로 불러옵니다.

원래라면 함수를 호출하고 준비물로 람다 표현식을 전달하는데 이 과정에서 메모리 관련 오버헤드가 발생하므로 함수 호출 대신에 함수의 내부 코드를 불러와서 오버헤드를 방지하는 것입니다.

​		

### 인라인 키워드 사용시 주의점

#### 자바 바이트 코드 증가

오버헤드를 방지한다는 점에서 인라인 키워드는 굉장히 매력적으로 다가옵니다. 그러나 인라인 키워드를 사용하면 호출할 함수의 코드를 호출 지점으로 복사 붙여넣기를 하기 때문에 **자바 바이트 코드가 증가**한다는 단점이 있습니다. 

그래서 인라인 키워드는 코드량이 많지 않은 함수에 사용하는 것이 효율적입니다. 코틀린 공식 문서는 크기가 적당한 함수를 인라이닝하면 성능 면에서 이점을 얻을 수 있다고 말합니다.

> *"Inlining may cause the generated code to grow. However, if you do it in a reasonable way (avoiding inlining large functions), it will pay off in performance, especially at "megamorphic" call-sites inside loops."*
>
> **Kotlin docs**

​		

#### 함수 타입 파라미터를 사용하는 경우가 아니면 고려가 필요

또한 인라인은 함수를 인자로 받는 경우에 사용하는 것이 가장 효율적입니다. 그 이외의 경우에 인라인 키워드를 사용하는 것은 크게 효율적이지 않을 수 있기 때문에 개인이 잘 판단하면 될 것 같습니다.

실제로 함수 파라미터가 없거나 `reified` 타입 파라미터가 없는데 `inline` 키워드를 사용하는 경우 경고 문구가 뜹니다. 그래도 `inline` 키워드를 사용하고자 경고 문구를 삭제하고 싶다면 다음 주석을 추가하면 됩니다.

```kotlin
@Suppress("NOTHING_TO_INLINE") 
```



​		

## 인라인 관련 추가 기능

### noinline

일반적으로 인라인 함수에 전달되는 함수 인자는 모두 인라인 처리됩니다. 그러나 특정 함수 인자를 인라인 처리하고 싶지 않다면 `noinline` 을 앞에 추가하면 됩니다.

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) { ... }
```

인라인 처리된 람다 표현식은 인라인 함수 내부에서만 호출하거나 인라인 인자로 전달할 수 있습니다. 그러나 비인라인 처리가 된 람다 표현식은 필드에 저장하거나 전달되는 등 원하는 방식으로 조작할 수 있습니다.

​		

### crossinline







​		

## 참조

[**Android Interview Questions: 7 - What are the High-order functions in Kotlin?**](https://medium.com/@dawinderapps/android-interview-questions-7-what-are-the-high-order-functions-in-kotlin-695f7c8c3dee)

[**How Kotlin lambda capture variable**](https://medium.com/@yangweigbh/how-kotlin-lambda-capture-variable-ef90e11e531d)

[**Higher-order functions and lambdas**](https://kotlinlang.org/docs/lambdas.html)

[**Inline functions**](https://kotlinlang.org/docs/inline-functions.html)

[**First-Class and Higher-Order Functions**](https://medium.com/@rabailzaheer/first-class-and-higher-order-functions-86d14e40c688)

[**클로저(Closure): Java와 Kotlin 비교**](https://jaeyeong951.medium.com/java-%ED%81%B4%EB%A1%9C%EC%A0%80-vs-kotlin-%ED%81%B4%EB%A1%9C%EC%A0%80-c6c12da97f94)

