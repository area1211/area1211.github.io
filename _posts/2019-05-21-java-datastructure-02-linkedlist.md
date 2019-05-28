---
title: "[Java][자료구조]Linked List"
date: 2019-05-21 15:05:28 -0400
categories: jekyll update
---
출처
* [https://www.geeksforgeeks.org/implementing-a-linked-list-in-java-using-class/](https://www.geeksforgeeks.org/implementing-a-linked-list-in-java-using-class/)
* [https://stackoverflow.com/questions/41438392/why-class-node-in-linkedlist-defined-as-static-but-not-normal-class](https://stackoverflow.com/questions/41438392/why-class-node-in-linkedlist-defined-as-static-but-not-normal-class)

Linked List는 선형 자료 구조이다. 배열과 달리 요소들은 연속된 위치에 저장되지 않는다. 각 요소들은 양방향 포인터를 사용해서 연결된다.

LinkedList 내부의 Node 클래스는 왜 static 으로 정의 되어 있는 걸까?

``` java
public class Main{

  private String aField = "test";

  public static void main(String... args) {

    StaticExample x1 = new StaticExample();
    System.out.println(x1.getField());


    // 컴파일되지 않는다.
    // NonStaticExample x2 = new NonStaticExample();

    Main m1 = new Main();
    NonStaticExample x2 = m1.new NonStaticExample();
    System.out.println(x2.getField());

  }

  private static class StaticExample {
    String getField(){
        // 컴파일 되지 않는다.
        return aField;
    }
  }

  private class NonStaticExample {
    String getField(){
        return aField;
    }
  }
```

static class 로 정의 된 경우, 외부 클래스를 인스턴스화 하지 않고 바로 인스턴스를 생성할 수 있다. 그리고 외부 클래스의 인스턴스 필드에 접근할 수 없다.
nonstatic class 로 정의 된 경우, 외부 클래스에 대해 먼저 인스턴스를 만들어야 Non-static nested class의 인스턴스를 만들 수 있고 외부 클래스의 인스턴스 필드에 접근할 수 있게 된다.

LinkedList의 경우 Node 클래스는 외부 클래스인 LinkedList의 필드에 접근할 필요가 없다. 그렇다고 Node 클래스를 별도의 클래스 파일로 분리시키는 것은 make no sense 하다.
따라서 Node 클래스를 static nested class로 만든 것은 가장 sensible한 design choice 이었던 것이다.

추가. 코멘트에 다음과 같은 글이 있었다.
it's static because a Node<E> of one List<E> can be the same type as the nodes from every other List<E>. 
한 List<E>이 Node<E>는 다른 모든 List<E>로 부터의 Node들과 같은 타입이 될 수 있기 때문이다.
The type does not need to be bound to a single List<E> instance. 
즉, Node 클래스를 nonstatic으로 만들어서 특정 List<E>에 bound 되게 할 필요가 없다는 것이다.