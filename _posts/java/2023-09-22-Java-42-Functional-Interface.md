---
title:  "[Java] Java의 표준 함수형 인터페이스"
excerpt: "함수형 인터페이스의 종류와 설명"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[Java]"
 
date: 2023-09-22
last_modified_at: 2023-09-22
---

- 참고사이트
  - [블로그](https://ttl-blog.tistory.com/207){:target="_blank"}



### 함수형 인터페이스란?

- 1개의 추상 메소드를 갖는 인터페이스
- Java 의 람다 표현식은 함수형 인터페이스로만 사용 가능하다.


### Java의 표준 함수형 인터페이스

|계열|입력|출력|메소드|설명|
|------|------|------|------|------|
|Consumer|O|X|void accept(T)|입력을 소비|
|Supplier|X|O|T get()|출력을 공급|
|Function|O|O|T apply(R)|입력 -> 출력 함수 매핑|
|Operation|O|O|T apply(T)|입력을 연산하여 동일 타입의 출력으로 리턴|
|Predicate|O|boolean|boolean test(T)|입력을 판단|



매개변수가 2개인 경우 접두사 Bi 를 붙인다.


|함수형 인터페이스|메소드|설명|
|------|------|------|
|BiConsumer<T, U>|void accept(T t, U u)|두 개의 매개변수만 있고, 반환 값이 없다.|
|BiFunction<T, U, R>|R apply(T t, U u)|일반적인 함수. 두개의 매개변수를 받아서 결과를 반환|
|BiPredicate<T, U>|boolean test(T t, U u)|조건식을 표현하는데 사용. <br>매개변수 2개, 반환 타입은 boolean|




### Consumer 계열

> 파라미터를 입력 받아 그것을 소비하는 함수형 인터페이스
<br>
소비라는 것은 함수가 이용된다고 생각하면 된다. 리턴이 되지 않고 함수 내에서 사용이 되고 없어진다.


- 참고로 Consumer 는 void 로 반환값이 없지만, 람다의 body에 일반 표현식이 있으면 호환이 가능하다.


```
특별한 void 호환 규칙

람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다.
(물론 파라미터 리스트도 호환되어야 함). 
예를 들어 다음 두 행의 예제에서 List의 add 메소드는 
Consumer 콘텍스트(T -> void)가 기대라는 void를 반환하지만 유효한 코드다.

//Predicater는 불리언 반환값을 갖는다
Predicater<String> p = s -> list.add(s);

//Consumer는 void 반환값을 갖는다.
Consumer<String> b = s -> list.add(s);
```



|인터페이스|메소드|
|------|------|
|Consumer<T>|void accept(T t)|
|BiConsumer<T, U>|void accept(T t, U u)|
|IntConsumer|void accept(int value)|
|LongConsumer|void accept(long value)|
|DoubleConsumer|void accept(double value)|
|ObjIntConsumer<T>|void accept(T t, int value)|
|ObjLongConsumer<T>|void accept(T t, long value)|
|ObjDoubleConsumer<T>|void accept(T t, double value)|




```java
import java.util.function.*;

public class Main {
    public static void main(String[] args) {
        Consumer<String> consumer = (s)-> System.out.println(s);
        consumer.accept("A String.");

        BiConsumer<String, String> biConsumer = 
                         (t,u) -> System.out.println(t+","+u);
        // <String,String> 이므로 모두 String 전달
        biConsumer.accept("Hello","world");
        

        // 오토박싱/ 언방식 사용하면 비효율적이다.
        Consumer<Integer> integerConsumer = (x) -> System.out.println(x);
        integerConsumer.accept(10); // 값이 들어갈 땐 오토박싱 출력할 때 언박싱


        IntConsumer intConsumer = (x) -> System.out.println(x);
        intConsumer.accept(5); 



        // t는 <>안에 값 x는 objIntconsumber의 int의 자료형이 들어간다.
        ObjIntConsumer<String> objIntConsumer = 
                       (t,x) -> System.out.println(t + ": "+ x);
        objIntConsumer.accept("x",1024);
        // ObjLongConsumer,ObjDoubleConsumer
        // 총 4가지 타입이 있다.
    }
}

```




### Supplier 계열

- 아무런 입력을 받지 않고, 값을 하나 반환하는 함수형 인터페이스이다.
- 자료를 '공급'하는 공급자 역학을 한다.
- 외부로 데이터를 리턴해주는 역할을 한다.



|인터페이스|메소드|
|------|------|
|Supplier<T>|T get()|
|BooleanSupplier|boolean getAsBoolean()|
|IntSupplier|int getAsInt()|
|LongSupplier|long getAsLong()|
|DoubleSupplier|double getAsDouble()|




```java
import java.util.function.BooleanSupplier;
import java.util.function.IntSupplier;
import java.util.function.Supplier;

public class Main {
    public static void main(String[] args) {
        Supplier<String> supplier = () -> "A String";

        System.out.println(supplier.get()); 
        // get()을 해서 출력을 한다.
        // BiSupperlier는 입력은 여러 개 할 수 있지만, 
        // 출력은 하나 밖에 못하기 때문에 없다.

        BooleanSupplier boolsup = () -> true;
        System.out.println(boolsup.getAsBoolean()); 
        // 이것은 getAsBoolean()으로 출력한다.
        // IntSupplier, LongSupplier, DoubleSupplier

        IntSupplier rollDice = () -> (int)(Math.random() * 6);
        for (int i = 0; i < 10; i++) {
            System.out.println(rollDice.getAsInt());
        }

        int x = 4;
        IntSupplier intSupp = () -> x; //지역변수에도 접근할 수 있다.
        // 람다식을 활용할 때 모든 변수에 접근하여 활용할 수 있다.
        // 고정되어있는 값뿐만아니라 동적으로도 주변 값들을 공급할 수 있다.
        System.out.println(intSupp.getAsInt());

    }
}
```



### Function 계열

- Mapping : 입력 -> 출력을 연결하는 함수형 인터페이스
- 입력 타입과 출력 타입은 다를 수도, 같을 수도 있다.
- 매개값을 리턴값으로 매핑(타입변환)하는 역할을 한다.


|인터페이스|메소드|
|------|------|
|Function<T, R>|R apply(T t)
|BiConsumer<T, U, R>|R apply(T t, U u)|
|[P](Primitive Type)Function<R> (예시 : IntFunction<R>)|R apply(p value)|
|[P](Primitive Type)To[Q](Primitive Type)Function (예시 : ntToLongFunction)|q applyAsQ(p value) <br> (예시: long applyAsLong(int value))|
|To[P](Primitive Type)Function<T> (예시 : ToIntFunction)|p applyAsP(T t)|
|To[P](Primitive Type)BiFunction<T, U> (예시 : ToIntBiFunction)|p applyAsP(T t, U u)|



```java
public class Main {
    public static void main(String[] args) {
    
        Function<String,Integer> func = (s) -> s.length();
        // s 는 String타입, s.length() 는 Integer
        
        System.out.println(func.apply("Strings")); //apply로 출력한다

        // Bi가 붙으면 2개의 값 입력
        BiFunction<String,String,Integer> biFunction = (s,u) -> s.length() + u.length();
        System.out.println(biFunction.apply("one","two")); //6

        // IntFunction<R>은 리턴 자료형
        // P type Funtion은 입력을 P타입으로 받는다.
        IntFunction<String> intFunction = (value) -> String.valueOf(value);// "" + value도 가능.
        System.out.println(intFunction.apply(512));

        //ToP Type Function은 출력을 P타입으로 한다.
        ToIntFunction<String> funcFour = (s) -> s.length(); // 4:21
        System.out.println(funcFour.applyAsInt("abcde"));
        // 출력이 P타입인 경우에는 AsP가 들어간다.!!!
        //ToIntBiFunction<String,String>// int 출력을 하는 Bi 함수
        // P: Int, Long, Double

        // int 에서 double로 바꾸는 함수 PToQFunction : P -> Q로 매핑하는 함수
        IntToDoubleFunction funcfive;
        // IntToIntFunction은 없다. 동일한 것에 대해서는 다른게 있다.
    }
}

```


### Operator 계열

- 입력받은 타입과 동일한 타입의 출력을 하는 함수형 인터페이스
- Funtion 계열과 달리 입출력 타입이 다를 수 없다.
- applyXXX() 메소드를 가지고 있고 매개값을 리턴값으로 매핑하는 역할보다는
매개값을 이용해서 연산을 수행한 후 동일한 타입으로 리턴값을 제공하는 역할을 한다.


|인터페이스|메소드|
|------|------|
|UnaryOperator<T>|T apply(T t)|
|BinaryOperator<T>|T apply(T t1, T t2)|
|IntUnaryOperator|int applyAsInt(int value)|
|LongUnaryOperator|long applyAsLong(long value)|
|DoubleUnaryOperator|double applyAsDouble(double value)|
|IntBinaryOperator|int applyAsInt(int value1, int value2)|
|LongBinaryOperator|long applyAsLong(long value, long value2)|
|DoubleBinaryOperator|double applyAsDouble(double value, double value2)|


참고로 unary -> 단항이라는 뜻이다


```java
import java.util.function.BinaryOperator;
import java.util.function.IntBinaryOperator;
import java.util.function.IntUnaryOperator;
import java.util.function.UnaryOperator;

public class Main {
    public static void main(String[] args) {
        // 그냥 operator는 없다.
        // 입력이 1개 인 것을 Unary를 붙여서 표현
        UnaryOperator<String> operator = s -> s+"."; 
        // 리턴타입을 따로 입력받지 않는다 입출력이 같으니깐
        System.out.println(operator.apply("왔다")); // apply() 사용.

        BinaryOperator<String> operator1 = (s1,s2) -> s1 + s2;
        // 타입은 하나만 입력받게 되어있다. 출력은 동일한 타입이여야 하니깐?
        System.out.println(operator1.apply("나","왔다"));

        IntUnaryOperator op = value -> value*10; 
        //타입을 받지 않는다 어차피 int입력 int출력이니
        System.out.println(op.applyAsInt(5));
        // LongUnaryOperator, DoubleUnaryOperator

        IntBinaryOperator ibo = (v1,v2) -> v1 * v2;
        System.out.println(ibo.applyAsInt(10,20));
        //LongBinaryOperator, DoubleBinaryOperator
    }
}
```



### Preicate 계열

- 논리 판단을 해주는 함수형 인터페이스
- 입력을 받아서 boolean 타입 출력을 반환한다.
- 매개변수와 boolean 리턴값이 있는 testXXX() 메소드를 가지고 있다.


|인터페이스|메소드|
|------|------|
|Predicate<T>|boolean test(T t)|
|BiPredicate<T, U>|boolean test(T t, U u)|
|IntPredicate|boolean test(int value)|
|LongPredicate|boolean test(long value)|
|DoublePredicate|boolean test(double value)|


```java
import java.util.function.BiPredicate;
import java.util.function.IntPredicate;
import java.util.function.Predicate;

public class Main {
    public static void main(String[] args) {
        Predicate<String> predicate = (s) -> s.length() == 4; 
        // 조건식이 들어가야 한다.
        System.out.println(predicate.test("Four")); 
        // test()를 사용한다 true or false 값 출력
        System.out.println(predicate.test("six"));

        BiPredicate<String, Integer> pred2 = (s,v) -> s.length() ==v;
        System.out.println(pred2.test("abcd",23));
        System.out.println(pred2.test("abc",3));

        IntPredicate pred3 = x -> x > 0;
        //LongPredicate, DoublePredicate asP출력은 존재하지 않는다.

    }
}
```