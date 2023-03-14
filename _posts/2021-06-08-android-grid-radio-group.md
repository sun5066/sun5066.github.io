---
layout: post
title: "GridLayout + RadioGroup"
date: 2021-06-08 17:20:00 -0400
categories: android
tags: android, view, grid, layout
comments: 1
---

## **RadioGroup**

android.widget 패키지에 있는
RadioGroup의 코드를 보면 LinearLayout 을 상속받고 있습니다.

때문에 1줄의 가로/세로 정렬밖에 안되기 때문에

n개의 행/열을 가지려면 GridLayout 을 커스텀해서 만들어야합니다.

[소스코드 확인하기][git]

# 제트팩의 appcompat, gridlayout 필요
```
dependencies {
  ...
  implementation "androidx.appcompat:appcompat:{release-version}"
  implementation "androidx.gridlayout:gridlayout:{release-version}"
}
``` 


[git]: https://github.com/sun5066/android-grid-radio-group