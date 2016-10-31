---
layout: post
title:  "Java-Collection[1] "
date:   2016-10-26 12:32:00 +0200
tags: ['Java', 'Collection']
author: "Jang chulwoon"
---

## Collection 

`본 글은 Java Collection Interface를 정리한 글입니다.`  

세부 내용은 [Java doc](https://docs.oracle.com/javase/8/docs/api/)에서 확인하세요.  

+ Collection 이란  
여러 객체의 그룹으로, Collection 인터페이스는 Collection 계층의 **root interface** 입니다.    
일반적으로 JDK에서는 Collection 인터페이스를 직접적으로 구현하는 것을 제공하지 않고, **Set , List** 같이 서브인터페이스로 구현을 제공합니다.  


`collection의 subInterface`   

> BeanContext, BeanContextServices, BlockingDeque<E>, BlockingQueue<E>, Deque<E>, List<E>....   

> (+) Multisets은 직접적으로 인터페이스를 구현해야합니다.  

+ Collection을 구현하는서브인터페이스는 2개의 생성자를 구현해야 합니다.

 default 생성자와 Collection Type의 단일 인자를 받는 생성자 입니다. 인자를 받는 경우 collection을 복제 할 수 있어 원하는 타입의 동등한 collection을 생성 할 수 있습니다. 이 두가지 생성자가 강요되는 것은 아니지만 모든 Java Platform Library의 Colletion Interface는 이를 준수하고 있습니다.
  
``` 
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }   
```

>Collection의 서브인터페이스인 List를 구현한 ArrayList입니다.   
>ArrayList는 3가지의 생성자를 갖고 있는데, 그 중 2가지의 생성자는 위에서 말씀드린 생성자 입니다.   
>2번째 생성자는 collection type의 인자를 받아 복제하여 사용하는 코드입니다.   

+ 만약 collection이 작동을 지원하지 않는다면 UnsupportedOperationException을 던집니다. 
>기능을 지원하지 않는 경우 발생하는 Exception    

+ 몇몇 collection 인터페이스는 인자에 대한 제한을 갖고 있습니다.   
null 인자를 금지한다거나, 그들의 속성값의 제한을 합니다.  
만약 자격이 없는 인자를 추가하려고 시도한다면 확인되지 않은 exception을 던질것입니다.

> 일반적으로 nullpointerException(null 값 문제) 이나 ClassCastException(캐스팅 문제)을 던집니다.  
> 위 생성자를 보시면 collection의 인자가 Null일 경우 nullpointerException을 던진다고 합니다.


## Method  

   
> Java 8에 추가된 default method는 제외하고 정리해 보았습니다.      
    
+ boolean add(E e)     
Collection은 제네릭을 이용하여 선언됩니다. ( ex - List<Integer> list )     
해당 타입의 변수를 넘겨 받아 Collection에 추가하는 Method 입니다. ( ex - list.add(5) )     
   
> 의미 그대로 element를 삽입하는 Method 입니다.   
   
+ boolean addAll(Collection<? extends E> c)      
> method를 정리하기 전에 (Collection<? extends E> c) 부터 정리하겠습니다.    
    
추후 정리할 내용인 제네릭과 관련있는 부분입니다 ( ?를 **와일드카드 타입**이라 부릅니다. ).     
class 상속받을때 쓰는 extends 의미 그대로,  <? extends E> 는 E를 상속 받은 타입을 말합니다.     
즉 E 의 하위타입 또는 E 타입의 인자를 받겠다는 뜻입니다.      
만약 <? extends List> 로 쓰인다면 List 의 하위 클래스인 ArrayList , Linked List 등의 타입이 인자로 올 수 있습니다.     
다만, 상위 클래스에서 정의된 메소드만 사용 할 수 있습니다.     
하위 클래스에서만 정의된 메소드를 사용시 에러를 낸다고합니다.    
   
(예시)   
     
```

    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;

    interface People {
        void eat();
        void sleep();
    }

    class Student implements  People {
    
        @Override
        public void eat() {
            System.out.println("nice food  !");
        }
    
        @Override
        public void sleep() {
            System.out.println("zzzzzz !");
        }
        public void study(){
            System.out.println("studying !");
        }
    }


    public class Main {
        public static void act(List<? extends People> people){
            people.get(0).eat();
            //people.get(0).study(); 이부분은 컴파일 에러가 나옵니다.
    
        }
        public static void main(String args[]){
            List<Student> list = new ArrayList();
            Student student = new Student();
            list.add(student);
            act(list);
        }
    }

```
   
다시 method 설명으로 돌아와서 , addAll method 는  인자로 받은 collection 타입의 element들을 모두 추가하는 메소드 입니다.      
자세한 내용은 collection을 구현하고 있는 하위 클래스를 분석하며 정리하도록 하겠습니다.   

> 인자로 받은 collection 타입의 element를 해당 collection에 추가하는 method .   

+ void clear()   

의미 그대로 해당 collection에 있는 모든 인자를 clear 하는 method 입니다.   

> 해당 Collection의 모든 element를 지웁니다.    

+ boolean contains(Object o)   

인자로 받은 o 가 collection에 있으면 true를 반환. 없으면 false를 반환한다.   

> 해당 elements 가 있는지 확인하는 method .   

+ boolean containAll(Collection<?> c)   

인자로 받은 collection의 element 모두가  해당 collection에 있으면 true를 반환합니다.   

> 인자로 받은 collection의 모든 element가 포함되어있는지 확인하는 method   

+ boolean equeals(Object o)   
+ int hashCode()   
동일성과 동등성을 체크하기위한 method .   
Object 클래스에서 상속받아 구현하기도 한다.   

> 설명 할 내용이 많기 때문에 추후 이 주제로 정리하도록 하겠습니다.   


+ boolean isEmpty()   

> Collection이 비어있으면 true를 반환. 비어있지 않으면 false를 반환.   

+ Iterator<E> iterator()   

> Iterator 를 구현하기 위한 method . 이또한 추후 정리.

+ boolean remove(Object o)

> 값을 넘겨받은 object element를 삭제하는 method

+ boolean removeAll(Collection<?> c)   
 
> 넘겨받은 collection의 element들을 삭제하는 method.
   
+ boolean retainAll(Collection<?> c)    

> 넘겨받은 collection의 element들을 유지하는 method .
 
ArrayList를 찾아보며 정리하던 중 , removeAll 과 retainAll의 동작 방식이 같다는 걸 알게 되었습니다. 찾아보니 bug 라고 하더군요 ... (제가 갖고있는 버전에 문제가 있던것 같습니다.)
      
[자세한 설명](http://stackoverflow.com/questions/8372576/java-commons-collections-removeall)을 참고해 주세요.
 
+ int size()
   
> collection의 size를 반환하는 메소드 입니다 . (가장 많이 접했던 method가 아닐까합니다.)

+ Object[] toArray()  

> collection의 element들을 object 타입의 array로 변환하여 반환하는 method 입니다.

+ <T> T[] toArray()

> 이전의 toArray와 같은 방식으로 array를 반환하지만, 제네릭을 사용하여 타입을 사용자가 정할 수 있다는 차이가 있습니다.   




추후 Collection을 상속받은 List를 정리하면서 오늘 설명하지 못한 부분들을 정리하도록 하겠습니다.    

감사합니다. 

~ 2016 - 10 -31

