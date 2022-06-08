---
layout: post
title: RxFlow tutorial for begginers 1
subtitle: RxFlow를 천천히 알아보자.
# cover-img: /assets/img/toastMessageView.png
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [iOS, pattern, coordinator]
---

ref: [RxFlow](https://github.com/RxSwiftCommunity/RxFlow)\
Official 튜토리얼이 너무 어려워서 한 스텝씩 진행할 수 있는 가이드를 만들기로 한 나\
간단한 Step과 Flow만 생성하여 첫 화면까지만 만들어본다.\
[project source link](https://github.com/codesmithchad/gettingRxFlow/tree/first_flow)

&nbsp;

## 1 Add Steps
첫 단계로 먼저 스텝을 정의한다.\
지금은 첫 화면 표시만를 목적으로 하므로 첫번째 스탭만 추가한다.

```swift
// AppStep.swift

enum AppStep: Step {
    case appLaunched
}
```

&nbsp;

## 2 Add First Flow and Stepper
이 튜토리얼 앱의 첫번째 플로우이며 navigationController를 root presentable로 갖는다.\
(Flow 타입은 `var root: Presentable`와 `func navigate(to step: Step) -> FlowContributors`가 required이다.)\
SceneDelegate에서 첫번째 스테퍼로 사용될 AppStepper도 정의한다.

```swift
// AppFlow.swift

final class AppFlow: Flow {
    
    var root: Presentable {
        rootViewController
    }
    
    private lazy var rootViewController: UINavigationController = { // 1
        let viewController = UINavigationController()
        return viewController
    }()
    
    func navigate(to step: Step) -> FlowContributors {
        guard let step = step as? AppStep else { return .none }
        switch step {
        case .appLaunched: // 2
            return doesAppLaunched()
        }
    }
    
    private func doesAppLaunched() -> FlowContributors { // 3
        let introFlow = IntroFlow()
        Flows.use(introFlow, when: .created) { [weak self] flowRoot in
            self?.rootViewController.pushViewController(flowRoot, animated: false)
        }
        return .one(flowContributor: .contribute(withNextPresentable: introFlow, withNextStepper: OneStepper(withSingleStep: AppStep.appLaunched))) // warning: 마지막 AppStep.appLaunched은 차후 진행할 스텝으로 변경되어야 한다.
    }
}

final class AppStepper: Stepper {
    
    let steps = PublishRelay<Step>()
    
    var initialStep: Step { // 4
        AppStep.appLaunched
    }
}

```
1. navigationController 기반으로 화면을 다루고 싶으므로 첫번째 플로우의 presentable은 navigationController로 구성한다.
2. .appLaunched인 경우 doesAppLaunched()를 호출하여 FlowContributor 리턴하도록 한다. 리턴된 FlowContributor를 통해 해당 플로우로 navigate된다.
3. doesAppLaunched()에서는 두번째 플로우인 IntroFlow를 진행하도록 FlowContributor를 정의한다.
4. 첫 진입의 경우 트리거가 없으므로 initialStep으로 .appLaunched step을 진행하도록 지정한다. (SceneDelegate에서 coordinate를 정의할때 AppStepper를 인자로 넘겨 이 스텝으로 navigate 될 것이다.)

&nbsp;

## 3 Add Second Flow
두번째로 실행 될 플로우로 navigatoin controller의 첫 view controller로 들어가게 될 IntroViewController를 root presentable로 갖는다.
```swift
// IntroFlow.swift

final class IntroFlow: Flow {
    var root: Presentable {
        rootViewController
    }
    
    private let rootViewController = IntroViewController() // 1
    
    func navigate(to step: Step) -> FlowContributors { // 2
        return .none
    }
}
```
1. IntroViewController를 presentable로 가질 것이므로 rootViewController로 지정한다.
2. 아직 다음단계가 없는 상태로 navigate()는 .none만 리턴하도록 한다.

&nbsp;

## 4 Add Presentable
첫 화면으로 쓰일 뷰 컨트롤러를 추가한다.\
식별이 가능하도록 타이틀과 버튼을 추가하였다.

```swift
// IntroViewController.swift

final class IntroViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Main"
        view.backgroundColor = .systemBackground
        setupUI()
    }
    
    private func setupUI() {
        let button = UIButton(configuration: .borderedTinted(), primaryAction: buttonAction)
        button.setTitle("go Next!", for: .normal)
        button.setTitleColor(.gray, for: .normal)
        view.addSubview(button)
        button.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
    
    private var buttonAction: UIAction {
        UIAction(handler: { _ in
            print("dododoo")
        })
    }
}
```

&nbsp;

## 5 Use First Flow
SceneDelegate에 위에 정의한 플로우들이 적용되도록 구성한다.

```swift
// SceneDelegate.swift

final class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?
    private let coordinator = FlowCoordinator() // 1

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }
        window = UIWindow(windowScene: windowScene)
        
        let flow = AppFlow()
        coordinator.coordinate(flow: flow, with: AppStepper()) // 2
        Flows.use(flow, when: .created) { [weak self] flowRoot in //3
            self?.window?.backgroundColor = .systemBackground
            self?.window?.rootViewController = flowRoot
            self?.window?.makeKeyAndVisible()
        }
    }
}
```
1. FlowCoordinator를 생성한다.
2. AppFlow와 AppStepper를 인자로 coordinate를 조합한다.
3. 생성한 `AppFlow`와 `AppStepper`를 인자로 `Flows.use`를 호출하여 지정한 step에 맞는 flow를 실행하도록 한다./
또한 완료 클로저에서 flowRoot(presentable)를 전달받아 window의 rootViewController로 지정한다.


&nbsp;

[to be continued...](https://codesmithchad.github.io/2022-06-08-RxFlow_Tutorial2/)


&nbsp;
