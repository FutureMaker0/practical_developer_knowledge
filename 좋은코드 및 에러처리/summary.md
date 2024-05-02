# 에러 처리

## 원인 분석을 위한 에러처리
  - 완성도 있는 소프트웨어 개발을 위해 예상 가능한 모든 에러 상황을 고려하여 적절한 처리 로직을 작성해야 한다.
  - 대표적으로 자바의 try-catch 구문이 있다. try 구문에 에러가 나올 가능성이 있는 코드를 작성하고, 실행 과정에서 에러가 나오면 catch 구문에서 예외 처리를 한다.
  - finally는 어떤 상황에서든 무조건 수행되어야 하는 코드 영역이다. 예외가 발생하여 catch 구문이 실행되어도 finally는 동작한다.
  - 자원 낭비를 위해 자원을 해제해주는 작업 등이 finally에 포함된다.
    ```java
    public class MyClass extends MyCustomException {
      public static void main(String[] args) {
      
        try {
          /*
          예외가 발생할 수 있는 프로그램 코드 영역
          */
        } catch (MyCustomException e) {
            /*
            예외 발생 시 처리할 코드 영역
            */
        } finally {
            /*
            예외 발생 상황과는 관계 없이 무조건 수행되는 코드 영역
            */
        }
      }
    
    }
    ```

## 예외 처리 코드가 부재할 경우
  - 예외가 발생했을 때의 적절한 처리 로직을 구현하지 않으면, 프로그램이 중지되며 사용자 친화적 결과가 제공되지 않고 IDE 등이 제공하는 기계적 메시지가 클라이언트에 노출된다.
  - catch 구문에 시스템 로그라도 찍어서 어떤 문제가 있는지 확인할 수 있는 최소한의 조치가 있어야 한다.
  - JDK는 발생 가능한 여러 예외 상황을 Exception 하위 클래스에 두고 있는데, 수십 가지 종류가 존재하며 예외 상황에 맞게 이들을 활용할 수 있는 것이 개발자의 역량이다.

## 예외를 던진다(throw), 예외를 받는다.(catch)
  - throw: 예외사항이 발생하면 이 메서드를 호출한 코드에 Exception을 던진다. 던져서 처리하라고 권한을 넘긴다.
  - catch: 어디선가 던져진 Exception을 갈고리로 잡아 와서 처리할 처리할 준비를 한다.
  ```java
  public class MyClass extends MyCustomException {
      public static void main(String[] args) {
      
        try {
          /*
          프로그램 동작 로직
          */
        } catch (MyCustomExceptionLevel3 e) {
            System.out.pringln("예외를 잡았으며, 처리해야 합니다. 매우 구체화된 예외 처리입니다.");
        } catch (MyCustomExceptionLevel2 e) {
            System.out.pringln("예외를 잡았으며, 처리해야 합니다. 보통 구체화된 예외 처리입니다.");
        } catch (MyCustomExceptionLevel1 e) {
            System.out.pringln("예외를 잡았으며, 처리해야 합니다. 덜 구체화된 예외처리입니다.");
        } finally {

        }
      }
    
    }
  ```
  - Exception은 if-else 구문과 같이 위에서 아래로 순차적으로 수행되므로 Level3 와 같이 구체화된 예외처리가 가장 상단에 위치하며 아래로 갈수록 상위 부모 개념의 포괄적인
    Exception이 위치하게 된다.
  - 이 순서는 반드시 지켜져야 하는 것이, 포괄적 예외 처리를 위한 throw가 상단에 위치할 경우 예외가 발생했다는 사실만 인지할 뿐 정확히 어떤 예외가 발생했는지 파악하는데 어려움이
    발생할 수 있다.

## 예외(Exception)과 에러(Error)의 차이
  - 에러 역시 상위 부모 개념의 throwable을 상속받고 있는데, 예외와는 다르게 프로그램이 운영체제 영역까지 침범하여 강력히 조치를 취하는 클래스로 개발자가 사용할 수 있는 클래스가
    아니다.
    > throwable class: 프로그램에 예외를 던지거나 에러 로그와 메시지를 출력하는 역할을 하는 클래스
  - 에러 클래스는 자바로 구성된 프로그램에서 복구가 불가능한 매우 심각한 상황에서 실행된다. JVM 내부 에러, 메모리 부족, 스택 오버플로우 등 프로그램의 자체적 처리가 불가능한
    상황을 정의하고 있는 경우이다. 그렇기 때문에 발생할 경우 프로그램이 종료될 수 있다.
  - 예외는 프로그램 내부에서 비정상적 상황이 생긴 경우 내부에서 처리가 가능한 상황이고, 에러는 프로그램에서 처리가 불가능한 상황을 말한다.

  <img width="500" alt="스크린샷 2024-05-02 오후 1 30 43" src="https://github.com/FutureMaker0/designPattern_singleton/assets/120623320/1733a2a6-9571-4a26-9098-9e9f895e242a"></br>
  출저: https://doyoung.tistory.com/22

