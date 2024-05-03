# 체계적인 자원 관리

## 자바 메모리 관리
  - C언어는 할당한 메모리를 직접 해제해줘야 한다.
  - 자바는 메모리 해제 함수가 따로 없기 때문에 개발자가 직접 메모리를 해제할 수는 없다.
  - JVM(Java Virtual Machine)이라고 하는 가상 머신이 이 역할을 수행한다.

## JVM(Java Virtual Machine)
  - 자바 애플리케이션이 사용하지 않는 메모리를 찾아내 주기적으로 정리하는데, 이 과정을 가비지 컬렉션(garbage collection) 이라고 한다.
  - 접근하는 프로세스가 있는 메모리는 사용 중으로 판단하고, 그렇지 않은 메모리는 사용하지 않는다고 판단한다.

## Garbage Collection 발생 시점
> Visual VM과 같은 자원 모니터링 도구를 활용하여 heap 메모리 크기를 관찰하여 메모리 누수 여부를 확인할 수 있다.
> 전반적으로 우상향 하는 그래프일 때, 가비지 정리가 제대로 되지 않아 메모리 누수가 발생하는 것으로 판단할 수 있다.
  - 예제 코드 1
    ```java
    String exData = new String("exData");
    System.out.println(exData);
    ```
    - new를 통해 문자열 객체를 생성하면 heap 영역에 메모리를 할당하고, 해당 위치의 주소를 반환하여 반환된 주소 값을 exData 변수에 할당한다. 이후 exData를 통해 메모리에
      접근하면, 이 메모리는 사용 중인 것으로 판단하여 가비지 컬렉션의 대상이 아니다.
  - 예제 코드 2
    ```java
    String exData = new String("exData");
    System.out.println(exData);
    exData = null;
    ```
    - exData 변수에 null을 주면 더 이상 메모리를 참조할 수 없게 된다. 메모리 주소를 참조하는 변수가 존재하지 않기 때문에 가비지 컬렉션의 대상이 된다.
    - 특정 변수에 null을 주면 모두 가비지 컬렉션의 대상이 되느냐? 그렇지 않다.
  - 예제 코드 3
    ```java
    String exData = new String("exData");
    String exData2 = null;
    
    exData2 = exData;

    exData = null;
    ```
    - exData를 마지막에 null로 주었기 때문에 가비지 컬렉션이 되상이 될 것 같지만 그렇지 않다.
    - exData 변수가 가지고 있는 주소 값을 exData2 변수에 복사하여 여전히 메모리 값을 참조하여 사용하기 있기 떄문이다.
    - 그렇기 때문에 exData = null 코드 라인은 작용하지 않아 exData는 가비지 컬렉션의 대상이 되지 않는다.
  - 예제 코드 4
    ```java
    String exData = new String("exData");
    String exData2 = null;
    
    exData2 = exData;

    exData = null;
    exData2 = null;
    ```
    - JVM이 메모리 해제하도록 하기 위해서는 exData, exData2 모두 명시적으로 null을 할당해주어야 한다. 이렇게 하면 메모리를 참조하는 변수가 없어져 더 이상 사용되지 않는
      메모리로 판단하여 가비지 컬렉션을 시작한다.






    
