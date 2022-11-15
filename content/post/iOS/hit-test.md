---
title: "Hit Test"
date: 2022-11-15T21:19:30+09:00
draft: false
summary: "Touch event가 발생했을 때 이를 처리할 view를 결정하는 방법"
tags: ['iOS']
---

# Basic Concept

**Hit Testing** : 화면에 사용자 touch event가 발생했을 때, 그 event를 받아서 처리할 view를 결정하는 것. 

Hit testing을 통해 결정된 view는 event를 받아서 분석하고 적절하게 처리할 수 있다.

Touch event가 발생하면 window 아래 view hierarchy에 연결된 `UIView`들의 [`hitTest(_:with:)`](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest) method가 호출된다.  

`hitTest(_:with:)` method가 반환하는 view가 실제로 event를 받아서 처리한다.
- Touch event handling (e.g. `touchesBegan(_:with:)`)
- Gesture handling (e.g. `UITapGestureRecognizer`)

기본적으로 event가 발생한 위치의 맨 앞에 있는 view(z-order가 가장 큰 view)가 반환되지만, `UIView` subclass에서 `hitTest(_:with:)`를 override하여 event를 전달받을 view를 임의로 지정해 주는 것도 가능하다.

```swift
// Event가 발생하면 항상 특정 view로만 전달
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    return someView
}

// 특정 view에는 항상 event를 전달하지 않음. 이 경우, hierarchy의 다음 view를 탐색한다.
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    return nil
}
```

- `hitTest(_:with:)`에서 반환되는 view가 event를 받아 처리한다.
- `nil`을 반환하면 hierarchy의 다음 view에 대해 hit testing을 수행한다.(다음 view의 `hitTest(_:with:)` method를 호출한다.)

# 원리

View hierarchy에 있는 view들 중 **맨 앞에 있는 view부터** event가 발생한 point를 포함하는지 검사한다. 즉, 다음과 같은 조건의 view를 찾을 때 까지 hierarchy를 순회한다.

1. View 계층의 leaf node에 있고 (== z-order가 가장 크고 sibling들 중 가장 마지막에 추가된 view)
2. Frame 영역 안에 touch point를 포함하고 있는가

계층을 순회할 때는 '**역방향 깊이 우선 탐색(Reverse Pre-Order Depth-First Traversal)**' 알고리즘을 사용한다. 이 알고리즘은 view hierarchy의 leaf node를 먼저 찾고, 이전 sibling view 또는 상위 z-order view 방향으로 hierarchy를 순회하며 조건에 맞는 view를 찾는다. 즉, **Hierarchy의 끝에서부터 거꾸로 탐색해 나간다.**

1. Root view가 event 발생 위치(point)를 포함하는지?
2. 포함한다면 subview들 중 가장 큰 index의 view에 대해 1번을 검사한다. 
    - Subview index가 가장 큰 view == slibling view들 중 가장 마지막에 추가된 view
3. 2번에서 view가 point를 포함하지 않으면 다음 index의 view에 대해 1번을 반복한다.
4. 2번에서 view가 point를 포함하면 해당 view의 subview들에 대해 1~3를 반복한다.
5. **4에서 더 이상 subview가 없다면, 해당 view를 반환한다. (해당 view가 event를 처리한다.)**

아래 그림은 touch event가 발생했을 떄 hit testing에 의해 event를 전달할 view가 결정되는 과정을 보여준다.

{{< figure src="/images/iOS/hit-test-1.png" >}}

1. `UIWindow`(root view)는 화면 전체를 그리기 위한 view로, touch point를 포함하므로 subview인 'MainView'를 탐색한다.
2. 'MainView'는 `UIViewController`(root view controller)의 view가 되는데, 이 view 또한 touch point를 포함하므로 subview들 중 index가 가장 큰 'View C'를 탐색한다.
    - Root view controller의 view는 `UIWindow`의 크기에 맞게 화면에 나타나므로, 항상 touch point를 포함한다.
3. 'View C'는 touch point를 포함하지 않으므로 다음으로 index가 큰 'View B'를 탐색한다.
4. 'View B'는 touch point를 포함하므로, 그 subview들 중 index가 가장 큰 'View B.2'를 탐색한다.
5. 'View B.2'는 touch point를 포함하지 않으므로 다음으로 index가 큰 'View B.1'을 탐색한다.
6. 'View B.1'은 touch point를 포함하고 subview가 없으므로, root로부터 가장 멀리 떨어진 view(leaf node)가 되어 touch event를 전달받을 view로 결정된다.

## Hit Testing 구현

Hit testing에서 event를 전달받을 후보 view가 되기 위해서는 다음 조건을 만족해야 한다.

1. View가 화면에 보여야 한다.
2. User interaction이 가능해야 한다.
3. View 영역이 event 발생 위치(point)를 포함해야 한다.

View 계층에서 세 가지 조건을 만족하는 view를 찾기 위해 `hitTest(_:with:)`는 다음과 같이 구현될 수 있다.

```swift
func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    // 1번 조건 검사
    guard !isHidden, alpha > 0.01 else { 
        return nil 
    }
    
    // 2번 조건 검사
    guard isUserInteractionEnabled else { 
        return nil 
    }
    
    // 역방향으로 탐색하기 때문에, subviews를 뒤집어서(reversed) 탐색
    for subview in subviews.reversed() {
        let point = subview.convert(point, from: self)
        guard let hitView = subview.hitTest(point, with: event) else { 
            continue
        }
        
        // 재귀적으로 반환되는 hitView는 곧 subview를 의미한다.
        // 즉, subview에서 hit testing이 성공하면 subview가 자기 자신을 반환할 것이다.
        return hitView
    }
    
    // 3번 조건 검사
    guard bounds.contains(point) else { 
        return nil 
    }
    
    return self
}
```

# Hit Testing 활용

## 위를 덮고 있는 view를 통과해서 event 전달하기

아래와 같이 5개의 `UISwitch`가 파란색 view로 덮여 있다. 스위치를 터치하더라도, 덮고 있는 view가 event를 가져가므로 스위치를 on/off할 수 없다.

{{< figure src="/images/iOS/hit-test-2.png" width="50%" >}}

이 때, 스위치를 터치해서 on/off할 수 있게 하려면 cover view에 event가 전달되지 않아야 한다. 따라서, event를 받기 위한 3가지 조건 중 한 가지를 만족하지 않도록 바꿔주면 event를 받지 못하고 아래에 있는 `UISwitch`가 event를 받을 것이다.

1. User interaction 비활성화
	```swift
    coverView.isUserInteractionEnabled = false
    ```
2. View 숨김(단, Cover view가 반드시 보여야한다면, 이 방법은 사용하지 못할 것이다.)
	```swift
    coverView.isHidden = true
    // or
    coverView.alpha = 0
    ```
3. `hitTest(_:with:)` 함수에서 `nil` 반환
	```swift
    class CoverView: UIView {
    	override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        	return nil
        }
	}
    ```

1번 방법은 cover view 전체가 event를 받지 못하게 하기 때문에 5개 switch를 모두 조작할 수 있게 된다. 만약 특정 스위치만 on/off할 수 있게 만들려면 `hitTest(_:with:)` method를 override해서 `nil`을 반환하는 조건을 추가로 구현해야 한다.

아래 코드는 가운데 있는 switch만 on/off가 가능하도록 만들어 준다.

```swift
class CoverView: UIView {
    
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        var hitView = super.hitTest(point, with: event)
        
        // 가운데 스위치가 있는 영역
        let rect = bounds.insetBy(dx: bounds.width / 3, dy: bounds.height / 3)

        // Point가 가운데 영역(rect)에 포함될 때는 nil을 반환한다.
        guard rect.contains(point) else { 
            return hitView 
        }

        return nil
    }
}
```

## 터치 영역 확장하기

아래와 같이 버튼의 크기가 너무 작아서 터치할 수 있는 영역을 확장해야 하는 경우가 있다. 단, 버튼의 크기는 유지해야 하므로 버튼의 크기를 키워서 터치 영역을 넓힐 수는 없다.

{{< figure src="/images/iOS/hit-test-3.png" >}}

버튼 바깥의 영역을 터치하는 것은 3번 조건을 만족하지 않는 상황(event 위치가 view에 포함되지 않음)에 해당한다. 아래 코드는 event를 검사할 때 touch point가 포함되는 영역을 확장해서 event를 받도록 한다.

```swift
class CustomButton: UIButton {
    
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        // Button 영역에서 상하좌우 10pt만큼 더 넓은 영역까지 touch point를 검사한다.
        let contains = bounds.insetBy(dx: -10, dy: -10).contains(point)
        return contains ? super.hitTest(point, with: event) : nil
    }
}
```

# Reference

- http://smnh.me/hit-testing-in-ios/
- https://developer.apple.com/documentation/uikit/uiview/1622469-hittest