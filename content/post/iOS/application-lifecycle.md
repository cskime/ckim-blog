---
title: "iOS App Lifecycle"
date: 2022-11-14T22:49:43+09:00
draft: false
summary: "iOS Application Lifecycle"
tags: ['iOS']
---

# Lifecycle

- 생명 주기(Lifecycle)란, '**시작과 종료 사이에서 일어나는 상태나 단계의 변화**'를 말한다.
- 즉, 앱의 생명 주기란 '**앱의 실행부터 종료까지 일어나는 일련의 상태 변화'**이다.

# App State

앱이 생명주기 동안 가지는 5가지 상태

1. **Not Running** : 앱이 실행되지 않음
2. **In-active** : 앱이 실행중이지만, 사용자와 상호작용 할 수 없는 상태 (직접 사용할 수 없는 상태)
    - 멀티태스킹 창에 진입했을 때
    - 앱 실행 중 전화, 알림 등에 의해 앱을 사용할 수 없게 될 때
3. **Active** : 앱이 실행중이면서, 사용자와 상호작용하여 event를 받을 수 있는 상태 (직접 사용할 수 있는 상태)
4. **Background** : 앱이 화면에서 사라지고, 상호작용도 할 수 없는 상태
    - 일반적인 경우, background 상태에 진입한 앱은 곧 바로 수 초 내에 **Suspended** 상태로 전환됨
    - 앱 entitlement에서 **Background task**를 설정하는 경우, 앱은 background 상태에 머물며 낮은 우선순위로 시스템 자원을 사용하여 작업을 이어나갈 수 있다.
        - 음악 앱 사용 중 홈 화면으로 나가도 음악이 재생됨
        - 앱 사용 중 홈 화면으로 나가도 데이터 동기화 작업이 유지됨
        - 시계 앱에서 타이머 설정 후 홈 화면으로 나가도 타이머가 계속 실행됨
5. **Suspended** : 앱 데이터가 메모리에만 저장되어 있고, 실제로 실행되지 않는 상태입니다.
    - 언제든 빠르게 다시 시작할 수 있도록 앱을 메모리에 유지시킴
    - 시스템 메모리가 부족해지면 iOS는 가장 먼저 suspended 상태에 있는 앱을 메모리에서 해제하고 공간을 확보 (low memory refresh)

## 상태 전환

다섯 가지 상태는 상호 전환될 수 있다.

{{< figure src="/images/iOS/application-lifecycle-1.png" width="70%" >}}

- '**Active**' 상태는 반드시 '**Inactive**' 상태를 거쳐야 한다.
    - 두 상태는 '**Foreground**' 상태로 묶어서 '**Background**' 상태와 대비되는 개념으로 본다.
- '**Inactive**' 상태가 되는 경우
    1. 앱이 실행될 때 : Not Running -> **In-active** -> Active
    2. 앱이 background 상태로 진입할 때 : Active -> **In-active** -> Background
    3. 앱이 foreground 상태로 진입할 때 : Background -> **In-active** -> Active

# Respond to Lifecycle

UIKit fremwork는 앱의 상태가 전환되는 특정 시점에 event를 발생시킨다. 특정 시점에 발생하는 event를 catch해서 **생명 주기의 특정 시점에 custom code를 실행**시킬 수 있다.

## Using NotificationCenter

앱이 실행되면 `UIApplication` singleton 객체가 생성되는데, application 객체는 **lifecycle 동안 앱의 상태가 전환되는 시점에 notification을 발생**시킴

`UIApplication`이 발생시키는 notification은 5가지. `NotificationCenter`에 이 notification의 observer를 등록해서 사용한다.
{{< figure src="/images/iOS/application-lifecycle-2.png" width="90%" >}}

## Using UIApplicationDelegate (App-based lifecycle)

iOS 12까지는 `UIApplicationDelegate`에서 생명 주기 관리

{{< figure src="/images/iOS/application-lifecycle-3.png" >}}

'**`UIApplicationDelegate`이 생명 주기를 관리한다**'는 것은 이 delegate의 method들이 생명 주기의 특정 시점에 호출된다는 것이다. 이렇게 호출되는 method들은 다섯 가지가 있다.

{{< figure src="/images/iOS/application-lifecycle-4.png" width="70%" >}}

이 method들을 `UIApplicationDelegate` protocol을 채택하고 있는 `AppDelegate`에 구현하여, 특정 시점에 custom code를 실행시킨다.

```swift
@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    ...

    func applicationDidBecomeActive(_ application: UIApplication) {
        print("Did Become Active")
    }

    func applicationWillResignActive(_ application: UIApplication) {
        print("Will Resign Active")
    }
    
    func applicationDidEnterBackground(_ application: UIApplication) {
        print("Did Enter Background")
    }

    func applicationWillEnterForeground(_ application: UIApplication) {
        print("Will Enter Foreground")
    }
    
    func applicationWillTerminate(_ application: UIApplication) {
        print("Will Terminate")
    }
}
```

## Using UISceneDelegate (Scene-based lifecycle)

- iOS 13부터 multitasking 기능이 추가되면서 `Scene`이라는 개념 도입
- iOS 13 이상의 OS가 설치된 기기에서 실행되는 앱은 기본적으로 `UIScene`이 생명 주기를 관리

> 앱에서 scene을 사용하지 않고 `UIApplicationDelegate`를 구현하면 '**App-based lifecycle**'을 사용할 수 있다.

{{< figure src="/images/iOS/application-lifecycle-5.png" >}}

Scene이 관리하는 생명 주기에서 특정 시점에 호출되는 method들은 `UISceneDelegate`에 정의되어 있다.

{{< figure src="/images/iOS/application-lifecycle-6.png" width="90%" >}}

이 method들을 `UISceneDelegate` protocol을 채택하고 있는 `SceneDelegate`에 구현하여, 특정 시점에 custom code를 실행시킴

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    ...

    func sceneDidBecomeActive(_ scene: UIScene) {
        print("Did Become Active")
    }

    func sceneWillResignActive(_ scene: UIScene) {
        print("Will Resign Active")
    }

    func sceneWillEnterForeground(_ scene: UIScene) {
        print("Will Enter Foreground")
    }

    func sceneDidEnterBackground(_ scene: UIScene) {
        print("Did Enter Background")
    }
}
```

## App-Based vs Scene-Based Lifecycle

- Scene-based : 화면에 나타나고 사라지는 것과 관련된 lifecycle 관리 ('**UI Lifecycle**')
- App-based : 앱의 실행/종료와 관련된 lifecycle 관리 ('**Process Lifecycle**')

{{< figure src="/images/iOS/application-lifecycle-8.png" width="90%" >}}

- Process Lifecycle
    - `application(_:didFinishLaunchingWithOptions:)`
    - `applicationWillTerminate(_:)`
- UI Lifecycle
    - `sceneDidBecomeActive(_:)`
    - `sceneWillResignActive(_:)`
    - `sceneDidEnterBackground(_:)`
    - `sceneWillEnterForeground(_:)`

### UI Lifecycle

- `UISceneDelegate`에는 '**willTerminate**' event가 없고 '**UI가 화면에 표시되고 사라지는 것과 관련된 event**'와 관련된 method만 존재함
- 이것은 '**Scene-Based 생명 주기는 새로운 lifecycle이 아니라, 기존 App-Based cycle에서 UI와 관련된 lifecycle만 분리한 것**'이기 때문

{{< figure src="/images/iOS/application-lifecycle-7.png" width="90%" >}}

### Process Lifecycle

- `UIScene`을 사용하더라도, 앱은 여전히 `UIApplicationDelegate`에 정의된 lifecycle method를 사용함
- 이 method들은 UI와 직접 관련이 없고, '**앱의 실행/종료**'에만 관련이 있음

# Reference

- [Managing your app's life cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)
