---
title: "시간대(Timezon)를 고려한 Date 다루기"
date: 2022-11-16T21:16:05+09:00
draft: false
summary: "시간대(Timezone)를 고려해서 Date 객체를 다루는 방법"
tags: ['iOS']
---

# Date

- [`Date`](https://developer.apple.com/documentation/foundation/date) : Calendar system 또는 시간대(timezone)에 독립적인 특정 시점의 날짜를 가리키는 객체.
- 즉, `Date`는 '**기준시([GMT](https://ko.wikipedia.org/wiki/그리니치_평균시))**'를 나타낸다.
- `Date` 인스턴스를 console에 출력해 보면 현재 날짜 및 시간과 다르게 출력되는데, '**우리나라는 GMP+9 시간대에 있기 때문**'이다. 출력된 시간은 현재 시간으로부터 정확히 9시간 전이다.
    ```swift
    let today = Date()
    print(today)

    // 현재 : 2022-11-16 21:19:40
    // 출력 : 2022-11-16 12:19:40 +0000
    ```

## 기준시(또는 그리니치 평균시, GMT)와 시간대(Timezone)

'**GMT(Greenwich Mean Time)**'는 런던 그리니치 천문대의 경도를 기준으로 하는 기준시이다. 1970년 1월 1일을 기점으로 사용하는 UTC(협정 세계시)와 표현만 다른 같은 개념이다.

시간대(timezone)는 그리니치 천문대를 기준 경도(0)로 하여 세계 각국의 경도값에 따라 '**시간차**'가 발생하는 범위이다. 국가 별로 경도에 따라 기준시(GMT)로부터 얼마 만큼의 시간차가 발생하는지 계산하고, '**GMT+시차**'의 형태로 local timezone을 나타낸다.

> [국가별 시간대](https://ko.wikipedia.org/wiki/시간대)

우리나라는 GMT+9 시간대에 속해 있다. `Date` 객체는 기준시에 따른 날짜를 나타내므로, 우리 나라 시간대에 맞는 날짜/시간을 사용하려면 `Date`에 GMT+9 시간대의 시차를 적용해야 한다.

# 시간대를 고려하여 날짜 사용하기

## 시차를 직접 계산하는 방법

`Timezone`의 `secondsFromGMT(for:)` method는 주어진 날짜(`Date`)에 대해 '**현재 timezone의 GMT로부터 시차**'를 반환한다. 이 값을 `Date` 인스턴스에 더하면 현재 timezone에서의 정확한 날짜/시간을 얻을 수 있다.

```swift
let today = Date()
let timezone = Timezone.autoupdatingCurrent                 // 현재 설정된 timezone
let secondsFromGMT = timezone.secondsFromGMT(for: today)    // 현재 timezone의 시차
let localizedDate = today.addingTimeInterval(TimeInterval(secondsFromGMT))
print(localizedDate)
```

## `DateFormatter`를 사용하는 방법

`DateFormatter`의 [`timeZone`](https://developer.apple.com/documentation/foundation/timezone) 속성에 현재 timezone을 지정하면, `DateFormatter`가 formatting된 날짜를 표시할 때 시차를 적용해 준다.

```swift
func localizedRepresentation(_ date: Date, format: String) -> String {
    let dateFormatter = DateFormatter()
    dateFormatter.timeZone = TimeZone.autoupdatingCurrent
    dateFormatter.dateFormat = format
    return dateFormatter.string(from: date)
}

let localizedDate = localizedRepresentation(today, format: "yyyy-MM-dd HH:mm")
print(localizedDate)
```