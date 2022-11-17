---
title: "WKScriptMessageHandler에 의한 memory leak 해결하기"
date: 2022-11-17T23:17:03+09:00
draft: false
summary: "Message handler를 등록할 때 reference cycle에 의한 memory leak을 방지하는 방법"
tags: ['iOS']
---

# Overview

WebKit에서 JS interface를 사용하기 위해 message handler를 등록할 때, handler가 메모리에서 해제되지 않을 수 있음

# 원인

- `WKUserContentController`의 `add(_:name:)` method는 handler 객체를 강한 참조함
- `SomeViewController`가 메모리에서 해제될 때 handler도 명시적으로 해제해야 하지만, 이것을 누락하는 실수를 할 가능성이 높음
- 명시적으로 해제하지 않을 때, 순환 참조 발생

```swift
class SomeViewController: UIViewController, WKScriptMessageHandler {

    func configureWebView() {
        
        // Web view가 self(SomeViewController)를 강한 참조한다.
        let userContentController = WKUserContentController()
        userContentController.add(self, name: "MessageHandlerName")
                
        let configuration = WKWebViewConfiguration()
        configuration.userContentController = userContentController
            
        let webView = WKWebView(frame: frame, configuration: configuration)
        view.addSubview(webView)
    }

    // MARK: WKScriptMessageHandler

    func userContentController(
        _ userContentController: WKUserContentController, 
        didReceive message: WKScriptMessage
    ) {
        print(message.body)
    }
}
```

# 해결방법

## 1. 직접 handler 제거

- `WKUserContentController`의 `removeScriptMessageHandler(forName:)` method를 사용하여 handler 객체에 대한 참조를 해제
- iOS 14부터 `removeAllScriptMessageHandlers()` method 사용 가능
    - Handler를 여러 개 추가했을 때 한 번에 해제

```swift
class SomeViewController: UIViewController, WKScriptMessageHandler {
    
    var messageHandlers = [String]()

    deinit {
        removeAllScriptMessageHandlers()
    }
    
    func removeAllScriptMessageHandlers() {
    	let userContentController = webView
        	.configuration
        	.userContentController
            
    	if #available(iOS 14.0, *) {
        	userContentController.removeAllScriptMessageHandlers()
        } else {
            messageHandlers.forEach(
                userContentController.removeScriptMessageHandler(forName:)
            )
        }
    }
}
```

## 2. Handler를 약한 참조하기

- ARC에 의한 메모리 관리 활용  
- 매번 직접 참조를 해제시키는 것 보다 ARC에 의해 관리되는 것이 더 자연스럽고 실수를 줄일 수 있음
- 구현 방법
    - `WKScriptMessageHandler`를 구현하는 class 정의    
        ```swift
        class WeakScriptMessageHandler: NSObject, WKScriptMessageHandler {
            weak var handler: WKScriptMessageHandler?

            init(handler: WKScriptMessageHandler) {
                self.handler = handler
                super.init()
            }

            func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
                handler?.userContentController(userContentController, didReceive: message)
            }
        }
        ```
    - Handler를 등록할 때 `WeakScriptMessageHandler`로 wrapping
        ```swift
        class SomeViewController: UIViewController {

            override func viewDidLoad() {
                super.viewDidLoad()

                let userContentController = WKUserContentController()
                userContentController.add(
                    WeakScriptMessageHandler(handler :self), 
                    name: "MessageHandlerName"
                )
            }

            func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
                print(message)
            }
        }
        ```
    - WebKit은 `SomeViewController`대신 `WeakScriptMessageHandler`를 참조하므로, `SomeViewController`의 reference count를 증가시키지 않음.

> 참고 : [wkwebview message handler memory leak(쉽게 실수하는 WKWebView 메모리 누수 수정)](http://monibu1548.github.io/2019/11/17/wkwebview-memory-leak/)