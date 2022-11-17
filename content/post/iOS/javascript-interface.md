---
title: "WebView 통신(a.k.a. Javascript Interface)"
date: 2022-11-17T22:18:45+09:00
draft: false
summary: "iOS 앱과 web view가 통신하는 방법. Swift와 javascript 상호 호출"
tags: ['iOS']
---

# Overview

- Javascript Interface : `WKWebView`에 로드된 javascript 코드와 iOS 앱의 Swift 코드가 상호 통신할 수 있게 해 주는 인터페이스
- WebKit은 javascript에서 native 함수를 호출하거나, native 코드에서 javascript 함수를 호출할 수 있는 방법을 제공함

# Native to JS

Native code에서 JavaScript 함수를 호출하는 방법은 두 가지가 있다.

## 1. User Script 등록

[`WKUserContentController`](https://developer.apple.com/documentation/webkit/wkusercontentcontroller)에 [`WKUserScript`](https://developer.apple.com/documentation/webkit/wkuserscript) 객체를 등록해 두면 특정 시점에 지정된 script를 실행시킨다.

```swift
let userScript = WKUserScript(
    source: "someFunction()",       // Javascript function
    injectionTime: .atDocumentEnd,	// WebView 로드 직후 source를 넣어서 실행한다.
    forMainFrameOnly: true
)

let userContentController = WKUserContentController()
userContentController.addUserScript(userScript)
let webView = WKWebView(frame: frame, configuration:configuration)
```

- [`injectionTime`](https://developer.apple.com/documentation/webkit/wkuserscriptinjectiontime)에 지정된 시점에 `source`에 전달한 함수를 호출한다.
    - `atDocumentStart` : Web page의 document element가 생성된 후, 로드되기 전에 주입
    - `atDocumentEnd` : Document element 로드된 후, subresource가 로드되기 전에 주입


## 2. JavaScript 함수 직접 호출

`WKWebView`의 `evaluateJavaScript(_:completion:)` method를 실행하여 원하는 시점에 JavaScript 코드를 실행하고 `completion` closure로 실행 결과를 받는다.

```swift
webView.evaluateJavaScript("someFunction()") { object, error in
    if let error = error {
        print(error.localizedDescription)
        return
    } 
    
    guard let result = object else {
        print("There is no result value")
        return
    }

    print("Get Result :", result)
}
```

- `object`와 `error`는 배타적으로 값을 갖는다.
    - `object`가 `nil`이면 `error`는 값을 갖는다.
    - `error`가 `nil`이면 `object`는 값을 갖는다.

# JS to Native

## 함수 호출

JavaScript code에서 native code를 호출할 때는 '**message handler**'를 사용한다.

```javascript
webkit.messageHandlers.MESSAGE_HANDLER_NAME.postMessage(message)
```

여기서 `MESSAGE_HANDLER_NAME`을 native code에서 `WKUserContentController`의 script handler로 등록한다.

```swift
class SomeViewController: UIViewController {

    func configureWebView() {
        // Message handler 등록
        let userContentController = WKUserContentController()
        userContentController.add(self, name: "MessageHandlerName")
        
        let configuration = WKWebViewConfiguration()
        configuration.userContentController = userContentController
        let webView = WKWebView(frame: frame, configuration: configuration)
    }
}
```

`add(_:name:)` method에 등록할 handler는 `WKScriptMessageHandler` protocol을 채택해야 한다. Javascript에서 native code를 호출하면 이 protocol에 정의된 `userContentController(_:didReceive)` method로 message(value)가 전달된다.

```swift
extension SomeViewController: WKScriptMessageHandler {

    func userContentController(
        _ userContentController: WKUserContentController,
        didReceive message: WKScriptMessage
    ) {
        if message.name == "MessageHandlerName" {
            print(message.body)
        }
    }
}
```

> Message로는 문자열 하나만 보낼 수 있으므로, 여러 개의 서로 다른 타입의 값을 보내려면 JSON같은 포맷으로 전달한다.

## Native Alert 호출

`WKUIDelegate` protocol을 구현하면 JavaScript에서 발생시킨 alert을 native alert으로 변환할 수 있다.

아래와 같이 alert에서 사용할 message를 받아 native alert을 보여주고, 확인/취소 버튼을 선택할 때 action을 다시 JavaScript code로 보내도록 구현할 수 있다.

```swift
extension SomeViewController: WKUIDelegate {

    func webView(
        _ webView: WKWebView,
        runJavaScriptConfirmPanelWithMessage message: String,
        initiatedByFrame frame: WKFrameInfo,
        completionHandler: @escaping (Bool) -> Void
    ) {
        // JavaScript에서 보낸 alert을 native alert으로 생성
        let alert = UIAlertController(
            title: nil, 
            message: message, 
            preferredStyle: .alert
        )
        
        // Ok button을 누를 때 confirm
        let okAction = UIAlertAction(
            title: "OK", 
            style: .default, 
            handler: { action in 
                completionHandler(true)
            }
        )

        // Cancel button을 누를 떄 cancel
        let cancelAction = UIAlertAction(
            title: "cancel", 
            style: .cancel, 
            handler: { action in 
                completionHandler(false)
            }
        )
        
        alert.addAction(okAction)
        alert.addAction(cancelAction)            
        present(alert, animated: true, completion: nil)
    }
}
```

# Reference

- [[SWIFT] 웹뷰와 자바스크립트 연동 (Native JavaScript 통신 방법)](https://g-y-e-o-m.tistory.com/13)
- [[iOS - swift] Javascript Interface (웹뷰와 자바스트립트 연동, WKWebView, WKScriptMessageHandler)](https://ios-development.tistory.com/401)
- [WKWebView에서 Javascript Alert 띄우기](https://kka7.tistory.com/69)