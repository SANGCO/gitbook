

# 아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라

> In summary, when **multiple threads** share **mutable data**, each thread that reads or writes the data must **perform synchronization**. In the absence of synchronization, there is no guarantee that one thread’s changes will be visible to another thread. The penalties for failing to synchronize **shared mutable data** are **liveness** and **safety failures**. These failures are among the most difficult to debug. They can be **intermittent** and **timing-dependent**, and program behavior can vary radically from one VM to another. If you need only **inter-thread communication**, and **not mutual exclusion**, the volatile modifier is an acceptable form of synchronization, but it can be tricky to use correctly.

> **여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화 해야 한다.** 동기화하지 않으면 한 스레드가 수행한 변경을 다른 스레드가 보지 못할 수도 있다. 공유되는 가변 데이터를 동기화하는 데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다. 이는 디버깅 난이도가 가장 높은 문제에 속한다. 간헐적이거나 특정 타이밍에만 발생할 수도 있고, VM에 따라 현상이 달라지기도 한다. 배타적 실행은 필요 없고 스레드끼리의 통신만 필요하다면 volatile 한정자만으로 동기화할 수 있다. 다만 올바로 사용하기가 까다롭다.



- `synchronized` 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.
- **배타적 실행**
  - 한 스레드가 변경 중이라 상태가 일관되지 않은 객체는 다른 스레드가 보지 못하게 막는다.
- 동기화의 용도를 배타적 실행만으로 생각하면 안된다.
- 메서드는 일관된 상태를 가지고 생성된 객체에 접근할 때 락을 건다.
  - 락을 걸어 놓고 객체의 상태를 확인한 후 필요하면 수정한다.
  - 동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없겠다.
- 동기화에는 배타적 샐행 말고도 중요한 기능이 하나 더 있다.
  - 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.
  - 동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기환된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.
- 언어 명세상 `long`과 `double` 외의 변수를 읽고 쓰는 동작은 원자적(atomic)이다.
  - 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장
- 자바 언어 명세는 스레드가 필드를 읽을 때 항상 **'수정이 완전히 반영된'** 값을 얻는다고 보장
  - 하지만 한 스레가 저장한 값이 다른 스레드에게 **'보이는가'**는 보장하지 않는다.
  - '수정이 완전히 반영된' 값을 읽어오기는 하는데 그 값이 바로 직전에 다른 스레드가 저장한 값은 아닐 수도 있겠군. 사실 바로 직전에 저장한 값을 읽어와야 하는데 말이지.
- 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.
  - 이는 한 스레드가 만든 변화가 다른 스레드에게 언제 어떻게 보이는지를 규정한 자바의 메모리 모델 때문이다.



```java
// 코드 78-1 잘못된 코드 - 이 프로그램은 얼마나 오래 실행될까?
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

- 공유 중인 가변 데이터를 원자적으로 읽고 쓸 수 있더라도 동기화에 실패하면 처참한 결과로 이어질 수 있다.
- 코드 78-1에 프로그램이 1초 후에 종료되리라 생각하는가?
  - 메인 스레드가 1초 후 stopRequested를 `true`로 설정하면 backgroundThread는 반복문을 빠져나올 것처럼 보이지만 도통 끝나지 않고 계속 실행된다.
  - 원인은 동기화에 있다.
    - 동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 볼지는 보증할 수 없다.
    - 동기화가 빠지면 가상머신이 다음과 같은 최적화를 수행할 수도 있다.

```java
// 원래 코드
while (!stopRequested)
    i++;

// 최적화한 코드
// OpenJDK 서버 VM이 실제로 적용하는 끌어올리기(hoisting)라는 최적화 기법이다.
if (!stopRequested)
  	while (true)
      	i++;
```



```java
// 코드 78-2 적절히 동기화해 스레드가 정상 종료한다.
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}  
```

- **쓰기 메서드(requestStop)와 읽기 메서드(stopRequested) 모두를 동기화했음에 주목**
  - 쓰기 메서드만 동기화해서는 충분하지 않다.
  - 쓰기와 읽기 모두를 동기화해야 동작이 보장된다.



```java
// 코드 78-3 volatile 필드를 사용해 스레드가 정상 종료한다.
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

- 반복문에서 매번 동기화하는 비용이 크진 않지만 속도가 더 빠른 대안을 소개하겠다.
  - 코드 78-2에서 stopRequested 필드를 volatile로 선언하면 동기화를 생략해도 된다. 
  - `volatile` 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.



```java
// 코드 78-4 잘못된 코드 - 동기화가 필요하다!
private static volatile int nextSerialNumber = 0;
 
public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

- 문제는 증가 연산자(`++`)다.
  - 이 연산자는 코드상으로는 하나지만 실제로는 nextSeriaNumber 필드에 두 번 접근한다.
    - 먼저 값을 읽고, 그런 다음 (1 증가한) 새로운 값을 저장
  - 만약 두 번째 스레드가 이 두 접근 사이를 비집고 들어와 값을 읽어가면 첫 번째 스레드와 똑같은 값을 돌려받게 된다.
  - 프로그램이 잘못된 결과를 계산해내는 이런 오류를 안전 실패(safety failure)라고 한다.
- generateSerialNumber 메서드에 `synchronized` 한정자를 붙이면 이 문제가 해결된다.
  - 동시에 호출해도 서로 간섭하지 않으며 이전 호출이 변경한 값을 읽게 된다는 뜻이다.
  - 메서드에 `synchronized`를 붙일거면 nextSerialNumber 필드에서 `volatile`을 제거해야한다.
- 이 메서드를 더 견고하게 하려면 `int` 대신 `long`을 사용하거나 nextSerialNumber가 최댓값에 도달하면 예외를 던지게 하자.
  - 여기서 말하는 최댓값이란 `int`나 `long`의 최댓값을 말한다.
  - `int` 대신 `long`으로 바꾼다고 뭐가 더 견고해 지는거지? 그냥 실행이 더 지속되는거지.



```java
// 코드 78-5 java.util.concurrent.atomic을 이용한 락-프리 동기화
private static final AtomicLong nextSerialNum = new AtomicLong();
 
public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

- 아이템 59의 조언에 따라 `java.util.concurrent.atomic` 패키지의 `AtomicLong`을 사용해보자.
  - **`java.util.concurrent` 패키지에는 락 없이도(lock-free; 락-프리) 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다.**
  - 이 패키지는 통신과 원자성(배타적 실행)이라는 동기화의 두 화과를 모두 지원
    - `volatile`은 통신 쪽만 지원
    - 우리가 generateSerialNumber에서 원했던 바로 그 기능
    - 더구나 성능도 동기화 버전보다 우수



- 이번 아이템에서 언급한 문제들을 피하는 **가장 좋은 방법은 애초에 가변 데이터를 공유하지 않는 것**이다.
  - **가변 데이터는 단일 스레드에서만 쓰도록 하자.**

- 한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다.
  - 그러면 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽어갈 수 있다.
    - 이런 객체를 **사실상 불변**(effectively immutable)이라 한다.
      - 다른 스레드에 이런 객체를 건네는 행위를 **안전 발행**(safe publication)이라 한다.



# 아이템 79. 과도한 동기화는 피하라

> In summary, to avoid deadlock and data corruption, never call an alien method from within a synchronized region. More generally, keep the amount of work that you do from within synchronized regions to a minimum. When you are designing a mutable class, think about whether it should do its own synchronization. In the multicore era, it is more important than ever not to oversynchronize. Synchronize your class internally only if there is a good reason to do so, and document your decision clearly (Item 82).

> 교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자. 일반화해 이야기하면, 동기화 영역 안에서의 작업은 최소한으로 줄이자. 가변 클래스를 설계할 때는 스스로 동기화해야 할 지 고민하자. 멀티코어 세상인 지금은 과도한 동기화를 피하는 게 과거 어느 때보다 중요하다. 합당한 이유가 있을 때만 내부에서 동기화하고, 동기화했는지 여부를 문서에 명확히 밝히자(아이템 82).



- 아이템 78에서 충분하지 못한 동기화의 피해를 다뤘다면, 이번 아이템에서는 반대 상황을 다룬다.
  - 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳기도 한다.
- **응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.**
  - 예를 들어 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안 되며, 클라이언트가 넘겨준 함수 객체(아이템 24)를 호출해서도 안 된다.
    - 여기서 말하는 함수 객체는 Comparator 같은 익명 클래스를 얘기하는거 같다. 어떻게 구현을 해서 넘겼는지 알 수 없으니 호출하지 말라는거 같네. 결국 앞에 재정의할 수 있는 메서드는 호출하지 말라는 것과 같은 맥락이네.
- 동기화된 영역을 포함한 클래스 관점에서는 이런 메서드는 모두 바깥 세상에서 온 외계인이다.
  - 그 메서드가 무슨 일을 할지 알지 못하며 통제할 수도 없다는 뜻이다.
  - **외계인 메서드**(alien method)가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다.



```java
// 관찰자 패턴을 구현하여, 원소가 추가되면 알려주는 집합
// 코드 79-1 잘못된 코드. 동기화 블록 안에서 외계인 메서드를 호출한다.
public class ObservableSet<E> extends ForwardingSet<E> {
		  
    public ObservableSet(Set<E> set) { 
      	super(set); 
    }

  	// 함수형 인터페이스 SetObserver
    private final List<SetObserver<E>> observers = new ArrayList<>();

  	// 옵저버를 등록
    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

  	// 옵저버를 제거
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

	  // add 메서드에서 호출
    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }
  
  	// 예제에서는 옵저버 등록하고 for문 돌면서 0부터 99까지 숫자를 add하고 있다.  
  	@Override public boolean add(E element) {
    boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);
        return result;
    }
  
}  
```

```java
// 집합 관찰자 콜백 인터페이스
@FunctionalInterface public interface SetObserver<E> {
    // ObservableSet에 원소가 더해지면 호출된다.
  	// ObservableSet 클래스에서 added가 외계인 메서드임.
    void added(ObservableSet<E> set, E element);
}
```

- 과도한 동기화로 인해 문제가 발생하는 예제다.
- 코드 79-1은 어떤 집합(Set)을 감싼 래퍼 클래스이고, 이 클래스의 클라이언트는 집합에 원소가 추가되면 알림을 받을 수 있다.
  - 관찰자 패턴
- SetObserver 인터페이스는 구조적으로 `BiConsumer<ObservableSet<E>, E>`와 똑같다.
  - 그럼에도 커스텀 함수형 인터페이스를 정의한 이유는 이름이 더 직관적이고 다중 콜백을 지원하도록 확장할 수 있어서다.
  - 하지만 BiConsumer를 그대로 사용했더라도 별 무리는 없었을 것이다(아이템 44).



```java
// ObservableSet 동작 확인 #1 - 0부터 99까지 출력한다.
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
    // addObserver 람다를 사용
  	set.addObserver((s, e) -> System.out.println(e));

    for (int i = 0; i < 100; i++) {
        set.add(i);
    }  
}
```

- 눈으로 보기에 ObservableSet은 잘 동작할 것 같다.
  - 예컨대 위 프로그램은 0부터 99까지를 출력한다.



```java
// ObservableSet 동작 확인 #2 - 정숫값이 23이면 자신의 구독을 해지한다.
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
		
  	// removeObserver 메서드에 함수 객체 자신을 넘겨야는데 람다는 자신을 참조할 수단이 없다.
		// 그래서 여기에서는 익명 클래스를 사용했다.
    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23) // 값이 23이면 자신을 구독해지한다.
               s.removeObserver(this);
        }
    }); 
  
  	for (int i = 0; i < 100; i++) {
        set.add(i);
    }  
}  
```

- 이 프로그램은 0부터 23까지 출력한 후 관찰자 자신을 구독해지한 다음 조용히 종료할 것으로 기대된다.
  - 그런데 실제로 실행해 보면 그렇게 진행되지 않는다!
  - 이 프로그램은 23까지 출력한 다음 ConcurrentModificationException을 던진다.
- 옵저버 하나 등록하고 숫자들을 add() 메소드에 넣는다.
  - ObservableSet.add() -> ObservableSet.notifyElementAdded() ->  SetObserver.added()
  - notifyElementAdded() 열심히 for문 돌면서 added() 메소드를 호출하고 있는데 호출한 added() 메소드에에서 콜백으로  removeObserver()를 호출하고 있다.
  - synchronized가 붙은 notifyElementAdded(), removeObserver() 동시에 동작으로 하려고 하니깐 에러가 발생한다.
  - notifyElementAdded 메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시 수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것까지 막지는 못한다.



```java
// 코드 79-2 쓸데없이 백그라운드 스레드를 사용하는 관찰자
// addObserver 체크
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
            ExecutorService exec = Executors.newSingleThreadExecutor();
          
          	try {
              	exec.submit(() -> s.removeObserver(this)).get();
            } catch (ExecutionException | InterruptedException ex) {
              	throw new AssertionError(ex);
            } finally {
              	exec.shutdown();
            }
        }
    }
});
```

- 구독해지를 하는 관찰자를 작성하는데, removeObserver를 직접 호출하지 않고 실행자 서비스(ExecutorService, 아이템 80)를 사용해 다른 스레드한테 부탁할 것이다.

- 이 프로그램을 실행하면 예외는 나지 않지만 교착상태에 빠진다. 
  - 백그라운드 스레드가 s.removeObserver를 호출하면 관찰자를 잠그려 시도하지만 락을 얻을 수 없다.
  - 메인 스레드가 이미 락을 쥐고 있기 때문이다.
  - 그와 동시에 메인 스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다리는 중이다.
    - 교착상태

- 실제 시스템(특히 GUI 툴킷)에서도 동기화된 영역 안에서 외계인 메서드를 호출하여 교착상태에 빠지는 사례는 자주 있다.



```java
// 코드 79-3 외계인 메서드를 동기화 블록 바깥으로 옮겼다. - 열린 호출
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```

- 이런 문제는 대부분 어렵지 않게 해결할 수 있다.
  - **외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 된다.**
  - notifyElementAdded 메서드에서 관찰자 리스트를 복사해서 쓰면 락 없이도 안전하게 순회할 수 있다.
    - 이 방식을 적용하면 앞서의 두 예제에서 예외 발생과 교착상태 증상이 사라진다.



```java
// 코드 79-4 CopyOnWriteArrayList를 사용해 구현한 스레드 안전하고 관찰 가능한 집합
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
  	observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
  	return observers.remove(observer);
}

private void notifyElementAdded(E element) {
  	for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```

- 사실 외계인 메서드 호출을 동기화 블록 바깥으로 옮기는 더 나은 방법이 있다.
  - CopyOnWriteArrayList
    - **자바의 동시성 컬렉션 라이브러리**
    - 정확히 이 목적으로 특별히 설계되었다.
    - 이름이 말해주듯 ArrayList를 구현한 클래스로, 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현
    - 내부의 배열은 절대 수정되지 않으니 순회할 때 락이 필요 없어 매우 빠르다.
    - 다른 용도로 쓰인다면 끔찍하게 느리겠지만, 수정할 일은 드믈고 순회만 빈번히 일어나는 관찰자 리스트 용도로는 최적



- 코드 79-3처럼 동기화 영역 바깥에서 호출되는 외계인 메서드를 **열린 호출**(open call)이라 한다.

  - 외계인 메서드는 얼마나 오래 실행될지 알 수 없는데, 동기화 영역 안에서 호출된다면 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야만 한다.
  - 따라서 열린 호출은 실패 방지 효과외에도 동시성 효율을 크게 개선해준다.
- **기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는 것이다.**
- 성능 측면도 간단히 살펴보자.

  - 멀티코어가 일반화된 오늘날, 과도한 동기화가 초래하는 진짜 비용은 락을 얻는 데 드는 CPU 시간이 아니다.
  - 바로 경쟁하느라 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다.
- 가변 클래스를 작성해야 한다면 다음 두 가지 중 하나를 선택하자.

  - 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 만들어라.
  - 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자(아이템 82).
    - 단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 두 번째 방법을 선택해야 한다.
  - 그니깐 동기화를 전혀 하지 말던지 동기화를 내부에서 수행하던지 하라는거네.
  - java.util은 (이제 구식이 된 Vector와 Hashtable을 제외하고) 첫 번째 방식을, java.util.concurrent는 두 번째 방식을 선택했다.(아이템 81).
- 자바도 초창기에 이 지침을 따르지 않은 클래스가 많았다.
  - StringBuffer 거의 단일 스레드에서 쓰였는데 내부적으로 동기화를 수행하고 있다.
    - 그래서 StringBuilder가 나왔다.
      - StringBuilder는 동기화하지 않는 StringBuffer다.
  - 스레드 안전한 의사 난수기 jva.util.Random은 동기화 하지 않는 버전인 java.util.concurrent.ThreadLocalRandom으로 대체되었다.
  - **동기화를 할지 말지 선택하기 어려우면 하지말고 문서에 "스레드 안전하지 않다"고 명시하자.**
- 클래스를 내부에서 동기화하기로 했다면, 락 분할(lock splitting), 락 스트라이핑(lock striping), 비차단 동시성 제어(nonblocking concurrency control) 등 다양한 기법을 동원해 동시성을 높여줄 수 있다.
- **여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 반드시 동기화해야 한다.**

```java
// 78-4 예제
// nextSerialNumber는 동기화하지 않으면 
// private인데 서로 관련 없는 스레드들이 동시에 읽고 수정할 수 있게 된다. 
private static volatile int nextSerialNumber = 0;
 
public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```




# 아이템 80. 스레드보다는 실행자, 테스크, 스트림을 애용하라

- 이 책 초판의 아이템 49에서는 단순한 작업 큐(work queue)를 선보였다. 
  - 그 클래스는 클라이언트가 요청한 작업을 백그라운드 스레드에 위임해 비동기적으로 처리해줬다.
  - 작업 큐가 필요 없어지면 클라이언트는 큐에 중단을 요청할 수 있고, 그러면 큐는 남아 있는 작업을 마저 완료한 후 스스로 종료한다.
- 이 책의 2판이 나오기 앞서 java.util.concurrent 패키지가 등장했다.
  - 이 패키지는 실행자 프레임워크(Executor Framework)라고 하는 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다.
  - 그래서 초판의 코드보다 모든 면에서 뛰어난 작업 큐를 다음의 단 한 줄로 생성할 수 있게 되었다.



```java
ExecutorService exec = Executors.newSingleThreadExector();
exec.execute(runnable);
exec.shutdown();
```

- 실행자 서비스의 주요 기능들
  - 특정 태스크가 완료되기를 기다린다(코드 79-2에서 본 get 메서드).
  - 태스크 모음 중 아무것 하나 (invokeAny 메서드) 혹은 모든 태스크(invokeAll 메서드)가 완료되기를 기다린다.
  - 실행자 서비스가 종료하기를 기다린다(awaitTermination 메서드).
  - 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용).
  - 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheduledThreadPoolExecutor 이용).



- 여러분에게 필요한 실행자 대부분은 java.util.concurrent.Executors의 정적 팩터리들을 이용해 생성할 수 있을 것이다.
  - 평범하지 않은 실행자를 원한다면 ThreadPoolExecutor 클래스를 직접 사용해도 된다.
  - 이 클래스로는 스레드 풀 동작을 결정하는 거의 모든 속성을 설정할 수 있다.
- 작업 큐를 손수 만드는 일은 삼가야 하고, 스레드를 직접 다루는 것도 일반적으로 삼가야 한다.
- 실행자 프레임워크에서는 작업 단위와 실행 매커니즘이 분리된다.
  - 작업 단위를 나타내는 핵심 추상 개념이 태스크다.
    - 태스크에는 두 가지가 있다.
      - Runnable과 그 사촌인 Callable이다.
        - Callable은 Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있다.
- 태스크를 수행하는 일반적인 매커니즘이 바로 실행자 서비스다.
  - 태스크 수행을 실행자 서비스에 맡기면 원하는 태스크 수행 정책을 선택할 수 있고, 생각이 바뀌면 언제든 변경할 수 있다.
- 핵심은 (컬렉션 프레임워크가 데이터 모음을 담당하듯) 실행자 프레임워크가 작업 수행을 담당해준다는 것이다.



- 포크-조인 태스크는 포크-조인 풀이라는 특별한 실행자 서비스가 실행해준다.
  - 포크-조인 태스크, 즉 ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있다.
    - ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리한다.
    - 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다.
    - 이렇게 하여 모든 스레드가 바쁘게 움직여 CPU를 최대한 활용하면서 높은 처리량과 낮은 지연시간을 달성한다.



# 아이템 81. wait와 notify보다는 동시성 유틸리티를 애용하라

> In summary, using wait and notify directly is like programming in “concur- rency assembly language,” as compared to the higher-level language provided by java.util.concurrent. **There is seldom, if ever, a reason to use** **wait** **and** **notify** **in new code.** If you maintain code that uses wait and notify, make sure that it always invokes wait from within a while loop using the standard idiom. The notifyAll method should generally be used in preference to notify. If notify is used, great care must be taken to ensure liveness.

> wait와 notify를 직접 사용하는 것을 동시성 '어셈블리 언어'로 프로그래밍하는 것에 비유할 수 있다. 반면 java.util.concurrent는 고수준 언어에 비유할 수 있다. **코드를 새로 작성한다면 wait와 notify를 쓸 이유가 거의(어쩌면 전혀) 없다.** 이들을 사용하는 레거시 코드를 유지보수해야 한다면 wait는 항상 표준 관용구에 따라 while 문 안에서 호출하도록 하자. 일반적으로 notify 보다는 notifyAll을 사용해야 한다. 혹시라도 notify를 사용한다면 응답 불가 상태에 빠지지 않도록 각별히 주의하자.





- 동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 일 역시 불가능하다.
  - 동시성을 무력화하지 못한다.
    - 동시에 여러 작업을 진행하는걸 막을 수 없다.
  - 여러 메서드를 원자적으로 묶는다.
    - 묶은 메서드들의 동작들은 쪼갤 수 없다는 뜻임.
    - JPA 강의 두개를 원자적으로 묶는 메서드를 만드는 부분.
    - 코드 81-1에 Map의 putIfAbsent(key, value);
      - 여러 동작을 묶는다.
        - 주어진 키에 매핑된 값이 있는 없는 체크
        - 키에 매핑된 값이 없으면 새 값을 집어넣는다.
        - 키에 매핑된 기존 값이 있으면 그 값을 반환
        - 키에 매핑된 기존 값이 없으면 null을 반환 



```java
// 코드 81-1 ConcurrentMap으로 구현한 동시성 정규화 맵 - 최적은 아니다.
private static final ConcurrentMap<String, String> map =
  	new ConcurrentHashMap<>();

public static String intern(String s) {
  	String previousValue = map.putIfAbsent(s, s);
    return previousValue == null ? s : previousValue;
}
```

- Map의 putIfAbsent() 메서드 덕에 스레드 안전한 정규화 맵(canonicalizing map)을 쉽게 구현할 수 있다.



```java
// 코드 81-2 ConcurrentMap으로 구현한 동시성 정규화 맵 - 더 빠르다!
public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null)
          	result = s;
    }
    return result;
}
```



```java
// 코드 81-3 동시 실행 시간을 재는 간단한 프레임워크
public class ConcurrentTimer {
    private ConcurrentTimer() { } // 인스턴스 생성 불가

  	public static void main(String[] args) throws InterruptedException {
        ExecutorService e = Executors.newFixedThreadPool(10);
        System.out.println(time(e, 10, () -> System.out.println("Thread")));
        e.shutdown();
    }
  	
  	public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done  = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                ready.countDown(); // 타이머에게 준비를 마쳤음을 알린다.
                try {
                    start.await(); // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();  // 타이머에게 작업을 마쳤음을 알린다.
                }
            });
        }

        ready.await();     // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await();      // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
      
        /*
          ready.await();
          long startNanos = System.nanoTime();
          start.countDown();
          done.await();
          Thread
          Thread
          Thread
          Thread
          Thread
          Thread
          Thread
          Thread
          Thread
          Thread
          time :618814
         */ 
    }
}
```

- 흐름 이게 맞나?
  - ready.countDown()
  - start.await()
    - 모든 작업자 쓰레드 준비 기다린다.
  - ready.await()
    - 모든 작업자 준비 기다린다.
  - start.countDown()
  - done.countDown()
  - done.awit() 
    - 모든 작업자의 작업이 끝나기를 기다린다.

- action.run()이 하나하나 찍히는게 아니라 한번에 쫙
  - ready가 빠지면 모든 작업자가 동시에 작업을 시작하지 않는다.

- 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 간단한 프레임워크
  - 타이머 스레드가 시계를 시작하기 전에 모든 작업자 스레드는 동작을 수행할 준비를 마친다.
    - 타이머 스레드가 시계를 시작
      - 메인 스레드의 동작 `long startNanos = System.nanoTime();` 이건가? 
    - 모든 작업자 스레드는 동작을 수행할 준비
      - ready.countDown()
      - ready.await()
  - 마지막 작업자 스레드가 준비를 마치면 타이머 스레드가 '시작 방아쇠'를 당겨 작업자 스레드들이 일을 시작하게 한다.
    - 마지막 작업자 스레드가 준비를 마치면
      - ready.await()
    - 이머 스레드가 '시작 방아쇠'를 당겨
      - start.countDown()
    - 작업자 스레드들이 일을 시작
      - done.countDown()
  - 마지막 작업자 스레드가 동작을 마치자마자 타이머 스레드는 시계를 멈춘다.
    - 마지막 작업자 스레드가 동작을 마치자마자
      - done.awit() 

- ready 래치는 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지할 때 사용한다.
- 통지를 끝낸 작업자 스레드들은 두 번째 래치인 start가 열리기를 기다린다.
- 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시간을 기록하고 start.countDown을 호출하여 기다리던 작업자 스레드들을 깨운다.
  - 마지막 작업자 스레드가 ready.countDown을 호출
    - ready.await()가 마지막 작업자 스레드가 ready.countDown을 호출 할 때 까지 기다릴거 아녀?
  - 타이머 스레드가 시작 시간을 기록
    - long startNanos = System.nanoTime();
- 왜 타이머 스레드라는 얘매한 용어를 사용하지?
  - 내가 메인 스레드에서 실행시키면 시간을 기록하는 메인 스레드가 타이머 스레드인가?



```java
// 코드 81-4 wait 메서드를 사용하는 표준 방식
synchronized (obj) {
  	while (<조건이 충족되지 않았다>)
      	obj.wait(); // (락을 놓고, 깨어나면 다시 잡는다.)
  
  	... // 조건이 충족됐을 때의 동작을 수행한다.
}
```





# 아이템 82. 스레드 안정성 수준을 문서화하라

> To summarize, every class should clearly document its thread safety properties with a carefully worded prose description or a thread safety annotation. The synchronized modifier plays no part in this documentation. Conditionally thread- safe classes must document which method invocation sequences require external synchronization and which lock to acquire when executing these sequences. If you write an unconditionally thread-safe class, consider using a private lock object in place of synchronized methods. This protects you against synchronization interference by clients and subclasses and gives you more flexibility to adopt a sophisticated approach to concurrency control in a later release.

> 모든 클래스가 자신의 스레드 안정성 정보를 명확히 문서화해야 한다. 정확한 언어로 명확히 설명하거나 스레드 안전성 애너테이션을 사용할 수 있다. synchronized 한정자는 문서화와 관련이 없다. 조건부 스레드 안전 클래스는 메서드를 어떤 순서로 호출할 때 외부 동기화가 요구되고, 그때 어떤 락을 얻어야 하는지도 알려줘야 한다. 무조건적 스레드 안전 클래스를 작성할 때는 synchronized 메서드가 아닌 비공개 락 객체를 사용하자. 이렇게 해야 클라이언트나 하위 클래스에서 동기화 메커니즘을 깨뜨리는 걸 예방할 수 있고, 필요하다면 다음에 더 정교한 동시성을 제어 메커니즘으로 재구현할 여지가 생긴다.




비공개 락 객체 private lock object 



```java
// 코드 82-1 비공개 락 객체 관용구 - 서비스 거부 공격을 막아준다.
private final Object lock = new Object();

public void foo() {
  	synchronized(lock) {
      	...
    }
}
```





# 아이템 83. 지연 초기화는 신중히 사용하라

> In summary, you should initialize most fields normally, not lazily. If you must initialize a field lazily in order to achieve your performance goals or to break a harmful initialization circularity, then use the appropriate lazy initialization technique. For instance fields, it is the double-check idiom; for static fields, the lazy initialization holder class idiom. For instance fields that can tolerate repeated initialization, you may also consider the single-check idiom.

> 대부분의 필드는 지연시키지 말고 곧바로 초기화해야 한다. 성능 때문에 혹은 위험한 초기화 순환을 막기 위해 꼭 지연 초기화를 써야 한다면 올바른 지연 초기화 기법을 사용하자. 인스턴스 필드에는 이중검사 관용구를, 정적 필드에는 지연 초기화 홀더 클래스 관용구를 사용하자. 반복해 초기화해도 괜찮은 인스턴스 필드에는 단일검사 관용구도 고려 대상이다.





이중검사 관용구 double-check idiom

지연 초기화 홀더 클래스 관용구 lazy initialization holder class idiom

단일검사 관용구  single-check idiom



```java
// 코드 83-1 인스턴스 필드를 초기화하는 일반적인 방법
private final FieldType field1 = computeFieldValue();
```



```java
// 코드 83-2 인스턴스 필드의 지연 초기화 - synchronized 접근자 방식
private FieldType field2;
private synchronized FieldType getField2() {
    if (field2 == null)
      	field2 = computeFieldValue();
    return field2;
}
```



```java
// 코드 83-3 정적 필드용 지연 초기화 홀더 클래스 관용구
private static class FieldHolder {
  	static final FieldType field = computeFieldValue();
}

private static FieldType getField() { 
  	return FieldHolder.field; 
}
```







```java
// 코드 83-4 인스턴스 필드 지연 초기화용 이중검사 관용구
private volatile FieldType field4;

private FieldType getField4() {
    FieldType result = field4;
    if (result != null)    // 첫 번째 검사 (락 사용 안 함)
        return result;

    synchronized(this) {
        if (field4 == null) // 두 번째 검사 (락 사용)
            field4 = computeFieldValue();
        return field4;
    }
}
```



```java
// 코드 83-5 단일검사 관용구 - 초기화가 중복해서 일어날 수 있다!
private volatile FieldType field5;

private FieldType getField5() {
    FieldType result = field5;
    if (result == null)
        field5 = result = computeFieldValue();
    return result;
}

private static FieldType computeFieldValue() {
    return new FieldType();
}
```









# 아이템 84. 프로그램의 동작을 스레드 스케줄러에 기대지 말라

> In summary, do not depend on the thread scheduler for the correctness of your program. The resulting program will be neither robust nor portable. As a corollary, do not rely on Thread.yield or thread priorities. These facilities are merely hints to the scheduler. Thread priorities may be used sparingly to improve the quality of service of an already working program, but they should never be used to “fix” a program that barely works.

> 프로그램의 동작을 스레드 스케줄러에 기대지 말자. 견고성과 이식성을 모두 해치는 행위다. 같은 이유로, Thread.yield와 스레드 우선순위에 의존해서도 안 된다. 이 기능들은 스레드 스케줄러에 제공하는 힌트일 뿐이다. 스레드 우선순위는 이미 잘 동작하는 프로그램의 서비스 품질을 높이기 위해 드물게 쓰일 수는 있지만, 간신히 동작하는 프로그램을 '고치는 용도'로 사용해서는 절대 안 된다.





```java
// 코드 84-1 끔찍한 CountDownLatch 구현 - 바쁜 대기 버전!
public class SlowCountDownLatch {
    private int count;

    public SlowCountDownLatch(int count) {
        if (count < 0)
            throw new IllegalArgumentException(count + " < 0");
        this.count = count;
    }

    public void await() {
        while (true) {
            synchronized(this) {
                if (count == 0)
                    return;
            }
        }
    }
    public synchronized void countDown() {
        if (count != 0)
            count--;
    }
}
```







