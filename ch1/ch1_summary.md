## 1.1 쓸모없는 비교 피기

논리 조건문을 배운 초보들은 종종 불 값으로 조건문을 구성한다. 하지만 이런 비교는 정말 쓸모가 없다! 코드 내 잡음과 마찬가지다.

아래 코드는 Laboratory 컴포넌트 코드로, 중첩 if-else 블록 두 개에 '불 반환값과 불 원시 타입(true, false)를 비교하는 조건문'을 넣어서 Sample을 분석하고 있다. 하지만 불 변수나 반환 타입은 그럴 필요가 사실 없다.

**불 표현식은 불 원시값과 비교하지 않아도 된다!**

```Java
package general.avoid_unneccessary_comparisons.problem;

import general.Result;
import general.Microscope;
import general.Sample;

class Laboratory {

    Microscope microscope;

    Result analyze(Sample sample) {
        if (microscope.isInorganic(sample) == true) {
            return Result.INORGANIC;
        } else {
            return analyzeOrganic(sample);
        }
    }

    private Result analyzeOrganic(Sample sample) {
        if (microscope.isHumanoid(sample) == false) {
            return Result.ALIEN;
        } else {
            return Result.HUMANOID;
        }
    }
}
```

위는 불 표현식을 불 원시값과 비교하면서 어수선하게 되었다. 이를 고치면 아래와 같다.

```Java
class Laboratory {

    Microscope microscope;

    Result analyze(Sample sample) {
        if (microscope.isInorganic(sample)) {
            return Result.INORGANIC;
        } else {
            return analyzeOrganic(sample);
        }
    }

    private Result analyzeOrganic(Sample sample) {
        if (!microscope.isHumanoid(sample)) {    // 부정 연산자(!)를 사용한다.
            return Result.ALIEN;
        } else {
            return Result.HUMANOID;
        }
    }
}
```

어떤 컴파일러든 이렇게 제거할 것이다. 아래는 이 코드에서도 더 개선할 부분을 찾는다.


## 1.2 부정 피하기

방금 예제에서 불 표현식과 불 원시값을 비교하는 필요 없는 부분을 제거했다. 그런데 이 코드는 더 개선할 부분이 남아 있다.

예제에서는 메서드 두 개에서 Sample을 받아 Result를 반환한다. 코드에서 명백히 틀렸다고 말할 부분은 없지만, 필요 이상으로 복잡하다. if 조건문 부분을 보자. 첫 번째 조건문은 sample이 isInorganic()인지 테스트한다. 두 번째 조건문은 불 연산자를 사용해 부정, 즉 "!"로 테스트한다.

**코드를 작성할 때는 일반적으로 긍정 표현이 좋다.**  부정 표현은 간접적인 행동 계층을 하나 더 추가한다. 단순히 "X가 해당된다"가 아니라 "X가 해당되지 않는다"라는 표현을 추가로 이해해야 한다.

아래는 부정 표현을 간소화한 것이다.

```Java
class Laboratory {

    Microscope microscope;

    Result analyze(Sample sample) {
        if (microscope.isOrganic(sample)) {    // 부정 표현인 isInorganic() 대신 isOrganic 호출
            return analyzeOrganic(sample);
        } else {
            return Result.ORGANIC;
        }
    }

    private Result analyzeOrganic(Sample sample) {
        if (microscope.isHumanoid(sample)) {
            return Result.HUMANOID;
        } else {
            return Result.ALIEN;
        }
    }
}
```

첫 번째 부분은 부정 표현인 isInorganic() 대신 isOrganic을 호출하였다. 다시 말해 <U>if와 else 블록 본문을 서로 바꾸었다.</U>

두 번째 부분은 isHumanoid() 함수를 똑같이 호출하되 부정을 제거했다. 마찬가지로 if와 else 블록 본문을 서로 바꾸었다.

부정 표현을 제거하려면 첫 번째 부분(isInorganic 대신 isOrganic을 호출)처럼 적절한 메서드가 필요하기도 하다. 호출하려는 코드가 외부 라이브러리에 있으면 메서드를 호출하지 못할 수도 있다. 하지만 제어할 수만 있다면 적절한 클래스에 메서드를 추가하는 것을 망설이지 마라. **부정적 메서드는 모두 제거하는 것이 가장 좋다.** 


## 1.3 불 표현식을 직접 반환

```Java
class Astronaut {

    String name;
    int missions;

    boolean isValid() {
        if (missions < 0 || name == null || name.trim().isEmpty()) {    // ||는 논리적 OR: 어느 하나가 true라면 true
            return false;    // 불 표현식을 직접 반환한다.
        } else {
            return true;
        }
    }
}
```

위 코드는 전형적인 유효성 검사 방법인데, 객체, 즉 정수와 String의 몇 가지 속성을 확인하고 있다. 

* missions: 화성 탐사 미션의 개수다. 이 수는 음수면 안 된다.

* name: String 속성은 null이면 안 된다. 만약 이 속성이 null이면 NullPointerException이 발생해 어느 시점에 실행이 중지될 수 있다.

* name.trim().isEmpty(): Astronaut는 실제 이름을 가져야 하니 빈 문자여도 안 된다. (name.trim() 호출은 문자열 앞뒤 공백이나 탭과 같은 여백 문자를 모두 제거한다.)

이 코드에 기능상 오류는 없지만 사실 if 문이 없어도 목적을 달성한다. 지저분한 코드를 고치자.

```Java
class Astronaut {

    String name;
    int missions;

    boolean isValid() {
        
        return missions >= 0 && name != null && !name.trim().isEmpty();

    }
}
```

**직접 불 표현식을 반환했다.** 이렇게 바꾸면 한 단계 들여쓰기할 필요도, 분기문을 둘 필요도 없다. 기본적으로 '드 모르간의 법칙'을 적용해 조건문을 부정했다.

* [드 모르간의 법칙](https://bite-sized-learning.tistory.com/358)

> !A && !B == !(A || B)    // 참

> !A || !B == !(A && B)    // 참

* 예시

> 어린이용 놀이 기구를 탈 수 '없는(부정)' 사람이 '20세 이상 혹은 키 140cm 이상'이라면 굉장히 복잡하다. !(A || B)<br>
> 이를 '20세 미만이며 동시에 키 140 미만인 사람만 탈 수 있다(긍정)'으로 보면 훨씬 간단하다. !A && !B

실제 코드로 구현하는 조건문은 이보다 복잡할 수 있다. 만약 그렇다면 조건문을 더 작은 덩어리로 분할하는 방향으로 고려해야 한다. 아래 예제를 보자.

```Java
// 조건문을 더 작은 덩어리로 분할한다.
boolean isValid() {
    boolean isValidMissions = missions >= 0;
    boolean isValidName = name != && !name.trim().isEmpty();
    return isValidMissions && isValidName;
}
```

**조건문을 세 개 이상 합칠 때는 위와 같은 간소화를 고려해야 한다.** 조건문 덩어리를 다른 곳에서도 써야 하면 이어지는 절인 1.4 불 표현식 간소화에서 설명하는 대로 개별 메서드에 넣는 것이 좋다.

해법은 반환 타입이 불일 때만 동작하는 것에 유의하자.


## 1.4 불 표현식 간소화

마찬가지로 여러 조건문이 합쳐진 불 표현식 예제를 보자. 아래 예제의 willCrewSurvive() 메서드는 한 조건문 안에서 낮은 수준의 각 세부 내용을 확인한다.

```Java
class SpaceShip {

    Crew crew;
    FuelTank fuelTank;
    Hull hull;
    Navigator navigator;
    OxygenTank oxygenTank;

    boolean willCrewSurvive() {
        return hull.holes == 0 &&
            fuelTank.fuel >= navigator.requiredFuelToEarth() &&
            oxygenTank.lastsFor(crew.size) > navigator.timeToEarth();
    }
}
```

이렇게 조건문이 많이 합쳐진 코드도 몇 가지 간단한 요령으로 쉽게 만들 수 있다. 조건문은 너무 길면 이해하기 어려우며, 하나로 합쳐야만 한다면 다른 식으로 묶는 것이 낫다.

한 메서드 안에서는 추상화 수준이 비슷하도록 명령문을 합쳐야 한다. 더 높은 수준의 메서드가 다음으로 낮은 수준의 메서드를 호출하는 것이 이상적이다. 

> 불 조건에서는 **괄호**를 잘 활용하라. &&가 항상 ||보다 먼저 평가된다는 사실을 아는가? 많은 개발자들이 불 연산자 우선순위를 따로 기억하지 못하며, x && y || z가 어느 것부터 연산이 될지 바로 파악하지 못한다.

```Java
// 불 표현식 간소화
class SpaceShip {

    Crew crew;
    FuelTank fuelTank;
    Hull hull;
    Navigator navigator;
    OxygenTank oxygenTank;

    boolean willCrewSurvive() {

        boolean hasEnoughResources = hasEnoughFuel() && hasEnoughOxygen();
        
        return hull.inIntact() && hasEnoughResources;
    
    }

    private boolean hasEnoughOxygen() {
        return oxygenTank.lastsFor(crew.size) > navigator.timeToEarth();
    }

    private boolean hasEnoughFuel() {
        return fuelTank.fuel >= navigator.requiredFuelToEarth();
    }

}
```

willCrewSurvive() 메서드는 그대로지만, 이제 다른 메서드를 호출해 그 반환값을 집계한다.

먼저 불 변수 hasEnoughResources를 추가해 소모성 자원이라는 주제에 맞게 유사한 특징을 한데 묶었다. 이 변수는 hasEnoughFuel()과 hasEnoughOxygen() 메서드 두 개를 호출해 결과를 가져온다. 

다음으로 고려하지 못한 hull.holes == 0 조건 덩어리와 새로 만든 불 변수 hasEnoughResources를 합쳤다. 여기서는 Hull 클래스의 hull.isIntact() 메서드를 사용했다. 이미 의미 있는 이름이 있으므로 다른 불 변수에 저장할 이유는 없다.

코드 행은 비록 늘었지만 코드가 훨씬 이해하기 편해졌다. 그뿐만 아니라 변수명과 메서드명으로 결과가 무엇을 나타내는지 알기 쉬워졌다. 메서드도 간단하게 구성되었기 때문에 편리하다.


## 1.5 조건문에서 NullPointerException 피하기

NullPointerException은 <ㅕ>null을 참조하는 메서드를 호출하거나 속성에 접근할 때</U> 발생한다. 이러한 문제를 막으려면 메서드 인수가 유효한지 검사해야 한다.

```Java
class Logbook {    // location 인수로 명시한 파일 시스템 내 특정 파일에 로그 메시지를 기록한다.

    void writeMessage(String message, Path location) throws IOException {
        if(Files.isDirectory(location)) {
            throw new IllegalArgumentException("The path is invalid!");
        }

        if(message.trim().equals("") || message == null) {
            throw new IllegalArgumentException("The message is invalid!");
        }

        String entry = LocalDate.now() + ": " + message;

        Files.write(location, Collections.singletonList(entry),
            StandardCharset.UTF_8, StandardOpenOption.CREATE,
            StandardOpenOption.APPEND);
    }

}
```

위 메서드에서 유효성 검사를 일부 수행하고 있지만 두 가지 심각한 문제를 지니고 있다. 

먼저 위 메서드는 null 참조를 올바르게 확인하지 않는다. location이 null이면 Files.isDirectory()는 별다른 설명 없이 NullPointerException과 함께 실패한다.

message가 null이면 message.equals("")를 먼저 확인하니 두 번째 조건문도 마찬가지다.

인수를 검증할 때는 순서가 중요한데, **반드시 null을 먼저 확인한 후** 도메인에 따라 "유효하지 않은" 값을 검사해야 한다. <U>다시 말해 빈 문자열이나 빈 리스트처럼 일반적인 기본값부터 먼저 검사한 후 특정 값을 확인하는 것이 좋다.</U>

또한 메서드 인수로 null을 전달하는 방식은 메서드가 매개변수 없이도 올바르게 기능한다는 뜻이니 피해야 한다. 꼭 해야 한다면 매개변수가 있는 메서드와 없는 메서드로 리팩터링해라. 대부분 매개변수가 null이면 호출하는 편에서 프로그래밍 오류가 발생했다는 뜻이다.

```Java
// 유효성 검사를 적절히 수행하는 코드
class Logbook {    // location 인수로 명시한 파일 시스템 내 특정 파일에 로그 메시지를 기록한다.

    void writeMessage(String message, Path location) throws IOException {
        // 메서드 서명 내 인수 순서에 따라 검사하도록 순서를 바꾼다.
        // 매개변수로 null을 전달하는지 제일 먼저 검사한다.
        if(message == null || message.trim().isEmpty()) {
            throw new IllegalArgumentException("The message is invalid!");
        }

        if(location == null || Files.isDirectory(location)) {
            throw new IllegalArgumentException("The path is invalid!");
        }

        String entry = LocalDate.now() + ": " + message;

        Files.write(location, Collections.singletonList(entry),
            StandardCharset.UTF_8, StandardOpenOption.CREATE,
            StandardOpenOption.APPEND);
    }

}
```

새로운 코드는 논의한 문제를 모두 해결했다. 우선 모든 인수에 null 값 여부를 확인한다. 이어서 도메인에 따른 제한을 확인한다.

또한 메서드 서명 내 인수 순서에 따라 확인하도록 바꿨다.(읽기 흐름이 크게 향상되는 좋은 습관이다.) 이렇게 하면 어떤 매개변수를 누락할 위험이 거의 없다. 끝으로 내장 매서드를 사용해 빈 문자열인지 확인(isEmpty())했다.

항상 이 정도 수준으로 매개변수 검증을 해야 하는지 의문이 든다면 그렇지는 않다. 경험상 <U>매개변수 검사는 public과 protected, default 메서드</U>에만 하면 된다. 이런 메서드는 코드 어디에서도 접근 가능하고, 접근이 어떻게 일어나는지 제어하기 어렵기 때문이다.


## 1.6 스위치 실패 회피하기

switch 문은 쓸 때 주의해야 한다. 아래 코드의 authorizeUser()는 매개변수를 검증하고 null 참조를 확인한다. 자바 API에 있는 편리한 매개변수 검증 방법인 Objects.requireNonNull()을 사용해 입력이 null이면 예외를 발생시킨다. 하지만 authorizeUser()에는 스위치 실패(switch Fallthrough)라는 고전적인 버그가 남아 있다.

```Java
class BoardComputer {

    CruiseControl cruiseControl;

    void authorize(User user) {

        Objects.requireNonNull(user);    // 매개변수 값이 null이면 예외를 발생시킨다.

        switch (user.getRank()) {

            case UNKNOWN:
                cruiseControl.logUnauthorizedAccessAttempt();
                // switch 문 끝에 break 문을 작성하지 않았기 때문에 버그가 일어난다.(다음 case를 실행하게 된다.)
            
            case ASTRONAUT:
                cruiseControl.grantAccess(user);
                break;

            case COMMANDER:
                cruiseControl.grantAccess(user);
                cruiseControl.grantAdminAccess(user); 
                break;

        }

    }

}
```

위 코드는 switch 문의 첫 번째 case에서 break 문이 없기 때문에 버그가 일어난다. 첫 번재 case는 실패하고 항상 cruiseControl.grantAccess()를 실행할 것이다.

switch 문은 이런 동작으로 악명이 높다. break 문 또는 블록 끝에 다다라야 실행을 멈추기 때문이다. 

```Java
class BoardComputer {

    CruiseControl cruiseControl;

    void authorize(User user) {

        Objects.requireNonNull(user);    // 매개변수 값이 null이면 예외를 발생시킨다.

        switch (user.getRank()) {

            case UNKNOWN:
                cruiseControl.logUnauthorizedAccessAttempt();
                break;
            
            case ASTRONAUT:
                cruiseControl.grantAccess(user);
                break;

            case COMMANDER:
                cruiseControl.grantAccess(user);
                cruiseControl.grantAdminAccess(user); 
                break;

        }

    }

}
```

break 문을 제대로 삽입했다. 이제 코드는 버그가 없지만 이게 완벽한 대안이냐고 하면 그렇지 않다. 예제에서 switch는 원래는 분리해야 할 두 가지 관심사를 섞고 있다. (허가받지 않은 접근과 허가된 접근을 함께 다루고 있다.)

이런 식으로 **서로 다른 관심사는 묶기보다 다른 코드 블록에 넣어야 한다.** 가독성과 버그 양쪽 면에서 분리하는 것이 이득이다.

switch 문은 관심사를 분리하기 어렵기 때문에, 보통 if 문 사용을 권장한다. 


## 1.7 항상 괄호 사용하기

이제 볼 예제는 위에서 다룬 예제를 if 문으로 바꾼 코드다.

```Java
class BoardComputer {

    CruiseControl cruiseControl;

    void authorize(User user) {

        Objects.requireNonNull(user);    // 매개변수 값이 null이면 예외를 발생시킨다.

        if(user.isUNKNOWN())
            cruiseControl.logUnauthorizedAccessAttempt();

        if(user.isASTRONAUT())
            cruiseControl.grantAccess(user);

        if(user.isCOMMANDER())
            cruiseControl.grantAccess(user);
            cruiseControl.grantAdminAccess(user); 
            
        }

    }

}
```