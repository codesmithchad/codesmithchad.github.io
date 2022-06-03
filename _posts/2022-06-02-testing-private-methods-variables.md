---
layout: post
title: Testing private methods and variables in Swift from SwiftLee
subtitle: raywenderlich.com을 참고하여 TDD를 학습해 본다.
# cover-img: /assets/img/toastMessageView.png
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [iOS, test, tdd, xctest, unit test, ui test]
---

> ref: [Testing private methods and variables in Swift](https://www.avanderlee.com/swift/testing-private-methods-variables/)

테스트 코드를 작성할때 프라이빗 메소드와 변수를 종종 마주하게 된다.\
코드들이 예상대로 동작하는지 확인하여야 하거나 100% 코드 커버리지에 도움이 되지 않을까 생각할 수 있다.\
\
스위프트 유닛 테스트 뉴비라면 [“Unit tests best practices in Xcode and Swift”](https://www.avanderlee.com/swift/unit-tests-best-practices/)를 먼저 보고 와라.

이 포스팅을 시작하기 전에 나는 프라이빗 함수나 변수를 확실하게 접근할 수 있는 정답은 없다고 말하고 싶다.\
변칙적인 방법으로 프라이빗 메소드를 드러낼 수 있지만 테스트만을 위한 더 심각한 문제를 발생시킬 수 있는 “code smell”이 추가될 것이다.\
대신 당신의 관점을 바꾸고 프라이빗 함수와 상수를 테스트할 필요가 없다는 것을 지금이 아닌 마지막에 보여주고 싶다.

## 100% Code coverage should not be your goal

100% 코드 커버리지가 목표가 되어서는 안된다.
양질의 테스트를 작성하는것이 낫다.
특히 테스트를 작성할 여유가 없는경우 100%에 도달하려 하지 않는것이 좋다.
시간을 아낀다는 점에서도 이득이라 느낄 것이다.

&nbsp;

## How do I verify that my private code is working?

> 프라이빗 코드가 정상 동작한다고 어떻게 장담하지?

종종 100% code coverage에 연연하지 않더라도 시나리오 상 특정 프라이빗 코드의 유효성을 확인하고자 할 것이다.\
\
퍼블릭 api를 통해 모든 프라이빗 코드는 접근이 가능해야 한다.\
만약 모두 프라이빗으로 작성되었다면 그 코드는 어디곳에서도 사용하지 못할것이다.\
\
당신의 코드는 오직 클래스 내에서만 호출되는 경우가 있고 이 경우 클래스 이니셜라이저에서 활성화 된다.\
\
아래의 앱이 포그라운드 혹은 백그라운드 진입시 트래커 시그널을 보내는 세션 모니터의 예제코드를 살펴보자.\

```swift
final class Tracker {
    private var hasActiveSession: Bool = false

    func startNewSession() {
        hasActiveSession = true
    }

    func stopCurrentSession() {
        hasActiveSession = false
    }
}

final class SessionMonitor {

    private let tracker: Tracker

    init(tracker: Tracker) {
        self.tracker = tracker
        startObserving()
    }

    private func startObserving() {
        NotificationCenter.default.addObserver(self, selector: #selector(startSession), name: UIApplication.willEnterForegroundNotification, object: nil)
        NotificationCenter.default.addObserver(self, selector: #selector(endSession), name: UIApplication.didEnterBackgroundNotification, object: nil)
    }

    @objc private func startSession() {
        tracker.startNewSession()
    }

    @objc private func endSession() {
        tracker.stopCurrentSession()
    }
}
```

이 경우 모든 로직들이 잘 분리되었고 최소한의 내용만 노출되도록 하였다.\
\
willEnterForegroundNotification와 didEnterBackgroundNotification가 호출될 때 스타트 세션과 엔드 세션 메소드가 제대로 호출되는지 확인하고 싶다.\
\
이를 달성하기 위한 방법은 여러가지가 있다.\
tracker와 hasActiveSession를 non private으로 변경하면 유닛 테스트가 읽는다.\
이것은 동작하는데 문제가 없고 우리가 직면한 문제를 해결하는 가장 빠른 방법이지만 잘 나눠놓은 api로부터는 멀어지게 된다.\
이렇게 하는 대신 새로운 프로토콜을 소개하고 의존성 주입을 통해 동작하고 하겠다.


### Dependency injection as a solution to validating private code

의존성 주입은 mock버전의 특정 인스턴스를 주입함으로써 프라이빗 코드를 확인이 가능하도록 해준다. \
지금의 경우 mock 버전의 Tracker 인스턴스를 만드는 것을 의미한다.\
\
이를 위해 우선 소개할 것은 SessionTracking 프로토콜 이다.

```swift
protocol SessionTracking {
    func startNewSession()
    func stopCurrentSession()
}
```

그리고 나면 이 프로토콜을 따르는 Tracker 인스턴스를 만들 수 있다.

```swift
final class Tracker: SessionTracking {
    private var hasActiveSession: Bool = false

    func startNewSession() {
        hasActiveSession = true
    }

    func stopCurrentSession() {
        hasActiveSession = false
    }
}
```

그리고 마지막으로 새로 정의한 프로토콜이 적용된 SessionMoniter 클래스를 만들 수 있다.

```swift
final class SessionMonitor {

    private let tracker: SessionTracking

    init(tracker: SessionTracking) {
        self.tracker = tracker
        startObserving()
    }

    private func startObserving() {
        NotificationCenter.default.addObserver(self, selector: #selector(startSession), name: UIApplication.willEnterForegroundNotification, object: nil)
        NotificationCenter.default.addObserver(self, selector: #selector(endSession), name: UIApplication.didEnterBackgroundNotification, object: nil)
    }

    @objc private func startSession() {
        tracker.startNewSession()
    }

    @objc private func endSession() {
        tracker.stopCurrentSession()
    }
}
```

이 시점에서 바뀐점은 없다.\
이 코드들은 여전히 이전과 동일하게 잘 동작한다.\
어쨌든, 세션 모니터에서 외부 요청을 감지하는데 사용할 수 있는 mock 버전의 tracker 인스턴스 주입하는 문을 열었다.\
다른 말로 Tracker 인스턴스가 각 노티피케이션으로 부터 불리워 지는지 확인할 수 있게 되었다.

#### MAKING USE OF A MOCKED VERSION OF A CLASS

새 프로토콜을 테스트에서 사용할 수 있도록 SessionTracking을 따르는 MockedTracker 인스턴스를 만들수 있다.

```swift
final class MockedTracker: SessionTracking {
    private(set) var hasActiveSession: Bool = false

    func startNewSession() {
        hasActiveSession = true
    }

    func stopCurrentSession() {
        hasActiveSession = false
    }
}
```


아주 심플하고 기본적이게 `hasActiveSession` 프로퍼티만 true/false로 설정한다.\
\
유닛 테스트에서 이 `MockedTracker`를 아래와 같이 사용할 수 있다.

```swift
final class SessionMonitorTests: XCTestCase {

    private var monitor: SessionMonitor!

    /// It should signal the tracker upon foreground background changes.
    func testSessionRound() {
        let tracker = MockedTracker()
        monitor = SessionMonitor(tracker: tracker)

        XCTAssertFalse(tracker.hasActiveSession)

        NotificationCenter.default.post(name: UIApplication.willEnterForegroundNotification, object: nil)
        XCTAssertTrue(tracker.hasActiveSession)

        NotificationCenter.default.post(name: UIApplication.didEnterBackgroundNotification, object: nil)
        XCTAssertFalse(tracker.hasActiveSession)
    }
}
```

여기서 한 일:
* 우선 hasActiveSession 프로퍼티를 읽을수 있게 하는 MockedTracker 인스턴츠를 생성하였다.
* SessionMonitor 인스턴스는 디펜던시로 MockedTracker를 주입하여 생성하였다.\
테스트가 끝나기 전까지 relaease 되지 않도록 클래스 변수에 인스턴스를 저장하였다.
* validating이 끝나면 초기 hasActiveSession는 false로 설정된다.\
기대하는 노티피케이션을 날릴 것이고 정확한 결과가 나오는지 확인할 것이다.

이제 퍼블릭으로 설정하지 않고도 프라이빗 속성인 SessionMoniter를 테스트할 수 있다.\
이는 큰 발전이지만 우리의 모든 요구조건을 충족시키진 못한다.\
여전히 Tracker 인스턴스를 테스트해야 하고 이 인스턴스가 정확하게 동작하는지 확인 필요하다.

## Exposing internal variables and methods as a final resort

이 Tracker 인스턴스의 경우 private hasActiveSession 프로퍼디는 여전히 읽을 수 없는 상태이다.\
앞서 선언해 두었던 MockedTracker는 SessionMonitor를 테스트하는데 사용되지만 실제 Tracker 클래스를 테스트할수는 없다.\
\
이것을 가능하게 하기위해 import한 타겟에 @testable 속성을 부여함으로 가능하게 할 수 있다.

```swift
@testable import MyApplication
```

@testable 속성은 모듈을 컴파일할때 테스트가 가능하도록하는 하는 역할만 한다.\
지금의 경우 테스트를 실행하면 테스트 클래스에서 사용이 가능하다는 것을 의미한다.\
이 속성은 아래의 몇가지를 발생시킨다.
* 해당 스코프에 import한 모듈의 접근레벨을 상승시킨다.
* 이것은 클래스와 클래스 맴버가 internal 혹은 public 처럼 오픈된 형태로 만든다.
* 구조체같은 다른 요소들이 퍼블릭으로 선언되었을때 처럼 동작한다.

이것을 좀 더 자세히 설명하게 위해 앞서 선언했던 클래스로 돌아가 보자.

```swift
final class Tracker: SessionTracking {
    private var hasActiveSession: Bool = false

    func startNewSession() {
        hasActiveSession = true
    }

    func stopCurrentSession() {
        hasActiveSession = false
    }
}
```

@testable을 사용하지 않으면 테스트에 접근도 불가하고 'Use of unresolved identifier ‘Tracker'라는 에러가 발생한다.\
@testable 속성을 부여한 후 비로서 Tracker 인스턴스에 접근이 가능하다. 하지만 여전히 hasActiveSession에는 접근이 불가능 하다.\
이제 오직 한가지 방법밖에 없고 그것은 non private으로 만드는 것이다.\
결국 그것 외에는 방법이 없다.

```swift
final class Tracker: SessionTracking {
    private(set) var hasActiveSession: Bool = false

    func startNewSession() {
        hasActiveSession = true
    }

    func stopCurrentSession() {
        hasActiveSession = false
    }
}
```

클래스 내부에서만 쓰기가 가능하도록 프로퍼티에 private(set)를 추가하였다.\
이것은 클래스 외부에서 내부요소의 읽기 접근이 가능하도록 해서 테스트 코드를 쓰기에 충분하다.

## Conclusion

마침내 모든 유즈 케이스가 테스트가 가능하게 되었다\
이것으로 100% 코드 커버리지가 가능하게 되었지만 이것이 주 목표가 되어서는 안된다.\
가능한한 많은 유즈케이스에 좋은 테스트 코드를 작성하도록 노력해보자.\
의존성 주입이 목표를 이루는데 도움이 되지 않는다면 프로퍼티에 읽기 접근 권한을 부여할 수도 있다.