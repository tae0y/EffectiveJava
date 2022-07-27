# EffectiveJava

## 1장
### 1. 생성자 대신 정적 팩터리 메서드
- 시그니처가 같은 생성자가 여러 개 필요하면, 메서드 이름을 서로 다르게 해주면 된다.
- 싱글톤 객체를 사용할 때 유용하다. 객체가 생성되어 있으면, 새로 생성않고 해당 객체를 반환만 해준다.
- 반환할 객체의 클래스를 유연하게 선택할 수 있다.
- 반환타입의 하위타입 객체를 반환할 수 있다.
- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
- 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체 클래스가 존재하지 않아도 된다.
- 그러나
  - 상속을 하려면 public이나 protected 생성자가 필요하니, 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
  - 해당 메서드의 역할이 무엇인지 파악하기 어려울 수 있다. 반면 생성자는 누가봐도 '객체를 생성'하는 코드이다.

### 2. 생성자에 매개변수가 많다면 빌더
- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.
- 매개변수가 많다면 변수별로 setter를 사용해서 설정해줄 수도 있으나 객체가 완성되기 전까지는 일관성이 유지되지 않는다. 
- 그래서 클래스를 불변으로 만들 수 없고, 스레드 안전성을 얻으려면 추가 작업이 필요하다
- 빌더 패턴은 코드가 장황한 편이므로 매개변수가 4개 이상인 경우에 추천한다.

### 3. private 생성자나 열거 타입으로 싱글턴임을 보증
- 싱글턴 객체를 가짜 구현으로 대체할 수 없기 때문에 테스트하기가 어렵다
- 정적 팩터리 메서드로 싱글톤 객체를 생성하고 반환할 수 있다
  - 생성된 객체는 private final, 생성자도 private으로 관리
  - 메서드 참조나 제네릭 팩터리 메서드로 활용할 수도 있다(`Supplier<Elvis>`)
- 리플렉션, 직렬화 등으로 싱글턴 속성이 깨질 수 있다
  - 접근제한해제/`Enum`: `Singleton.class.getDeclaredConstructors()`나 `constructor.setAccessible(true)`로 접근제한을 해제할 수 있다. Java가 내부적으로 열거형 값이 한번만 인스턴스화되도록 보장하기 때문에 Enum을 사용하면 대응할 수 있다. 대신 지연 초기화가 안된다.
  - 역직렬화/`readResolve`: `file.txt`에 class를 작성하고 해당파일을 역직렬화해서 객체를 생성하는 경우 싱글턴 속성이 유지되지 않는다. class 내부에 readResolve 메서드를 구현하면 회피할 수 있다.
  - `clone`/재정의
  - 자세한 내용: [싱글턴 속성을 깨뜨리지 않도록 대응하는 방법](https://www.geeksforgeeks.org/prevent-singleton-pattern-reflection-serialization-cloning/)

### 4. 인스턴스화를 막으려면 private 생성자를 사용
- 정적 멤버만 담은 유틸리티 클래스를 인스턴스화하지 않도록 생성자를 private으로 관리한다. 다만 상속이 불가능해진다.

### 5. 자원 직접 명시 > 의존 객체 주입 (이른바 '의존성주입')
- 미리 멤버로 정의해두지 않고, 생성 시점에 필요한 객체를 주입받을 수 있다
- 혹은 생성자로 서플라이어를 전달받아(`()->Something`) 동적으로 생성해줘도 된다

### 6.같은 객체를 반복하여 생성하지 않기
- 문자열 match 메서드에서 정규표현식을 문자열로 넘기면 매번 Pattern 객체가 생성되고 GC 대상이 된다. 대신 Pattern 객체를 만들어 캐싱해두고 필요할때마다 사용하는 편이 성능면에서도 가독성면에서도 좋다.

### 7. 다쓴 객체는 참조 해제하기
- 해당 참조를 다 썼을때 null로 바꿔준다
- 다만, 객체 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효범위 밖으로 밀어내는 것이다
- 메모리 누수 시나리오
  - 자기 메모리를 직접 관리하는 클래스: 배열로 구현한 스택에서 pop하면 인덱스만 낮춰주는 경우, 비활성 영역에서 참조하는 객체도 유효한 것으로 인식해서 메모리를 해제할 수 없다. 이와 같이 메모리를 직접 관리하는 경우 null을 지정해주어 gc할 수 있게 해야한다.
  - 캐시: 캐시 엔트리의 유효기간을 정확히 정의하기 어려워서 aging방식을 흔히 사용한다. 혹은 WeakHashMap을 사용하여 캐시 외부에서 키를 참조하는 동안만 살아있는 캐시를 만들 수도 있다.
  - 리스너/콜백: 콜백을 등록만하고 명확히 해지하지 않으면 콜백이 쌓여만 간다. 약한참조로 저장하면 가비지 컬렉터가 즉시 수거해간다. 이를테면 WeakHashMap을 사용하자.

### 8. finalizer와 cleaner 사용을 피하기 (그냥 절대 쓰지말자)
- 자바9부터 finalizer가 deprecated되었다. 그러나 cleaner 역시 예측할 수 없고, 느리고, 불필요하다.
- 자바는 객체 회수를 위해 try-with-resources, try-finally를 사용한다.
- 생성자나 직렬화 과정에서 예외가 발생하면 생성되다만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있다. 그리고 finalizer를 정적필드에 참조 할당하여 가비지 컬렉터가 수집하지 못하게 만들고, 일그러진 객체를 사용해 허용되지 않은 작업도 수행할 수 있다. final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들어 final로 선언하자.
- 대안
  - finalizer와 cleaner 대신 AutuCloseable을 구현해 사용하면 된다 (팁: 인스턴스별로 close여부를 확인하는 필드를 선언해두고, 해당 필드가 close인데 참조하면 예외를 던지도록 구현해두면 좋다)

### 9. try-finally 보다는 try-with-resources
- AutoCloseable을 구현한 객체는 `try(statements){}` 구문으로 자원을 회수할 수 있다
- finally에서 close메서드를 호출하는 경우, 중첩된 구문에서 디버깅이 어려워지는 문제가 있다
- close를 명시하지 않은 경우 finalizer가 안전망으로 호출되는데, 동작을 보장하지 않는 문제가 있다

### 10. equals는 일반 규약을 지켜 재정의
- 클래스를 확장해 구현했을때 기존 클래스와 확장한 클래스를 정확히 equals 비교할 수는 없다
- equals 메서드 구현 방법 정리
  - `==` 연산자 사용해 자기자신 참조인지 확인
  - `instanceof`로 입력이 올바른 타입인지 확인
  - 입력을 올바른 타입으로 형변환
  - 입력 객체와 자기 자신의 대응되는 핵심필드들이 모두 일치하는지 확인
- 단위테스트
  - 대칭적인가? `a.equals(b) == b.equals(a)`
  - 추이성이 있는가? `if a.equals(b) and b.equals(c) then a.equals(c) is true`
  - 일관적인가? (두 객체가 같다면 앞으로도 계속 같아야 한다. 비교시점에 따라 달라지면 안됨.)
- equals를 재정의할땐 hashcode도 반드시 재정의
- `AutoValue` 라이브러리를 사용하면 자동으로 만들어준다 👍

### 11. equals를 재정의하려거든 hashCode도 재정의
- equals에서 일부 필드만을 사용해서 비교할 수도 있다, 그러나 논리적으로 같은 객체는 hashcode도 같아야하기에 equals를 재정의하면 hashcode도 재정의해줘야 한다. equals에서 포함하지 않은 필드는 반드시 제외하고 hashing 해야 한다. 그 반대는 상관없다.
- 해시 함수 제작 요령을 참조 (라빈 핑거프린트 알고리즘)
- 또는 구아바의 `com.google.common.hash.Hashing`도 좋다

### 12. toString을 항상 재정의
- 좋은 toString이 좋은 로깅을 만든다
- toString()의 포맷을 정하려면 구체적으로, 앞으로 변하지 않게 만들어야 하며
- toString()의 포맷을 정하지 않으려면, 그 이유를 밝혀둬야 한다
- toString에 포함된 값을 얻어올 수 있는 API를 제공해야 toString을 파싱하는 경우를 방지할 수 있다
- `AutoValue` 라이브러리를 사용하면 자동으로 만들어준다 👍

### 13. clone 재정의는 주의해서 진행
- clone을 구현한 A를 상속해서 B클래스를 만들었다. 여기에 clone을 구현한다면
  - 생성자, 팩토리 메서드를 타지 않고 객체가 생성된다
  - super.clone이 호출되는데 B.clone에서 원하는 것과 다를 수 있다
  - 원본/복제의 필드가 같은 레퍼런스를 참조하면 오류가 발생할 수 있다
  - A.clone이 구현되지 않았다면 CloneNotSupportedException이 발생한다
- 차라리 객체를 집어넣으면 복사한 객체를 반환하도록 생성자/팩토리 메서드를 오버로딩해두는 편이 낫다

### 14. Comparable을 구현할까?
- 구현하면 좋다
- 확장한 클래스에서 comparable을 안전하게 구현하는 방법: 독립된 클래스로 만들고, 상위 클래스 인스턴스 필드를 포함시킨다. 그리고 해당 인스턴스를 반환하는 '뷰' 메서드를 제공한다. 이렇게하면 하위 클래스에서 상위 클래스와 혼동되지 않게 comparable을 구현할 수 있다.
- (참고)정렬된 컬렉션은 동치성을 비교할 때 equals대신 compareTo를 사용한다
- 필드의 값을 비교할 때 `>`나 `<` 대신 박싱된 기본타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자 !

### 15. 클래스와 멤버의 접근 권한을 최소화
- 잘 설계된 컴포넌트는 구현과 API를 깔끔히 분리한다
- 접근제어자별 접근 가능한 범위 
  
| 접근 제어자 | 같은 클래스의 멤버 | 같은 패키지의 멤버 | 자식 클래스의 멤버 | 그 외의 영역 |
| :---------: | :----------------: | :----------------: | :----------------: | :----------: |
|   public    |         ○          |         ○          |         ○          |      ○       |
|  protected  |         ○          |         ○          |         ○          |      X       |
|   default   |         ○          |         ○          |         X          |      X       |
|   private   |         ○          |         X          |         X          |      X       |
- 상위 클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스보다 좁게 설정할 수 없다. 이 제약은 상위 클래스의 인스턴스를 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다는 규칙을 지키기 위해서이다. 이를테면 인터페이스를 구현한 클래스에서, 클래스는 인터페이스가 정의한 모든 메서드를 public으로 선언해야 한다.
> **인터페이스**: 무조건 public, 멤버변수는 final static이며, 메서드는 구현부를 가질 수 없음, 다중상속이 가능함
>
> **추상클래스**: 무조건 public은 아님, 멤버는 접근제어자를 알아서 잘 설정하면 됨, 단일상속만 가능함
- public 클래스의 인스턴스 필드가 public이라면 그 필드와 관련되 모든 것은 불변식을 보장할 수 없게 된다. 여기에 더해 필드가 수정될 때 다른 작업을(락 획득) 할 수 없게 되므로, public 가변 필드를 갖는 클래스는 일반적으로 스레드 안전하지 않다.
- 변수를 final로 선언해도, 해당 변수를 참조한 다른 인스턴스는 수정할 수 있다
- 길이가 0이 아닌 배열은 수정할 수 있다
  - private final 불변 리스트로 대체
  - 배열을 private으로 선언하고, 접근 메서드에서는 해당 배열의 복사본을 반환
  
### 16. public 클래스에서 public 필드가 아닌 접근자 메서드를 사용하라
- public 클래스에서 멤버를 public으로 노출하면, 캡슐화의 이점을 살리지 못한다
  - API를 수정하지 않고는 내부 표현을 바꿀 수 없고
  - 불변식을 보장할 수 없으며
  - 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다

### 17. 변경 가능성을 최소화하라
- 클래스를 불변으로 만드는 다섯 가지 방법
  - 객체 상태를 변경하는 메서드(setter)를 제공하지 않음
  - 클래스를 확장할 수 없도록 함(예: final로 선언 등)
    - 모든 생성자를 private 또는 package-private으로 만들고 public 정적 팩터리를 제공
      - public이나 protected 생성자가 없어 다른 패키지에서 확장할 수 없음
      - 객체 캐싱 기능을 추가해 성능을 높일 수도 있음(불변이므로)
  - 모든 필드를 final로 선언
  - 모든 필드를 private으로 선언
  - 자신 외에는 내부 가변 컴포넌트에 접근할 수 없도록 함(접근 필요하면 객체를 복사해서 건네줌)
- 그리고 한 가지 더
  - 메서드를 함수형 프로그래밍 방식으로 구성한다: 같은 입력에 대하여 같은 결과를 응답할 수 있게 한다. 자기자신 또는 전달받은 객체를 변경하는 대신 새로운 객체를 만들어 결과값을 반환한다.
- 불변 객체의 이점
  - 단순하다. 믿을 수 있다. 상태 전이 내용을 상세하게 문서로 남기지 않아도 된다.
  - 스레드 안전하다. 안심하고 공유할 수 있다. 불변객체는 공유해도 부작용이 없으므로, 복사 메서드를 만들 필요도 없다.
  - 불변 객체끼리 내부 데이터를 공유해도 된다.
  - 객체를 만들때 불변 객체를 구성요소로 사용하면 불변식을 유지하기도 좋다.
  - 불변 객체는 실패 원자성을 제공한다. (실패 원자성: 메서드에서 예외가 발생한 후에도 그 객체는 여전히 유효한 상태여야 한다)
  - 불변 객체는 값이 다르다면 반드시 독립된 객체로 만들어야 한다.
- 불변 객체를 인자로 받았을 때
  - 신뢰할 수 없는 하위 클래스의 인스턴스라면, 가변이라 가정하고 방어적으로 복사해 사용해야 한다.
  - 해당 객체를 인자로 받는 팩터리 메서드를 만들어두고, 인자로 전달된 객체 대신 그 객체를 복사한 것에 접근한다.
  - 이를테면 BigInteger, BigDecimal은 불변 객체이나 메서드는 모두 재정의할 수 있으므로 불변성을 해칠 수 있다.
- 불변 객체 내부데이터의 지연초기화/캐싱
  - 객체가 불변성을 유지하면, 내부데이터를 가변멤버로 선언하고 지연초기화/캐싱해도 상관없다.
- 불변 객체의 성능 문제
  - 성능 때문에 연산도중 불변 객체를 계속해서 생성하는 것이 부담된다면, 쌍을 이루는 가변 동반 클래스를 public으로 제공하자.
  - https://github.com/JunHyeok96/effective-java/issues/15
  - https://github.com/Java-Bom/ReadingRecord/issues/47
- 클래스가 최대한 불변성을 가지도록 선언하자는 것이다.
