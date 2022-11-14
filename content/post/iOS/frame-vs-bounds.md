---
title: "Frame과 Bounds 차이"
date: 2022-11-13T23:00:00+09:00
draft: false
summary: "Frame과 Bounds 속성의 차이"
tags: ['iOS']
---

# Overview

- 공통점 : 원점(origin)과 크기(size)로 사각형 영역을 표현함
- 차이점 : **어떤 view의 좌표계**에서 원점과 크기를 표현하는가?
    - Frame : **Superview 좌표계를 기준**으로 자기 자신의 위치 및 크기를 나타냄
    - Bounds : **자기 자신의 좌표계를 기준**으로 자기 자신의 위치 및 크기를 나타냄

# 원점(Origin) 비교

- Frame : Superview 좌표계의 원점(origin)을 기준으로 현재 view의 원점이 얼마나 떨어져 있는지를 나타냄
- Bounds : 자기 자신 좌표계의 원점(origin)을 기준으로 현재 view의 원점이 얼마나 떨어져 있는지를 나타냄
    - View를 초기화한 직후에는 `(0, 0)`이 될 수 밖에 없다.

{{< figure src="/images/iOS/frame-vs-bounds-1.png" width="80%" >}}

# 크기(Size) 비교

- Frame : Superview 좌표계에서 현재 view가 차지하는 사각형 영역의 크기를 나타냄
- Bounds : 자기 자신 좌표계에서 현재 view가 차지하는 사각형 영역의 크기를 나타냄
- View를 회전시켰을 때 차이점을 명확하게 알 수 있다.
    - Frame : 회전된 상태의 view를 모두 포함하는 영역이 size로 지정됨
    - Bounds : 자기 자신의 좌표계가 회전된 것과 같으므로, 동일한 size로 지정됨

{{< figure src="/images/iOS/frame-vs-bounds-2.png" width="80%" >}}

# 원점을 이동시킬 때 차이

## Frame의 원점 이동

- Frame의 원점을 `(x, y)`만큼 이동시키면 **superview 내에서 위치를 이동**하는 것과 같으므로, view도 `(x, y)`만큼 이동한 위치에 그려진다.
- Frame이 이동할 때, 이동하는 view의 subview들도 같은 만큼 이동한다.

{{< figure src="/images/iOS/frame-vs-bounds-3.png" width="80%" >}}

## Bounds의 원점 이동

- Bounds의 원점을 `(x, y)`만큼 이동시키면 **자기 자신의 좌표계 자체를 이동**하는 것과 같다.
- 좌표계를 이동했지만, 자기 자신은 좌표계를 기준으로 표시되므로 위치가 변하지 않는다.
- **대신, 자기 자신의 좌표계를 기준으로 위치하는 subview들이 `(-x, -y)`만큼 이동해서 그려진다.**

{{< figure src="/images/iOS/frame-vs-bounds-4.png" width="80%" >}}

## Bounds의 원점을 이동하는 것과 `UIScrollView`의 동작 원리

`UIScrollView`는 `bounds`의 원점(origin)을 이동시키는 방법을 사용하는 대표적인 view이다. 

`UIScrollView`는 오른쪽으로 스크롤하면 content도 오른쪽으로 이동한다. 즉, view를 오른쪽으로 스크롤하면 content의 x 좌표는 증가한다. View는 `bounds`의 원점이 이동하는 **반대 방향으로** 이동하므로, content를 오른쪽으로 이동시키려면 `bounds`의 x좌표를 이동한 좌표 `dx`만큼 빼줘야 한다. 그러면 `bounds`의 원점 x값이 점점 작아지면서 해당 view가 점점 **오른쪽으로 이동하는 것 처럼 보일 것이다**.

실제로 scroll view의 content를 스크롤 해 보면, `bounds` 속성값이 스크롤하는 방향과 반대로 업데이트된다.

`UIScrollView`의 동작을 실제로 구현해 보면 다음과 같다.

> Content view의 **superview**의 `bounds.origin` 값을 업데이트하는 것에 유의한다. 만약 image view의 `bounds` 속성을 변경한다면, image view의 subview가 없기 때문에 아무 일도 일어나지 않는다.

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    let imageView = UIImageView(image: UIImage(named: "image"))
    imageView.frame = CGRect(x: 0, y: 0, width: 1200, height: 1200)
    view.addSubview(imageView)
    view.addGestureRecognizer(
        UIPanGestureRecognizer(
            target: self, 
            action: #selector(panGestureHandler(_:))
        )
    )
}

@objc
func panGestureHandler(_ recognizer: UIPanGestureRecognizer) {
    let translation = recognizer.translation(in: view)
    view.bounds.origin.x -= translation.x
    view.bounds.origin.y -= translation.y
    recognizer.setTranslation(.zero, in: view)
}
```

구현 결과 : 
{{< figure src="/images/iOS/frame-vs-bounds-6.gif" width="50%" >}}

# Reference 

- [Frame](https://developer.apple.com/documentation/uikit/uiview/1622621-frame)
- [Bounds](https://developer.apple.com/documentation/uikit/uiview/1622580-bounds)