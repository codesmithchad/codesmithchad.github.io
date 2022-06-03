---
layout: post
title: RxFlow
subtitle: RxFlow의 리드미를 읽어보자
# cover-img: /assets/img/toastMessageView.png
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [iOS, coordinator, pattern]
---

RxFlow를 이해하는데 반드시 필요한 6가지 용어
* Flow
    * 플로우는 네비게이션 영역을 규정한다. 이곳이 네비게이션 액션을 정의하는 곳이다.
* Step
    * 스탭은 내비게이션이 향할 방향을 표현하는 방법이다. 플로들과 조합하여 가능한 모든 네비게이션 액션을 서술한다. 스탭은 id들이나 url들 같은 내부값을 내포하여 다른 플로우들에 전달할 수 있다.
* Stepper
    스태퍼는 플로우들 안의 스탭들을 방출할 수 있는 무엇이든 될 수 있다 
    a Stepper can be anything that can emit Steps inside Flows.
* Presentable
    it is an abstraction of something that can be presented (basically UIViewController and Flow are Presentable).
    화면을 표시할 수 있는것들의 추상이다.
* FlowContributor
    FlowCoordinator 플로우의 다른 스탭에서 무엇을 할 것인지를 알려주는 간단한 데이터 구조체다. 
    it is a simple data structure that tells the FlowCoordinator what will be the next things that can emit new Steps in a Flow.
* FlowCoordinator
    플로우와 스탭에서 네이게이션의 행동을 정의하였다면 FlowCoordinator는 이것들을 섞어 앱안의 모든 네비게이션을 컨트롤하는 일을 한다.
    FlowCoordinators는 FxFlow로 부터 제공되고 직접 구현할 필요는 없다.
    once the developer has defined the suitable combinations of Flows and Steps representing the navigation possibilities, the job of the FlowCoordinator is to mix these combinations to handle all the navigation of your app. FlowCoordinators are provided by RxFlow, you don't have to implement them.
