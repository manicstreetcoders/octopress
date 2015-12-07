---
layout: post
title: "Java Generic Array"
date: 2013-09-05 15:59
comments: true
categories: 
---
[Generic Array 를 new 할 수 없는 이유](http://stackoverflow.com/a/2931240)

    It's because Java's array (unlinke generics) contain, at runtime, information
    about its component type. So you must know the component type when you create
    the array. Since you don't know what `E` is at runtime, you can't create the
    array.

테스트해본다.

    public class GenericSet<E> {
        private E a[];
        public GenericSet() {
            a = new E[1];
        }
    }

역시 에러가 난다.

    code $ !javac
    javac GenericSet.java
    GenericSet.java:4: error: generic array creation
            a = new E[1];
                ^
    1 error

Effective Java 를 찾아보았다.

Generic 의 array 생성이 허용되면 발생할 문제를 적어놓았다.

    import java.util.List;
    import java.util.Arrays;

    // Why generic array creation is illegal - won't compile!
    List<String>[] stringLists = new List<String>[1];   // (1) 실제 컴파일하면 에러
    List<Integer> intList = Arrays.asList(42);          // (2)
    Object[] objects = stringLists;                     // (3)
    objects[0] = intList;                               // (4)
    String s = stringList[0].get(0);                    // (5)

- Line 3: legal (Arrays are covariant.)
- Line 4: legal (The runtime type of a `List<Integer>` instance is `List`, and the runtime type of a List<Integer>[] instance is `List[]`, and the runtime type of a `List<String>` instance is `List[]`.)
- Line 5: ClassCastException (The compiler automatically casts the retrieved element to `String`, but it's an `Integer`.)

[Type Erasure: 컴파일타임때의 Generic 타입 정보가 runtime 때 사라지기때문에](http://docs.oracle.com/javase/tutorial/java/generics/erasure.html)

즉, `List<String>` 과 `List<Integer>` 의 runtime type 은 모두 `List` 이다.

Java documentation 에서는 [Non-Reifiable Types](http://docs.oracle.com/javase/tutorial/java/generics/nonReifiableVarargsType.html#non-reifiable-types) 라고 부르고 있다.

Stackoverflow 의 질문: [Java how to: Generic Array creation](http://stackoverflow.com/questions/529085/java-how-to-generic-array-creation)

Typecasting 을 해보았다.

    public class GenericSet<E> {
        private E a[];
        public GenericSet() {
            a = (E[]) new Object[1];
        }
    }
    code $ !j
    javac GenericSet.java
    Note: GenericSet.java uses unchecked or unsafe operations.
    Note: Recompile with -Xlint:unchecked for details.

unchecked warning 이 나온다.

`@SuppressWarnings({"unchecked"})` 을 사용.

    @SuppressWarnings({"unchecked"})
    public class GenericSet<E> {
        private E a[];
        public GenericSet() {
            a = (E[]) new Object[1];
        }
    }
    code $ !j
    javac GenericSet.java

Generic Array Creation 에 대해서 너무 타이트하게 컴파일타임에 규제하는게 아닌가 싶다.

어쨌든 Reflection 을 사용해서 Generic type array 를 만들어 본다.
