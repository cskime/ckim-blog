---
title: "iOS Static/Dynamic Library/Framework"
date: 2022-11-18T20:42:50+09:00
draft: false
summary: "Static, dynamic의 개념 및 차이, Linking의 개념"
tags: ['iOS']
---

# Overview

- Library : 코드와 데이터의 모음
- Framework : Library 외에 문자열, image 등 resource들까지 포함하는 패키지(package)
- Link : 프로젝트 소스코드와 외부 라이브러리 또는 프레임워크를 '**실행 파일**'로 병합(merge)하는 작업
	- Linker : Link를 하는 프로그램
- Linking 방식에 따른 분류
	- Static : 실행 파일에 라이브러리 또는 프레임워크에 속한 파일들을 모두 복사하는 방식
	- Dynamic : 실행 파일에 외부 라이브러리 또는 프레임워크의 '**참조**'만 복사하는 방식

# Library vs. Framework

- Library : 프로젝트 외부의 독립된 코드 및 데이터 파일들의 집합. iOS 개발할 때는 Xcode target에 속하지 않은 외부 target을 의미.
- Framework : Library + resource files(e.g. 문자열(strings), images)
	- Framework가 다른 framework을 포함할 수도 있다. (== '**Umbrella Framework**')
	- Framework는 여러 버전의 bundle format을 가지고 있어서 오래된 프로그램(하위 버전)에서도 동작할 수 있다.
	- `.framework` 확장자를 가짐

# Linking

- Link : Library 또는 framework를 프로젝트 소스코드와 합쳐서(merge) 하나의 실행 파일을 만드는 작업
	- Linker : Linking을 하는 프로그램
- 소스코드와 외부 라이브러리를 **link**해서 device(e.g. iPhone, iPad, mac 등)에서 실행 가능한 '**실행 파일(executable file)**'을 생성한다.

## Statkc vs. Dynamic

Linking 방식에 따라 static, dynamic을 나눈다.

### Static(정적)

외부 library 또는 framework의 파일들을 프로젝트에 모두 복사해서 가지고 있는 것
- 모든 library 또는 framework 코드들이 메모리에 로드됨
- 직접 메모리에 접근해서 코드 실행
- `.a` 확장자

{{< figure src="/images/iOS/static-dynamic-library-framework-1.png" width="80%" >}}

**특징**
- 소스코드가 많아질수록 빌드 시간이 길어지고 메모리를 많이 차지한다.
- Library 또는 framework가 업데이트되면, 개발자가 직접 업데이트된 소스코드들을 프로젝트에 import 해야 함

### Dynamic(동적)

- 실행 파일에 외부 library의 '**참조**'만 복사해 둔다. **(코드를 직접 복사하지 않는다.)** 
- 프로그램이 실행되면 라이브러리의 참조가 메모리에 로드되고, **라이브러리 코드를 호출할 때** 참조로부터 라이브러리를 로드한다.
- **Dynamic loader(dyld)** : Dynamic library를 로드하는 프로그램
- `.dylib` 확장자
- 'Text-Based Dynamic Library'는 `.tbd` 확장자

{{< figure src="/images/iOS/static-dynamic-library-framework-2.png" width="80%" >}}
     
**특징**
- Static library에 비해 실행 파일(executable file) 크기가 작아서 시작 시간(launch time)이 빠르고 메모리도 덜 차지한다.
- 참조를 통해 코드를 로드하므로, 메모리에 직접 접근하는 static에 비해 느리다.
	- 프로젝트에서 dynamic library를 많이 사용할 수록 앱이 실행된 후 코드를 실행시킬 때 시간이 많이 걸린다. (Splash 화면을 더 오래 보게 된다.)
- Dynamic library가 업데이트되면 다시 compile하지 않아도 변경사항이 반영된다.

# Reference

- [Apple Document - Overview of Dynamic Libraries](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html#//apple_ref/doc/uid/TP40001873-SW2)
- [ZeddiOS - (Static/Dynamic) Library](https://zeddios.tistory.com/1308)
- [Static and Dynamic Libraries and Frameworks in iOS](https://www.vadimbulavin.com/static-dynamic-frameworks-and-libraries/)
