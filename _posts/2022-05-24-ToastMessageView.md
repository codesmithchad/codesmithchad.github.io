---
layout: post
title: Toast message view
subtitle: 바쁘다 바빠 현대 코딩, 이런거 만드느라 시간쓰지 말자
# cover-img: /assets/img/toastMessageView.png
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [iOS, UI, Templete, Toast, notification]
---

> 달갑진 않지만 안드로이드와 유사한 토스트 메세지 UI를 왕왕 요구받는다.
> 이 케이스를 빠르게 쳐내기 위해 템플릿을 만들기로 했다.



# 기능
* 네비게이션 영역에 독립적으로 동작한다. (vc를 팝한다고 같이 사라지지 않는다)
* 모든 화면의 최상위에 존재한다.
* 한번에 한개의 토스트 메세지만 표시한다.



# 구현

![toast](/assets/img/toastMessageView.png){: .mx-auto.d-block :}

## 1. 키윈도 익스텐션
키윈도를 쉽게 얻기 위해 UIApplication extension에 keyWindow를 추가한다.
```
extension UIApplication {
    static var topKeywindow: UIWindow? {
        UIApplication.shared.windows.filter({ $0.isKeyWindow }).first
    }
}
```


## 2. 필수 프로퍼티
```
static let shared = ToastMessageView() // 1
private let messageLabel = UILabel() // 2
private var hideAnchor: NSLayoutConstraint? // 3
private var isShowingToast: Bool { // 4
    hideAnchor?.isActive == false
}
private var hideTimer: Timer? // 5
```
1. 전역으로 사용되고 한개의 인스턴스만 존재하면되므로 싱글턴으로 구현
2. UILabel이 있어야 메세지를 표시할 수 있겠지?
3. 토스트를 가릴때 hideAnchor를 active (deactive되면 showAnchor가 적용)
4. 이미 표시중인 토스트가 있다면 닫기 처리를 선행해야 위한 플래그
5. 일정 시간이 지나면 토스트 숨김처리를 위한 타이머


## 3. UI 셋업
viewDidLoad나 init 시점에 호출하여 default UI를 구성한다.
```
private func setupUI() {
    guard let keyWindow = UIApplication.topKeywindow.first else { return }
    keyWindow.addSubview(self) // 1
    translatesAutoresizingMaskIntoConstraints = false // 2
    messageLabel.translatesAutoresizingMaskIntoConstraints = false
    addSubview(messageLabel)
    NSLayoutConstraint.activate([
        leftAnchor.constraint(equalTo: keyWindow.leftAnchor, constant: 14),
        rightAnchor.constraint(equalTo: keyWindow.rightAnchor, constant: -14),
        heightAnchor.constraint(greaterThanOrEqualToConstant: 40),
        
        messageLabel.topAnchor.constraint(equalTo: topAnchor, constant: 12),
        messageLabel.leftAnchor.constraint(equalTo: leftAnchor, constant: 12),
        messageLabel.bottomAnchor.constraint(equalTo: bottomAnchor, constant: -12),
        messageLabel.rightAnchor.constraint(equalTo: rightAnchor, constant: -12)
    ])
    // # anchors for toggle
    hideAnchor = topAnchor.constraint(equalTo: keyWindow.bottomAnchor) // 3
    hideAnchor?.priority = .defaultHigh
    hideAnchor?.isActive = true
    let showAnchor = bottomAnchor.constraint(equalTo: keyWindow.safeAreaLayoutGuide.bottomAnchor, constant: -14) // 4
    showAnchor.priority = .defaultLow
    showAnchor.isActive = true
}
```
1. 전역으로 표시되므로 키윈도에 얹기로 했다. 
2. 컨스트레인트를 지정한다. 텍스트 양에 따라 높이를 가변하기 위해 greaterThan으로 지정한다. 토스트를 가릴때는 토스트의 탑이 키윈도의 바텀에 붙고 보일때는 토스트의 바텀이 키윈도의 세이프 바텀에 붙는다. 노치가 없는경우 바텀에 마진없이 붙게 되므로 여유공간을 주기위해 constant를 추가하였다.
3. 토스트를 가릴때의 컨스트레인트 지정. priority를 high로 지정하여 우선권을 갖도록 한다.
4. 토스트를 보일때의 컨스트레인트 지정. priority를 low로 지정하여 hideAnchor가 deactive 될 때 showAnchor가 active 되도록 한다.


## 4. 토스트 토글 functions
에니메이션과 함께 토스트를 표시한다.
표시되는 경우 약간의 바운스로 쫄깃함을 추가하고 가리는 경우는 바운스가 없도록 한다. (가리는 경우에도 바운스를 넣으면 찝찝한 UI를 볼 수 있다.)
```
private func showToast(_ message: String?) { // 1
    messageLabel.text = message
    UIApplication.topKeywindow?.layoutIfNeeded()
    
    UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.6, initialSpringVelocity: 2, options: .curveEaseOut) { [weak self] in
        self?.hideAnchor?.isActive = false
        UIApplication.topKeywindow?.layoutIfNeeded()
        self?.addTimer()
    }
}
@objc private func hideToastSelector() { // 2
    hideToast()
}
private func hideToast(_ completion: ((Bool) -> Void)? = nil) { // 3
    UIView.animate(withDuration: 0.25, delay: 0, options: .curveEaseInOut, animations: { [weak self] in
        self?.hideAnchor?.isActive = true
        UIApplication.topKeywindow?.layoutIfNeeded()
        self?.removeTimer()
    }, completion: completion)
}
```
1. 토스트 보이기 펑션. 메세지를 인자로 받아 label에 세팅한 뒤 에니메이션과 함께 hideAnchor를 deactive한다. 일정시간 후에 가려지도록 타이머도 호출한다.
2. 탭, 슬라이드 다운 제스쳐의 selector가 호출할 수 있도록 objc 펑션을 추가한다. (컴플리션 클로저를 인자로 받게 하면 selector 호출시 bad access가 나서 아쉽지만 중복 생성)
3. 토스트 가리기 펑션. 에니메이션과 함께 hideAnchor를 activate한다. 타이머도 종료한다. 가린 직후 다음 토스트를 띄워야 할 수 있으므로 클로저를 받아 컴플리션에 할당한다.


## 5. Timer function 추가
표시할때 addTimer를 호출하여 타이머가 시작되도록 하고 숨길때 remove타이머를 호출하여 타이머가 종료되도록한다. 
```
private func addTimer() {
    hideTimer = Timer.scheduledTimer(timeInterval: 3, target: self, selector: #selector(hideToastSelector), userInfo: nil, repeats: false)
}

private func removeTimer() {
    hideTimer?.invalidate()
    hideTimer = nil
}
```


## 6. Gesture 추가
토스트를 스와이프 다운을 하거나 탭했을 경우 닫기가 동작하도록 한다.
viewDidLoad나 init 시점에 UI 셋업 직후 호출하도록 한다.
```
private func setupGesture() {
    let swipeDown = UISwipeGestureRecognizer(target: self, action: #selector(hideToastSelector))
    swipeDown.direction = .down
    addGestureRecognizer(swipeDown)
    addGestureRecognizer(UITapGestureRecognizer(target: self, action: #selector(hideToastSelector)))
}
```


## 7. 호출
싱글턴으로 구성하였으므로 init없이 어디서나 호출이 가능하다.
```
ToastMessageView.shared.showToastMessage("Hello toast!\nThis is a test toast message.")
```



# 추가 기능
Config 프로퍼티를 통해 속성변경이 가능하도록 한다.


## define
```
struct Config {
    var backgroundColor: UIColor = .black.withAlphaComponent(0.8)
    var maxLines: Int = 10
}
var config: Config? {
    didSet {
        guard let config = self.config else { return }
        backgroundColor = config.backgroundColor
        messageLabel.numberOfLines = config.maxLines
    }
}
```

* config 구조체 인스턴스를 만들고 didSet이 될 때 각 속성값을 할당하도록 한다.

```
if config == nil {
    config = Config()
}
```
* viewDidLoad시점에 config이 nil인 경우 기본값을 설정하도록 한다.


## usage
```
ToastMessageView.shared.config = ToastMessageView.Config(backgroundColor: .blue, maxLines: 0)
ToastMessageView.shared.showToastMessage("Hello toast!\nThis is a test toast message.")
```
* showToastMessage를 호출하기 전에 config을 할당한다.


# 여제
* closure 인자를 가진 objc function은 왜 selector에서 호출하면 bad access가 되는가
* 토스트 메세지를 누르면 닫기 이외의 추가적인 기능 (근데 굳이 토스트가 리치해질 필요가 있나??)
* 여러개의 메세지를 연달아 표시해야 하는 케이스가 발생하는 경우는?
    * 표시속성(immediately, serially)을 지정하여 바로 보여주거나 기다린 후 보여주기?
* config에 배경색이나 라인수 이외에 토스트와 레이블의 constraints, bottom margin, UILabel 속성, cornerRadius, animation 속성 등도 추가 필요