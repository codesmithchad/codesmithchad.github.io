---
layout: post
title: Banner Scroll View
subtitle: 바쁘다 바빠 현대 코딩, 또 이런거 만드느라 시간쓰지 말자
# cover-img: /assets/img/toastMessageView.png
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [iOS, UI, Templete, scroll, banner, 벤허]
---

> 피그마에 으름드은 스크롤 배너가 그려져있다.\
맘에 안든다. 눈에는 띄지만 잘 보지도 않고 원하는 화면은 항상 한참 넘겨야 나온다.\
보고 있는데 넘어가면 단전에서 드래곤브래스가 올라 온다.\
하지만 그럴싸한 대안이 없다. 커맨드 쉬프트 N을 누를수 밖에..

&nbsp;
# 기능
* 컨텐츠 영역을 뷰로 생성하여 던지도록 설계하여 활용도를 높인다.
* UIScrollView와 UIStackView를 활용하여 크기 계산해서 좌표잡던 과거의 방식은 반복하지 않고 싶다.

&nbsp;

---

&nbsp;
# 구현

## 1. 파일 생성
* BannerScrollView.swift
    * 스크롤 배너 모듈을 구현할 파일
    * UIScrollView를 상속받고 UIScrollViewDelegate를 conform 한다
* SampleBannerView.swift
    * 스크롤 배너의 각 화면이 될 뷰 파일
* BannerScrollExampleViewController.swift
    * 스크롤 배너 모듈을 실행할 파일

&nbsp;

## 2. 배너 프로토콜
일관되지 않은 배너들을 담았다가 고생하고 싶지 않으므로 규격을 지정한다.
```swift
protocol BannerViewable: UIView {
    var tapClosure: (() -> Void)? { get }   // 1
}
```
1. 배너 선택시의 기능을 클로저로 구성하도록 한다

&nbsp;

## 3. BannerScrollView의 property들
**BannerScrollView**
```swift
private let stackView = UIStackView() // 1
private var banners: [BannerViewable] // 2
private var bannerSize: CGSize = .zero // 3
```
1. 각 배너 view들을 담게 될 stack view
2. BannerViewable 프로토콜을 따르는 배너 view들
3. 각 배너의 사이즈를 지정하지 않으면 stack view 안에서 자리를 잡지 못하므로 추가하였다.

&nbsp;

## 4. init시 배너뷰들과 배너뷰들의 사이즈를 할당한다
**BannerScrollView**
```swift
init(frame: CGRect = .zero, banners: [BannerViewable], bannerSize: CGSize) {
    self.banners = banners
    self.bannerSize = bannerSize
    super.init(frame: frame)
    setupUI()
}
```

&nbsp;

## 5. setup UI
**BannerScrollView**
```swift
private func setupUI() {
    stackView.distribution = .fill // 1
    showsHorizontalScrollIndicator = false
    isPagingEnabled = true
    // delegate = self // 2
    addSubview(stackView) // 3
    stackView.translatesAutoresizingMaskIntoConstraints = false
    NSLayoutConstraint.activate([
        stackView.topAnchor.constraint(equalTo: topAnchor),
        stackView.leftAnchor.constraint(equalTo: leftAnchor),
        stackView.bottomAnchor.constraint(equalTo: bottomAnchor),
        stackView.rightAnchor.constraint(equalTo: rightAnchor)
    ])

    for banner in banners { // 4
        banner.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            banner.widthAnchor.constraint(equalToConstant: bannerSize.width),
            banner.heightAnchor.constraint(equalToConstant: bannerSize.height)
        ])
        stackView.addArrangedSubview(banner)
    }
}
```
1. stackView와 scrollView(self다.)의 ui 속성을 지정한다.
2. 페이지넘버등을 활용해야 하는 경우 필요하다.
3. scrollView에 stackView를 얹고 constraints를 지정한다.
4. banner의 size를 constrainting 하고 stackView에 add 한다.

&nbsp;

## 6. BannerViewable 을 따르는 SampleBannerView 생성
각 배너의 화면이 될 UIView
**SampleBannerView**
```swift
var tapClosure: (() -> Void)? // 1
var titleLabel = UILabel() // 2

init(frame: CGRect = .zero, tapClosure: (() -> Void)? = nil) { // 3
    self.tapClosure = tapClosure
    super.init(frame: frame)
    setupUI()
}
```
1. 탭 선택시 실행할 클로저
2. 각 배너에 표시할 내용
3. init시 클로저를 클래스 전역변수로 할당한다

&nbsp;

## 7. UI setting과 action setting
**SampleBannerView**
```swift
private func setupUI() {
    let tapGesture = UITapGestureRecognizer(target: self, action: #selector(tapAction)) // 1
    addGestureRecognizer(tapGesture)
    
    titleLabel.font = .systemFont(ofSize: 40, weight: .heavy) // 2
    titleLabel.textAlignment = .center
    titleLabel.translatesAutoresizingMaskIntoConstraints = false
    addSubview(titleLabel)
    NSLayoutConstraint.activate([
        titleLabel.topAnchor.constraint(equalTo: topAnchor),
        titleLabel.leftAnchor.constraint(equalTo: leftAnchor),
        titleLabel.bottomAnchor.constraint(equalTo: bottomAnchor),
        titleLabel.rightAnchor.constraint(equalTo: rightAnchor)
    ])
}

@objc private func tapAction() {
    tapClosure?() // 3
}
```
1. 레이블 탭시 클로저(3번)가 실행되도록 제스쳐를 등록한다
2. 레이블의 ui를 구성한다.

&nbsp;

## 8. 샘플 배너 생성
**BannerScrollExampleViewController**
```swift
private var sampleBanners: [BannerViewable] {
    let banners: [BannerViewable] = (0...5).enumerated().map({ idx, element in
        let tapClosure: () -> Void = {
            print("\(idx+1)번 샘플 배너를 눌렀다")
        }
        let bannerView = SampleBannerView(tapClosure: tapClosure)
        bannerView.titleLabel.text = String(idx)
        bannerView.backgroundColor = [.red, .green, .blue][idx%3]
        return bannerView
    })
    return banners
}
```
* map을 돌려 대충 샘플 뷰를 6개 생성하였다
* 각 배너는 생성시 클로저를 넘기도록 한다.

&nbsp;

## 9. BannerScrollExampleViewController의 ui 요소들을 설정
**BannerScrollExampleViewController**
```swift
private func setupUI() {
    view.backgroundColor = .systemBackground
    title = "Banner Scroll"
    
    let bannerHeight: CGFloat = 120
    let bannerScrollView = BannerScrollView(banners: sampleBanners, bannerSize: CGSize(width: view.bounds.width, height: bannerHeight))
    view.addSubview(bannerScrollView)
    bannerScrollView.translatesAutoresizingMaskIntoConstraints = false
    NSLayoutConstraint.activate([
        bannerScrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 100),
        bannerScrollView.leftAnchor.constraint(equalTo: view.leftAnchor),
        bannerScrollView.rightAnchor.constraint(equalTo: view.rightAnchor),
        bannerScrollView.heightAnchor.constraint(equalToConstant: bannerHeight)
    ])
}
```
* 8번의 배너들을 인자로 bannerScrollView를 생성하고 ui를 지정해 준다.

&nbsp;

---

&nbsp;

# 추가 기능
커런트 페이지 넘버가 결국 필요하게 되겠지..

**BannerScrollView**
```swift
func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {
    let currentPage = Int(scrollView.contentOffset.x / bounds.width)
    print("currentPage :: \(currentPage)")
}
```
* 위에서 주석을 걸어두었던 BannerScrollView의 `delegate = self`를 주석해제하고 scroll view delegate 펑션을 추가한다.

&nbsp;

---

&nbsp;

# 남은 이슈
* 타이머로 일정 시간마다 자동으로 넘어가게 하는 옵션이 필요하다.
* 사용자가 배너를 건드렸다면 일정 시간 혹은 영원히 스크롤을 멈추고 싶다.
* 무한 스크롤 옵션이 필요할 수도 있다.