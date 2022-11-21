---
title: "Podfile 사용법"
date: 2022-11-20T21:36:28+09:00
draft: false
summary: "Podfile 사용법"
tags: ['iOS']
---

# [Target](https://guides.cocoapods.org/syntax/podfile.html#target)

`target`은 Xcode의 target과 동일한 개념으로, Xcode에 추가된 target과 동일한 이름을 symbol로 입력해서 사용

## Multiple Targets

각 target에 해당하는 `target` block을 정의하여 여러 개의 target에 대해 의존성을 별도로 관리할 수 있다.

```ruby
target 'MyApp' do
  use_frameworks!
  pod 'Alamofire'
end

target 'MyAppTests' do
  use_frameworks!
  pod 'Quick'
  pod 'Nimble'
end
```

## 부모-자식 Target

부모 target에서 설치한 라이브러리들 자식 target에서 같이 사용하려면, 자식 target을 부모 target 하위에 작성한다.

```ruby
target 'MyApp' do
  use_frameworks!
  pod 'Alamofire'

  target 'MyAppTests' do
    inherit! :search_paths
    pod 'Quick'
    pod 'Nimble'
  end
end
```

- [inherit!](https://guides.cocoapods.org/syntax/podfile.html#inherit_bang) : 현재 target의 상속 mode 설정
  - `none` : 상위 target으로부터 어떤 동작도 상속받지 않음
  - `complete` : 상위 target의 모든 동작을 상속받음
  - `search_paths` : 상위 target의 search path만 상속받음
- 일반적으로 하위 target에서는 `inherit! search_paths`를 설정한다.

> Unit test를 작성할 때, pod으로 설치한 라이브러리를 import하면 Podfile에서 라이브러리를 명시했는데도 **'Missing required module'** error가 발생하는 경우가 있다. Test target을 하위 target으로 작성했다면, test target에 상속 설정이 되어있는지 확인해 보자.

## Abstract Target

상속 관계를 만들지 않으면서 여러 개의 target에 같은 라이브러리를 설치할 때 사용. **Abstract target을 프로젝트에는 존재하지 않음**

```ruby
# There are no targets called "Shows" in any Xcode projects
abstract_target 'Shows' do
  pod 'ShowsKit'
  pod 'Fabric'

  # Has its own copy of ShowsKit + ShowWebAuth
  target 'ShowsiOS' do
    pod 'ShowWebAuth'
  end

  # Has its own copy of ShowsKit + ShowTVAuth
  target 'ShowsTV' do
    pod 'ShowTVAuth'
  end
end
```

> `abstract_target` block을 사용하지 않더라도, root에 설치할 라이브러리를 명시하면 하위에 정의해 둔 target들이 공통으로 해당 라이브러리를 설치한다. Abstract target은 **하위 target들을 보기 좋게 묶어 주는 것 + 범위를 지정해 주는 것**
> ```ruby
> pod 'ShowsKit'
> pod 'Fabric'
>
>
> # Has its own copy of ShowsKit + ShowWebAuth
> target 'ShowsiOS' do
>   pod 'ShowWebAuth'
> end
>
> # Has its own copy of ShowsKit + ShowTVAuth
> target 'ShowsTV' do
>   pod 'ShowTVAuth'
> end
> ```

# 기타 명령어

## [use_frameworks!](https://guides.cocoapods.org/syntax/podfile.html#use_frameworks_bang)

`use_frameworks!`는 Pods 프로젝트에서 static library 대신 framework을 사용하도록 하는 문법이다. Framework의 linking 방식을 static과 dynamic 중 선택할 수 있다. 만약 Swift를 사용하고 dynamic library(framework)를 사용해야 한다면, `use_frameworks!`를 target block 안에 추가해 준다.<br>
(옵션을 명시하지 않으면 dynamic이 기본값으로 설정되는 것 같다.)

```ruby
# Default
target 'MyApp' do
  use_frameworks!
end

# Dynamic Linking
target 'MyApp' do
  use_frameworks! :linkage => :dynamic
  pod 'AFNetworking', '~> 1.0'
end

# Static Linking
target 'ZipApp' do
  use_frameworks! :linkage => :static
  pod 'SSZipArchive'
end
```

## [inhibit_all_warnings!](https://guides.cocoapods.org/syntax/podfile.html#inhibit_all_warnings_bang)

`pod install` 명령어를 실행하면 Warning이 출력되는 경우가 있는데, 이것을 신경쓰지 않고 싶다면 `inhibit_all_warnings!`를 추가한다. 이 명령어는 출력되는 모든 warning 문구를 제거해 준다.

```ruby
target 'MyApp' do
  inhibit_all_warnings!
end
```

만약 특정 라이브러리의 warning만 제거하고 싶거나 또는 남기고 싶다면, 설치할 라이브러리에 `inhibit_warnings` 옵션을 추가해 준다.

```ruby
target 'MyApp' do
  # Warning 제거
  pod 'Alamofire', :inhibit_warnings => true

  # Warning 제거하지 않음
  pod 'Alamofire', :inhibit_warnings => false
end
```
# Target 지정

```ruby
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/Artsy/Specs.git'

platform :ios, '9.0'
inhibit_all_warnings!

target 'MyApp' do
  pod 'Alamofire', '~> 3.1'

  target 'MyAppTests' do
    inherit! :search_paths
    pod 'Quick', '~> 2.0.1'
  end
end
```

- `inhibit_all_warnings!` : Pod 라이브러리에서 발생하는 warning 제거
- `inherit! :search_paths` : 
- 

# 특정 version으로 설치

## Version 지정

> [Specifying pod versions](https://guides.cocoapods.org/using/the-podfile.html#specifying-pod-versions)

> Pod 라이브러리의 버전은 [Semantic Versioning](http://semver.org/)을 따른다.

- 버전을 명시하지 않으면 설치 가능한 최신 버전으로 설치
- `0.1.2` : 0.1.2로 설치
- Logical operator 사용
  - `>=(>) 0.1.2` : 0.1.2를 포함하고(또는 포함하지 않고) 0.1.2보다 큰 최신 버전 설치
  - `<=(<) 0.1.2` : 0.1.2보다 작은 버전 중 최신 버전 설치
- Optimistic operator 사용
    - Patch version까지 사용한다면, 다음 minor 버전 미만에서 최신 버전 설치 (e.g. `~> 0.1.2` : 0.2보다 작은 최신 버전)
    - Minor version까지 사용한다면, 다음 major 버전 미만에서 최신 버전 설치 (e.g. `~> 0.1` : 1.0보다 작은 최신 버전)
    - Major version만 사용한다면, 다음 major 버전 미만에서 최신 버전 설치 (e.g. `~> 0` : 1.0보다 작은 최신 버전)

## Repository 지정

> [From a podspec in the root of a library repo](https://guides.cocoapods.org/using/the-podfile.html#from-a-podspec-in-the-root-of-a-library-repo)

- main branch를 기본으로 가져옴
  ```ruby
  pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git'
  ```
- branch 지정
  ```ruby
  pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :branch => 'dev'
  ```
- tag 지정
  ```ruby
  pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :tag => '3.1.1'
  ```
- commit 지정
  ```ruby
  pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :commit => '0f506b1c45'
  ```

# Build Setting 설정

[`post_install`](https://guides.cocoapods.org/syntax/podfile.html#post_install)은 CocoaPods의 [Hook](https://guides.cocoapods.org/syntax/podfile.html#group_hooks) 중 하나로, 이름 그대로 install이 끝난 뒤(post)에 Xcode 설정을 변경할 수 있는 기회를 제공한다.

`post_install` block은 `Pod::Installer`라는 객체를 받는데, installer를 사용해서 target별로 build configuration의 속성들을 지정해 줄 수 있다.

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['GCC_ENABLE_OBJC_GC'] = 'supported'
    end
  end
end
```

Unit Test에서 외부 라이브러리를 `@testable import`하려고 하면 다음과 같은 error가 발생한다.

{{< figure src="/images/iOS/podfile-1.png" >}}

이 error는 `Pods` 프로젝트에서 Alamofire target의 build settings 중 **'Enable Testability'** 옵션이 `NO`로 설정되어 있기 때문에 발생하는데, build settings의 값을 직접 변경해주더라도 pod을 install할 때 마다 값이 초기화된다.

해당 라이브러리의 testability 속성을 항상 `YES`로 설정하기 위해, 아래와 같이 pod install이 모두 끝난 뒤 'Alamofire' target의 특정 build configuration에 대해 'Enabled Testability' 속성을 `YES`로 바꿔준다.

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    if target.name == 'Alamofire'
      target.build_configurations.each do |config|
        if config.name == 'Debug-Stage'
          config.build_settings['ENABLE_TESTABILITY'] = 'YES'
        end
      end
    end
  end
end
```

# 기타 pod 설정

- 특정 pod 라이브러리에서 발생하는 warning 무시
  ```ruby
  pod 'Alamofire', :inhibit_warnings => true
  ```
- 특정 pod 라이브러리 target의 build configuration 설정
  ```ruby
  pod 'PonyDebugger', :configurations => ['Debug', 'Release']
  #or 
  pod 'PonyDebugger', :configurations => 'Debug'
  ```
- Local 경로 지정 : Remote repository를 사용할 수 없는 특수한 상황에서 local 경로에 위치한 pod 라이브러리 설치(`.podspec` 파일이 있는 directory)
  ```ruby
  pod 'Alamofire', :path => '~/Documents/Alamofire'
  ```

# Reference

- [The Podfile](https://guides.cocoapods.org/using/the-podfile.html)
- [Podfile Syntax Reference](https://guides.cocoapods.org/syntax/podfile.html#podfile)
- [CocoaPods 유용한 정보 모음: Cling Jang](https://github.com/ClintJang/cocoapods-tips)