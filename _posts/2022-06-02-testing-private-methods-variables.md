---
layout: post
title: [draft] Testing private methods and variables in Swift from SwiftLee
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

프라이빗 코드가 정상 동작하는지 어떻게 확인하지? 라고 생각할 것이다.