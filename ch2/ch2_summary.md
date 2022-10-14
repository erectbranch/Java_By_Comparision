# 2 코드 스타일 레벨 업


## 2.1 매직 넘버를 상수로 대체

특별한 맥락 없이 프로그램을 제어하기 위해 코드 안에 숫자를 삽입하는 경우가 있다. 이런 매직 넘버는 코드를 이해하기 어렵게 만들고, 오류를 만들기도 한다.

```Java
class CruiseControl {

    private double targetSpeedKmh;

    void setPreset(int speedPreset) {
        if(speedPreset == 2) {
            setTargetSpeedKmh(16944);
        } else if(speedPreset == 1) {
            setTargetSpeedKmh(7667);
        } else if(speedPreset == 0) {
            setTargetSpeedKmh(0);
        }
    }

    void setTargetSpeedKmh(double speed) {
        targetSpeedKmh = speed;
    }

}
```

위 예제는 크루즈 제어를 묘사한다. setPreset()을 정수와 함께 호출해 targetSpeedKmh를 설정하면서 CruiseControl을 구체 값(concrete value)로 변환한다. 이 방법은 오류가 발생하거나 오용되기 십상이다.

아래는 이 코드를 향상시킨 버전이다.

```Java
class CruiseControl {

    // static: 클래스 변수(상수 선언이므로 대문자 표기)
    // CruiseControl 클래스의 인스턴스를 생성하지 않고도 클래스 이름.변수로 사용할 수 있다.
    // final: 변경할 수 없게 한다.
    static final int STOP_PRESET = 0;
    static final int PLANETARY_SPEED_PRESET = 1;
    static final int CRUISE_SPEED_PRESET = 2;

    static final double CRUISE_SPEED_KMH = 16994;
    static final double PLANETARY_SPEED_KMH = 7667;
    static final double STOP_SPEED_KMH = 0;


    private double targetSpeedKmh;


    void setPreset(int speedPreset) {
        if(speedPreset == CRUISE_SPEED_PRESET) {
            setTargetSpeedKmh(CRUISE_SPEED_KMH);
        } else if(speedPreset == PLANETARY_SPEED_PRESET) {
            setTargetSpeedKmh(PLANETARY_SPEED_KMH);
        } else if(speedPreset == STOP_PRESET) {
            setTargetSpeedKmh(STOP_SPEED_KMH);
        }
    }


    void setTargetSpeedKmh(double speed) {
        targetSpeedKmh = speed;
    }

}
```

이렇게 각 숫자마자 유의미한 이름을 달아서 매직 넘버를 제거했다.

코드에 사용 가능한 사전 설정 옵션과 타깃 속도를 나타내는 변수를 추가했다.(static과 final) 즉, 상수(constant)다. 두 한정어(modifier)는 변수를 딱 한 번만 존재하게(static) 하고 변경할 수 없게(final) 강제한다.

static 한정자는 두말할 나위 없이 최적화다. 변수를 CruiseControl 인스턴스와 결합시켜야 할 이유가 있을 수 있다. 어떤 경우라도 final로 선언하는 것이 좋다. 그렇지 않다면 가변이 되어 언제든지 바뀔 수 있다.


## 2.2 정수 상수 대신 열거형

하지만 여기서 더 향상시킬 수 있다. 매직 넘버보다 상수가 낫다는 사실을 확인했지만, 옵션을 모두 열거할 수 있다면 자바 타입 시스템이 제공하는 방법이 더 낫다.

입력 매개변수인 speedPreset은 정수(int)다. 즉 어떤 정수라도, 음수 값이라도 setPreset()에 넣을 수 있다는 뜻이다. 상수에서 정한 STOP_PRESET과 같은 수를 사용해야 한다고 <U>강제하지 않았다.</U>

따라서 **유효하지 않은 정수값이 setPreset()에 입력되면 메서드는 딱히 하는 일이 없다.** 조건을 확인하고 상태를 변경하지도 않고, 오류를 딱히 내놓지도 않고 그냥 반환한다. 충돌보다는 낫지만 버그가 생기기 쉬운 구조이다.

아래는 이를 개선한 코드다.

```Java
class CruiseControl {

    private double targetSpeedKmh;

    void setPreset(SpeedPreset speedPreset) {    // enum

        Objects.requireNonNull(speedPreset);     // null 검사

        setTargetSpeedKmh(speedPreset.speedKmh); // 통과한 값을 넘겨줌
    
    }

    void setTargetSpeedKmh(double speedKmh) {    // 값을 넘겨주는 부분
        targetSpeedKmh = speedKmh;
    }

}


enum SpeedPreset {

    STOP(0), PLANETARY_SPEED(7667), CRUISE_SPEED(16944);    // 가능한 모든 옵션

    final double speedKmh;

    SpeedPreset(double speedKmh) {
        this.speedKmh = speedKmh;
    }

}
```

위에서 쓴 자바 타입 시스템은 가끔 들어오는 유효하지 않은 입력값을 막는 데 큰 역할을 한다. **가능한 모든 옵션을 열거할 수 있는 경우라면 항상 정수 대신 enum을 사용**하는 것이 좋다.

SpeedPreset이라는 새 enum을 생성하고 여기에 주어진 인스턴스와 speedKmh를 저정하는 변수 하나를 넣었다.


## 2.3 For 루프 대신 For-Each

자료 구조를 순회하는 방법은 다양하다. 지금 볼 예제는 checks라는 list 자료 구조를 순회하는 코드다. for 루프를 사용하며 인덱스 변수 i로 checks를 순회한다. 매우 전통적인 순회 방법이며, 인덱싱된 컬렉션이면 어떤 종류라도 동작한다.(Set이나 Map 제외)

```Java
class LaunchCheckList {

    List<String> checks = Arrays.asList("Cabin Pressure",    // 순회할 List
         "Communication",
         "Engine");

    Status prepareForTakeoff(Commander commander) {
        for(int i = 0; i < checks.size(); i++) {

            boolean shouldAbortTakeoff = commander.isFalling(checks.get(i));    // 인덱스 i에 해당하는 값을 리스트에서 가져온다.

            if (shouldAbortTakeoff) {
                return Status.ABORT_TAKE_OFF;
            }

        }

        return Status.READY_FOR_TAKE_OFF;

    }

}
```

위 예제에서는 리스트 내 다음 원소에 접근할 때가 아니면 인덱스를 쓰지 않는다. 즉, **인덱스를 계속 추적할 필요가 없다.** 사실 인덱스 변수가 제공하는 정보를 자세히 알아야 할 경우는 드물다.

게다가 인덱스 변수는 실수할 여지가 있다. protected가 아니므로 언제나 덮어쓸 수 있다. 또한 초보자인 경우 종료 기준을 틀리기 쉬운데(<나 <= 중 무엇을 쓸지 고민하는 등) IndexOutOfBoundsExceptions이 일어나면 당황스러울 것이다.

아래는 이를 개선한 코드다.

```Java
class LaunchCheckList {

    List<String> checks = Arrays.asList("Cabin Pressure",    // 순회할 List
         "Communication",
         "Engine");

    Status prepareForTakeoff(Commander commander) {
        for(String check : checks) {    // checks 내 각 check에 대해 다음을 수행하라
                                        // 컬렉션 내 항목에 접근하는 지역 변수 String check

            boolean shouldAbortTakeoff = commander.isFalling(checks);

            if (shouldAbortTakeoff) {
                return Status.ABORT_TAKE_OFF;
            }

        }

        return Status.READY_FOR_TAKE_OFF;

    }

}
```

for(타입 변수명 : 복수명)으로 문법을 바꿔서 이해하기 쉬워졌다. 'checks 내 각 check에 대해 다음을 수행하라'로 읽으면 된다. 컬렉션 내 항목에 접근하는 지역 변수인 String check를 정의한 뒤 이어서 콜론과 순회할 자료 구조인 checks를 넣었다.

매 반복마다 자바는 자료 구조에서 새로운 객체를 가져와 checks에 할당한다. 반복 인덱스를 더 이상 다루지 않아도 된다. 심지어 배열과 Set처럼 인덱싱되지 않은 컬렉션에도 작동한다.

**사실 인덱스로 순회하는 전통적인 방식이 정답일 때는 거의 없다.** 아주 가끔씩 인덱스 기반 순회가 적절한 경우는 <U>'컬렉션의 특정 부분만 순회'하거나 '특정 목적으로 인덱스가 필요할 때'</U>뿐이다.

또 다른 순회 메커니즘으로 반복자(iterator)를 사용하는 방법이 있다.(엄밀히 말해 for-each 루프도 iterator에 기반하지만, 여러가지를 숨기는 방식으로 설계가 되었기 때문에 수행할 수 없는 작업들이 있다.)


## 2.4 순회하며 컬렉션 수정하지 않기

코드에서는 항상 배열이나 리스트를 비롯해 다양한 자료 구조를 순회한다. 대부분 자료 구조를 읽기만 한다. 예를 들어 주문 항목 리스트에서 송장을 만들거나 이름을 찾는 등의 작업을 한다. 하지만 자료 구조를 바꿔야 할 때가 있으며, 이 작업은 주의해서 진행해야 한다.

아래 예제는 간단한 자료 구조 supplies list를 순회하는 코드다. 재고가 변질되었으면 list에서 항목을 제거할 것이다.

```Java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    void disposeContaminatedSupplies() {

        for(Supply supply : supplies) {

            if(supply.isContaminated()) {    // 재고가 변질되었으면
                supplies.remove(supply);     // 리스트에서 항목을 제거한다.
            }

        }

    }

}
```

언뜻 보면 위 코드는 문제가 없어 보인다. 하지만 재고 목록 내 한 supply라도 오염이 되었을 경우 무조건 충돌한다. 더 큰 문제는 오염된 supply가 나오기 전까지는 정상적으로 잘 작동한다는 점이다. 따라서 더 오류를 확인하기 힘들다.

코드에서 문제가 되는 부분은 for 루프 내 supplies.remove(supply)를 호출하는 부분이다. 이렇게 실행할 경우 List 인터페이스의 표준 구현이나 Set, Queue와 같은 콜렉션 인터페이스의 구현은 ConcurrentModificationException을 던진다. **List를 순회하면서 List를 수정할 수 없다.**

아래는 이를 수정한 코드다.

```Java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    void disposeContaminatedSupplies() {
        
        iterator<Supply> iterator = supplies.iterator();

        while (iterator.hasNext()) {

            if(iterator.next().isContaminated()) {
                iterator.remove();
            }
        
        }
    
    }

}
```

문제를 해결할 방법 중 하나로는 리스트를 순회하며 오염된 제품을 다 찾고 난 뒤, 순회가 끝나면 제거하는 것이다. 탐색한 뒤 제거하는 2단계를 거치므로, 동작은 잘 되지만 시간과 메모리가 더 소모된다.

수정한 코드는 supplies 컬렉션의 <U>Iterator를 활용하는 while 루프</U>라는 새로운 순회 방식을 사용한다. **Iterator는 첫 번째 원소에서 시작해 리스트 내 원소를 가리키는 포인터처럼 동작한다.** hasNext()를 통해 원소가 남아 있는지 묻고, next()로 다음 원소를 얻은 뒤 가장 최근 얻은 원소를 remove()하며 제거한다. 

CopyOnWriteArrayList와 같은 특수 List 구현은 순회하며 수정하기도 한다. 하지만 대가가 따르는데, 리스트에 원소를 삽입하거나 제거할 때마다 매번 전체 리스트를 복사한다.(별개로 자바 8부터는 람다를 사용하는 Collection.removeIF() 메서드도 사용할 수 있다.)


## 2.5 순회하며 계산 집약적 연산하지 않기

자료 구조를 순회할 때는 수행할 연산 유형에 주의해야 한다. 아래 코드는 정규식으로 Supply 객체를 찾는 find() 메서드의 전형적인 예다.

```Java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    List<Supply> find(String regex) {   // 정규식(regular expression), 줄여서 regex로 쿼리 문자열을 만든다.

        List<Supply> result = new LinkedList<>();

        for(Supply supply : supplies) {

            if(Pattern.matches(regex, supply.toString())) {
                result.add(supply);
            }

        }

        return result;

    }

}
```

자바를 비롯해 다양한 프로그래밍 언어에서 정규식(regular expression), 짧게 줄여 regex로 쿼리 문자열을 만든다. 정규식이 있으면 방대한 텍스트 데이터 집합을 효율적으로 다룰 수 있다.

이렇게 코드를 작성할 경우 문제가 생긴다. Pattern.matches(regex, supply.toString()는 예제를 반복할 때마다 정규식을 컴파일하게 된다. 따라서 시간과 처리 전력을 소모하게 된다. 

아래는 정규식을 반복해 컴파일하지 않도록 설계한 코드다.

```Java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    List<Supply> find(String regex) {

        List<Supply> result = new LinkedList<>();

        Pattern pattern = Pattern.compile(regex);

        for(Supply supply : supplies) {

            if(pattern.matcher(supply.toString()).matches()) {
                result.add(supply);
            }

        }

        return result;
    
    } 

}
```

바뀐 코드에서는 메서드를 호출할 때 정규식을 딱 한 번만 컴파일하면 된다. 루프를 반복해도 표현식 문자열은 바뀌지 않기 때문이다. 

다행히 Pattern API로 한번에 쉽게 컴파일 할 수 있다. Pattern.matches() 호출에 있는 두 연산을 분해했는데, 첫 번째는 표현식 컴파일이고 두 번째는 검색 문자열을 실행하는 부분이다.

첫 번째 단계는 Pattern.compile() 호출로 추출해 Pattern의 인스턴스인 컴파일된 정규식을 생성한다. 계산이 많이 필요한 단계이므로 이 결과를 지역변수에 저장한다.

두 번째 단계는 컴파일된 표현식을 실행한다. 쉽고 빠른 연산이며, 매 Supply 인스턴스마다 실행해야 하는 연산이므로 루프 본문에 넣는다.

예제에서는 검색할 String을 위한 Matcher를 먼저 생성했다. **정규식을 조금만 고쳐 사용하는 것만으로 성능을 크게 높일 수 있다.**


## 2.6 새 줄로 그루핑

코드 블록이 서로 붙어 있으면 보통 한 덩어리로 간주한다. 별개 블록을 새 줄로 분리하는 것만으로 코드 이해도가 향상될 수 있다.

아래 예제는 마일과 킬로미터 간 변환률을 반환하는 enum이다. 코드 시멘틱은 괜찮지만 문제가 숨어 있다. 

```Java
enum DistanceUnit {

    MILES, KILOMETERS;

    static final double MILE_IN_KILOMETERS = 1.60934;
    static final int IDENTITY = 1;
    static final double KILOMETER_IN_MILES = 1 / MILE_IN_KILOMETERS;

    double getConversionRate(DistanceUnit unit) {
        
        if(this == unit) {
            return IDENTITY;
        }
        if(this == MILES && units == KILOMETERS) {
            return MILE_IN_KILOMETERS;
        } else {
            return KILOMETER_IN_MILES;
        }

    }

}
```

문제는 바로 여백이 빠졌다는 점이다. 무엇보다 getConversionRate() 내 코드가 서로 붙어 있다. 이 함수는 정말 하나의 블록이 맞는가?

아래는 코드를 분리한 것이다.

```Java
enum DistanceUnit {

    MILES, KILOMETERS;

    // IDENTITY 필드를 다른 상수들과 분리
    // 마일, 킬러미터와 같은 단위와 상관이 없고 추상적이다.
    static final int IDENTITY = 1;

    static final double MILE_IN_KILOMETERS = 1.60934;
    static final double KILOMETER_IN_MILES = 1 / MILE_IN_KILOMETERS;


    double getConversionRate(DistanceUnit unit) {
        
        if(this == unit) {
            return IDENTITY;
        }

        // 확인하는 사항이 다른 if 블록을 분리
        if(this == MILES && units == KILOMETERS) {
            return MILE_IN_KILOMETERS;
        } else {
            return KILOMETER_IN_MILES;
        }

    }

}
```


## 2.7 이어붙이기 대신 시각화

아래는 별로 복잡하지는 않지만 읽기 어려운 코드 예제다. 1.5절의 Logbook 클래스를 이용한다.

```Java
class Mission {

    Logbook logbook;
    LocalData start;

    void update(String author, String message) {
        LocalDate today = LocalDate.now();
        String month = String.valueOf(today.getMonthValue());
        String formattedMonth = month.length() < 2 ? "0" + month : month;
        String entry = author.toUpperCase() + ":[" + formattedMonth + "-" + today.getDayOfMonth() + "-" + today.getYear() + "](Day " + (ChronoUnit.DAYS.between(start, today) + 1) + ")>" + message + System.lineSeperator();
        logbook.write(entry);
    }

}
```

코드가 생성할 출력도 이해와 가독성 면에서 굉장히 중요하다. 긴 문자열을 생성할 때 서식 문자열을 이용하면 더 읽기 쉽게 만들 수 있다.

예제는 String 변수가 여러 개이고 어떤 변수는 자바의 인라인(in-line) 표기까지 사용한다. 심지어 int 값을 합치는 인라인 계산 실행도 있어서 혼란이 가중된다. 

이 코드는 대폭 간소화할 수 있다.

```Java
class Mission {

    Logbook logbook;
    LocalDate start;

    void update(String author, String message) {
        final LocalDate today = LocalDate now();
        String entry = String.format("%S: [%tm-%<te-%<tY](Day %d)> %s%n", 
                                     author, today, 
                                     ChronoUnit.DAYS.between(start, today) + 1, message);
        logbook.write(entry);
    }

}
```

서식 문자열은 %로 표기하는 특수 위치 지정자(placeholder) 문자를 사용해 하나의 블록으로 일관된 String을 정의한다.

예제에서 %S는 toString() 메서드를 사용해 객체를 대문자 String으로 변환한다. 매개변수인 author을 이렇게 처리한다. 날짜 변수인 today는 월을 나타내는 %tm, 날짜를 나타내는 %te, 연도를 나타내는 %tY에 쓰인다. < 문자를 추가하면서 위치 지정자 세 개가 같은 입력 데이터를 읽는다. %d는 십진 값을 처리하고 %s는 String을 받아 들인다. 마지막으로 %n으로 행바꿈을 적용한다.

String.format()이나 System.out.printf()와 같은 포맷 메서드는 위치 지정자 문자가 포함된 데이터를 String 뒤에 나열한 순서대로 받아들인다. 


## 직접 만들지 말고 자바 API 활용하기

프로그래밍 초창기에는 전부 스스로 만들어야 했다. C언어는 char[]로 String을 생성해야 했고, 리스트도 스스로 구현해야 했다. 실력은 늘겠지만 시간이 많이 걸리고 오류가 발생하기 쉽다. 하지만 시대는 변했고 이제 API에 있는 기능을 다시 구현하지 말고 가능하면 사용해야 한다.

아래 코드를 보자.

```Java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    int getQuantity(Supply supply) {
        
        if(supply == null) {    // 입력 매개변수가 null인지 검증한다.
            throw new NullPointerException("supply must not be null");
        }

        int quantity = 0;

        for(Supply supplyInStock : supplies) {
    
            if(supply.equals(supplyInStock)) {
                quantity++;
            }
    
        }

    return quantity;

    }

}
```

getQuantity() 메서드는 재고 내 Supply 수량을 반환한다. null 값이 없도록 입력 매개변수를 검증하고, 내부 검증 구조로 추상 타입을 사용하고 있다. 하지만 이 코드는 필요 이상으로 너무 장황하다.

```Java
class Inventory {

    private List<Supply> supplies = new ArrayList<>();

    int getQuantity(Supply supply) {
        
        Objects.requireNonNull(supply, "supply must not be null");

        return Collections.frequency(supplies, supply);

    }

}
```

유틸리티 클래스인 Collections는 Collection 내 객체 출현 횟수를 세는 frequency() 메서드를 제공한다. 또한 1.6절에서도 본 Object 유틸리티 클래스의 requireNonNull() 메서드를 사용했다. 이 메서드는 객체가 null이면 메세지와 함께 NullPointerException을 던진다.

이처럼 자바 API를 잘 알고 활용하면 오류를 줄이고 코드를 굉장히 단축시킬 수 있다.


---