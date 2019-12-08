# 프로그래밍 관련 원칙

[TOC]



## 디미터 법칙(The Law of Demeter)

- 소프트웨어 개발 가이드라인 중 하나

- Demeter Project라는 프로그래밍 프로젝트에서 유래

- 최소 지식 원칙 : 한 모듈이 구현을 알아야 하는 다른 모듈들을 최대한 적게 유지하자

- 모듈들 사이의 결합도를 줄여 코드 품질을 높이자 

  - 결합도가 높으면 그만큼 모듈 재사용하기 힘듦

- 객체 O의 메소드 m은 다음의 객체들의 타입의 메소드만 호출해야 한다는 법칙

  1. O 객체 자신의 메소들 (O itself)
  2. m의 파라미터로 넘어온 객체들의 메소드들(M's parameter)
  3. m 안에서 생성 되거나 초기화된 객체들의 메소드들(Any objects created/instantiated within M)
  4. O 객체의 직접 소유하는 객체의 메소드들 (O's direct component objects)
  5. O 객체의 m에서 접근이 가능한 전역변수의 메소드들(A global variable, accessible by O, in the scope of M)

  ```java
  class Demeter {
  
      private A a;
      private int func() { return 0; }
      public void example(B b) {
          C c = new C();
          int f = func(); // 1번의 경우
          b.invert(); // 2번의 경우
          a = new A();
          a.setActive(); // 3번의 경우
          c.print(); // 4번의 경우
          // Static으로 설정된 Setting 변수가 있다고 가정할때.
          string UserID = GlobalValues.Setting.getUserID(); //5의 경우
      }
  }
  ```

  위 종류에 해당하는 객체의 메소드들만 호출해야 하낟. 그렇지 않으면 유지보수 측면에서 문제점들이 발생한다.

  ```java
  // 잘못된 예시
  objectA.getObjectB().doSomething(); 
  
  objectA.getObjectB().getObjectC().doSomething(); 
  ```

  이렇게 구현이 되면 사용하는 클래스의 메소드는 ObjectA가 변경이 되었을 때만 코드를 수정하거나 고민하는 것이 아니라 ObjectB,ObjectC의 변경에도 모두 검토가 필요함.

  즉, 이런식으로 호출하면 ObjectB에만 노출되어야 할 메소드들이 ObjectA에게도 노출이 된다. 이런 경우에 ObjectB,ObjectC의 doSomething에 문제가 발생하면 모든 객체(ObjectA,B,C)에 모두 영향을 미치게 된다. 최대한 노출 범위를 제한해서 유지보수가 쉽게 해야한다.

- 컴포넌트의 모든 인터페이스를 엔티티로 올릴수 없는 노릇

  - 단순히 `.`을 줄이기 위해 올린 인터페이스는 컴포넌트,엔테티 인터페이스를 수정하고 그 다음 엔티티을 사용하는 곳을 수정해야하는 수정의 연쇄를 피할 수 없다.
  - 결론은 그때그때 다르게 `현실세계의 묘사`를 적용 여부 기준으로 삼아보자

## 파레토 법칙(Pareto principle)

- 80 대 20 법칙, 전체 결과의 80%가 전체 원인의 20%에서 일어나는 현상

- 소프트웨어 개발 80%를 20%의 기능을 개발하는데 소비한다. 프로그램 사용자의 20%가 80%의 부하를 발생시킨다 등으로 사용이 된다. 프로젝트 설계할 때 이 법칙을 감안해서 설계하면 프로젝트를 위험에 빠트리게 하는 위험요소들을 사전에 파악하고 관리가 가능할 것이다.
- 예를 들어서 중요한 기능을 먼저 구현하던지 특정 사용자들에게서 집중적으로 자원이 소모되는 것을 감안해서 설계하고 테스팅한다.

## 콘웨이 법칙(Conway's Law)

-  *“모든 시스템은 그 조직의 의사소통 구조와 동일하게 만들어진다.”* 
- 코드 구조는 개발팀의 구조를 따라간다는 법칙
-  예를 들면, 컴파일러를 네개의 그룹에서 만든다면 컴파일러는 네 단계로 구성된다. 이것은 어떻게 보면 당연한 결과인데, 팀의 구조다 다르면 콤포넌트로 분리되서 서로 인터페이스로 연계가 된다. 이것에는 장단점이 존재하게 되는데 적절한 수준에서 구조를 정리하고 팀이 분리되어 있더라도 통합이 필요한 것은 통합을 해서 적정수준으로 코드구조를 관리해야한다.
-  이 법칙이 주장하는것은 `작업의 효울성`을 높이기 위해서는 `유연한 조직구조`가 필요하다는 것이다. 

----------

[출처]

 https://www.slideshare.net/SangminLim/ss-63457159 

 https://hongjinhyeon.tistory.com/138 [생각대로 살지 않으면 사는대로 생각한다.]