---
title: "Iterator Pattern"
date: 2019-06-04
tags:
  - Design Pattern
  - 디자인 패턴
  - iterator
---
> ### The essence of the Iterator Pattern is to "Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation." - GoF
>
> ### Iterator패턴은 집합체(aggregate object)의 구현방법을 노출하지 않고, 집합체(aggregate object)의 요소(element)에 순차적으로 접근하는 방법을 제공한다.

![Iterator UML Diagram](../../../assets/images/design_pattern/Iterator_UML_class_diagram.png)<center>Iterator UML Diagram</center>


장점
---
1. 집합체의 내부 구현과 무관하게 저장된 요소들에게 하나씩 접근 할 수 있게 해줍니다.
2. 통일된 방식으로 저장된 모든 요소에 접근 할 수 있습니다.
3. 집합체 객체(aggregate)는 요소들의 관리에 집중 할 수 있고, 반복자 객체(interater)는 반복작업에 집중 할 수 있다.

당신의 동료가 index를 이용한 `get(int index)`라는 함수를 가지는 집합체 클래스 A를 새로 만들었다고 생각해봅시다. 당신과 다른 동료들은 클래스 A를 가지고 작업을 진행하고, A에 저장된 모든 요소에 접근할 때는 index를 0부터 증가시키는 반복문을 이용했습니다. 하지만 갑자기 엄청난 버그의 발견으로 클래스 A를 어쩔수 없이 변경했습니다. 그 과정에서 `get(int index)`를 사용할 수 없게 되었습니다. **당신과 팀원들은 이제 index를 이용한 반복문을 모두 찾아서 수정해야합니다.**

예제
---
### Iterator.java
```java
public interface Iterator {
    boolean hasNext();

    Object next();
}
```
### Aggregate.java
```java
public interface Aggregate {
    Iterator iterator();
}
```
### Student.java
```java
public class Student {
    private String name;
    private int id;

    public Student(String name, int id) {
        this.name = name;
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public int getId() {
        return id;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", id=" + id +
                '}';
    }
}
```
### ClassRoom.java
```java
public class ClassRoom implements Aggregate {
    private Student[] students;
    private int last;

    public ClassRoom() {
        students = new Student[25];
        last = 0;
    }

    public void setStudent(Student student) {
        students[last] = student;
        last++;
    }

    public Student getStudentByIndex(int idx){
        return students[idx];
    }

    @Override
    public Iterator iterator() {
        return new ClassRoomIterator();
    }

    private class ClassRoomIterator implements Iterator{
        private int index;

        ClassRoomIterator(){
            index = 0;
        }

        @Override
        public boolean hasNext() {
            return last > index;
        }

        @Override
        public Student next() {
            return students[index++];
        }
    }
}
```

### Main01.java
```java
public class Main01 {
    public static void main(String[] args) {
        ClassRoom mathClass = new ClassRoom();
        mathClass.setStudent(new Student("one", 1));
        mathClass.setStudent(new Student("two", 2));

        // 1. Iterator 사용
        Iterator iterator = mathClass.iterator();

        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }

        // 2. For문 사용.
        // 문제점: 길이를 제공하는 함수가 없다.(=ClassRoom 클래스에 의존한다)
        for (int i = 0; i < 2; i++) {
          System.out.println(mathClass.getStudentByIndex(i));
        }
    }
}
```
Main01.java에서 Iterator와 for문으로 동일한 결과값을 가져오는 것을 확인 할 수 있습니다. 드문 경우지만 위에처럼 for문을 사용할 때 집합체(ClassRoom 클래스)가 길이(Length)를 제공하지 않으면 for문 자체를 사용하기 곤란합니다. for문 대신에 Iterator 패턴을 사용하면서 얻는 이점이라 할 수 있습니다.

[더보기] Iterator vs for 비교
---
### ClassRoom.java
```java
public class ClassRoom implements Aggregate {
    private Student[] students;
    private int last;

    public ClassRoom() {
        students = new Student[25];
        last = 0;
    }

    public void setStudent(Student student) {
        students[last] = student;
        last++;
    }

    // 꼭! getStudentByName를 사용해주세요. 기존소스도 반드시 수정해주세요.
    @Deprecated
    public Student getStudentByIndex(int idx){
        return students[idx];
    }

    public Student getStudentByName(String name){
        for (int i=0; i < students.length; i++) {
          if(students[i].getName().equals(name)){
            return students[i];
          }
        }
        return null;
    }

    @Override
    public Iterator iterator() {
        return new ClassRoomIterator();
    }

    private class ClassRoomIterator implements Iterator{
        private int index;

        ClassRoomIterator(){
            index = 0;
        }

        @Override
        public boolean hasNext() {
            return last > index;
        }

        @Override
        public Student next() {
            return students[index++];
        }
    }
}
```

상황이 바뀌어서 ```getStudentByIndex(int idx)``` 메소드를 사용 할 수 없게 되었습니다. 그리고, 지금까지 사용된 모든 소스도 수정을 해야하는 상황이 되었습니다. 아래에 Main02 예제를 통해 확인해보도록 하겠습니다.

### Main02.java
```java
public class Main02 {
    public static void main(String[] args) {
        ClassRoom mathClass = new ClassRoom();
        mathClass.setStudent(new Student("one", 1));
        mathClass.setStudent(new Student("two", 2));
        mathClass.setStudent(new Student("three", 3));

        // 1. Iterator 사용
        Iterator iterator = mathClass.iterator();

        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }

        // 2. 기존 For문 사용금지. 폐기처분
        /*
        for (int i = 0; i < 2; i++) {
          System.out.println(mathClass.getStudentByIndex(i));
        }
        */

        // 3. getStudentByName 메소드로 For문 사용은 불가능함.
    }
}
```
ClassRoomIterator에서 hasNext()와 next() 메소드가 적절하게 구현되어 있기 때문에 Iterator를 사용하여 작성한 코드를 별도의 수정없이 계속 사용이 가능해졌습니다. 이 예제에서는 iterator에 변경이 없었지만 변경이 필요 했어도, 적절히 수정해주었다면 기존에 Iterator를 사용하는 코드는 계속 사용이 가능합니다. 이처럼 ```Iterator```는 ```for```보다 집합체의 구현에 ```덜 의존적```입니다.
