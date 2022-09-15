---
layout: post
title: RxFlow
subtitle: RxFlow의 리드미를 읽어보자
# cover-img: /assets/img/toastMessageView.png
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [iOS, coordinator, pattern]
---

## RxFlow aims to
* 스토리보드를 유닛 단위로 분리하여 뷰컨트롤러의 재사용성과 협업에 용이하도록 한다.
* 네비게이션 컨텍스트에 따라 뷰컨틀롤러를 다른 방식으로 표시할수 있게 한다.
* 의존성 주입이 용이하도록 한다.
* 뷰컨트로러로 부터 모든 네비게이션 메케니즘을 제거한다.
* rx 프로그래밍이 용이하도록 한다.
* 네비게이션을 선언형 
* 케이스별 네비게이션 이동을 선언형 방식으로 표현한다.
* 애플리케이션을 네비게이션의 로직 블럭으로 다루어 분리가 용이하도록 한다.

<!--
RxFlow의 장점
1. 스토리보드를 유닛 단위로 쪼개서 UIViewController의 재사용성을 키웁니다
2. 네비게이션의 흐름(context)에 맞게 UIViewController를 다른 방식으로 보여줄 수 있습니다
3. 의존성 주입(Dependency Injection)을 쉽게 구현할 수 있습니다
4. UIViewController에 있는 모든 네비게이션 매커니즘을 삭제합니다.
5. 반응형 프로그래밍(Reactive Programming) 사용을 촉진합니다
6. 네비게이션에서 일어나는 대부분의 케이스를 처리하면서 선언형으로 표현할 수 있습니다
7. 어플리케이션을 네비게이션의 논리적인 블록으로 나눌 수 있습니다
-->

&nbsp;

## The key principles
* 코디네이터 패턴의 단점
    * 새 앱을 만들때마다 환경을 위한 코디네이터 매케니즘을 작성하여야 한다.
    * 코디네이터 스택과 커뮤니케이션하기 위해 많은 보일러플레이트 코드가 발생한다.

* 그럼에도 장점
    * 플로우 안에서 더욱 분명하게 네비게이션을 더욱 선언적이게 한다.
    * 플로우간의 이동을 조정할 수 있는 플로우 코디네이터를 기본 제공한다.
    * 플로우 코디네이터의 네비게이션 이동 액션을 발생시키는데 rx 프로그래밍을 사용한다.

* RxFlow를 이해하는데 반드시 필요한 6가지 용어
    * Flow
        * 각 플로우는 `네비게이션 영역을 규정`한다. 
        * 이곳이 **뷰컨트롤러를 보여주거나 다른 플로우를 보여주거나 하는 네비게이션 액션을 정의**하는 곳이다.
    * Step
        * 스탭은 `내비게이션이 향할 방향을 표현`하는 방법이다. 
        * 플로우과 스탭을 조합하여 모든 네비게이션 액션을 서술한다. 
        * 스탭은 플로우에 정의된 스크린으로 **전달할 수 있는 id나 url 같은 내부값도 포함할 수 있다**.
    * Stepper
        * 스태퍼는 플로우들 안의 **스탭들을 방출할 수 있는 무엇**이든 될 수 있다 
    * Presentable
        * 화면에 표시되어질 수 있는 것들의 추상체이다.
    * FlowContributor
        * FlowCoordinator 플로우의 **다른 스탭에서 무엇을 할 것인지**를 알려주는 간단한 `데이터 구조체`다. 
    * FlowCoordinator
        * 플로우와 스탭에서 네비게이션의 행동을 정의하였다면 **FlowCoordinator는 이것들을 섞어 앱안의 모든 네비게이션을 컨트롤**하는 일을 한다.
        * FlowCoordinators는 **RxFlow에서 제공**하며 직접 구현할 필요는 없다.


    <!-- 
    RxFlow의 기본적인 용어
    1. Flow: Flow는 어플리케이션의 네비게이션 공간을 규정합니다. 이 공간에서는 우리가 네비게이션 액션들을 선언할 수 있습니다. (UIViewController 또는 다른 Flow를 띄우는 것처럼요)
    2. Step: Step은 어플리케이션의 네비게이션 상태입니다. Flow와 Step을 조합하면 가능한 모든 네비게이션 액션을 설명할 수 있습니다. Step은 내부 값(Ids, URLs 등이 있습니다)을 임베드할 수 있어서 Flow에 선언된 화면에 전달할 수도 있습니다.
    3. Stepper: Step을 발생시킨다면 무엇이든 될 수 있습니다. Stepper는 Flow의 모든 네비게이션 액션을 트리거해야 합니다.
    4. Presentable: 보일(present) 수 있는 무언가를 추상화한 것입니다. (기본적으로 UIViewController와 Flow가 Presentable입니다) Presentable은 Reactive 옵저버블을 제공하고 Coordinator가 Flow와 Step을 UIKit에서 더 편한 방식으로 조종하기 위해 이를 구독합니다.
    5. Flowable: Presentable과 Stepper를 조합하는 간단한 데이터 구조입니다. Flowable은 우리의 Reactive 매커니즘의 새로운 Step이 생성할 게 무엇인지 Coordinator에게 알려줍니다.
    6. Coordinator: 네비게이션 가능성을 표현하는 Flow와 Step의 적절한 조합을 정의했다면 Coordinator가 해야 할 일은 이 조합을 지속해서 잘 섞는 것입니다. 
    -->

&nbsp;

## How to use RxFlow

스텝은 최종 상태의 네비게이션 의도를 표현하는 조각이며 enum으로 선언하기 매우 편리하다.
```swift
enum DemoStep: Step {
    // Login
    case loginIsRequired
    case userIsLoggedIn

    // Onboarding
    case onboardingIsRequired
    case onboardingIsComplete

    // Home
    case dashboardIsRequired

    // Movies
    case moviesAreRequired
    case movieIsPicked (withId: Int)
    case castIsPicked (withId: Int)

    // Settings
    case settingsAreRequired
    case settingsAreComplete
}
```

 가능한한 독립적인 스텝을 유지하도록 한다. 인스턴스의 경우 showMovieDetail(withIdl: Int) 라는 스탭을 호출하면 강하게 연결되는 나쁜 케이스가 될 수 있다.

