---
layout: post
title: RxFlow totorial for begginers 1
subtitle: RxFlow를 천천히 알아보자.
# cover-img: /assets/img/toastMessageView.png
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [iOS, pattern, coordinator]
---

Official 튜토리얼이 너무 어려워서 한 스텝씩 진행할 수 있는 가이드를 만들기로 한 나
간단한 Step과 Flow만 생성하여 첫 화면까지만 만들어본다.
[project source link](https://github.com/codesmithchad/gettingRxFlow/tree/first_flow)

## 1 Add Steps
첫 단계로 먼저 스텝을 정의한다.\
최초 구동시 스탭과 임의 스텝 두개만 추가하도록 하겠다.

```swift
// AppStep.swift

enum AppStep: Step {
    case appLaunched
    case testStep
}
```

&nbsp;

## 2 Add First Flow and Stepper
이 앱의 첫번째 플로우를 추가한다.\
RxFlow의 Flow를 상속하도록 하고 root 변수와 navigate() 함수를 추가한다.


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
        default:
            return .none
        }
    }
    
    private func doesAppLaunched() -> FlowContributors { 
        let introFlow = IntroFlow()
        Flows.use(introFlow, when: .created) { [weak self] flowRoot in
            self?.rootViewController.pushViewController(flowRoot, animated: false)
        }
        return .one(flowContributor: .contribute(withNextPresentable: introFlow, withNextStepper: OneStepper(withSingleStep: AppStep.appLaunched)))
    }
}

final class AppStepper: Stepper {
    
    let steps = PublishRelay<Step>()
    
    var initialStep: Step { // 3
        AppStep.appLaunched
    }
}

```
1. 앱 실행 후 첫번째 스탭으로 UINavigationController을 presentable로 갖는 스텝을 생성한다. (네비게이션 컨트롤러를 사용할 것이므로 rootViewController는 UINavigationController)
2. .appLaunched인 경우 doesAppLaunched()를 호출하여 FlowContributor 리턴하도록 한다.
3. 첫 진입의 경우 트리거가 없으므로 initialStep으로 .appLaunched step을 진행하도록 지정한다.

&nbsp;

## 3 Add Second Flow
앱 실행 후 두번째 스탭으로 첫 화면인 IntroViewController를 presentable로 갖는 스탭을 생성한다.
```swift
// IntroFlow.swift

final class IntroFlow: Flow {
    var root: Presentable {
        rootViewController
    }
    
    private let rootViewController = IntroViewController() // 1
    
    func navigate(to step: Step) -> FlowContributors { // 2
        guard let step = step as? AppStep else { return .none }
        switch step {
        case .testStep:
            return .none
        default:
            return .none
        }
    }
}
```
1. IntroViewController를 presentable로 가질 것이므로 rootViewController로 지정한다.
2. 아직 다음단계가 없는 상태로 navigate()는 .none 만 리턴하도록 한다.

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

## 5 Set Root Step
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
2. AppFlow와 AppStepper를 생성하고 coordinate를 조합한다.
3. AppFlow가 적용되도록 플로우를 실행하고 클로저로 presentable인 flowRoot를 전달받아 window의 rootViewController를 지정한다.