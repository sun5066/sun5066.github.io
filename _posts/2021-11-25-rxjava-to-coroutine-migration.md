---
title: "android RxJava 코루틴/Flow 으로 마이그레이션"
date: 2021-12-03 14:15:00 -0400
categories: android
---

# Why Coroutine?


 - "비동기적"인 코루틴을 사용하여, RxJava 보다 나은 퍼포먼스를 기대
 - 공식 Develop에서 제공하는 가이드
 - 짧은 러닝커브 및 코드 가독성 향상

---

# Migration


 ## RxJava를 사용중인 목록
 
 - REST API
 - 아이템 리소스 다운로드 및 압축 해제
 - WebView Bridge
 - 각 화면별 타이머기능(방송 시간, 테이프 녹음 시간, 애니메이션 재생 쿨타임 등)


 ## 어떻게 대응?
 
 - 각 View(Activity, Fragment)에서는 lifecycleScope 사용
 - ViewModel 에서는 viewModelScope 사용
 - REST API 함수 반환타입에서 Observable<> Wrapping 제거 및 suspend 키워드 추가

 > 클럽라이브 구조상 RxJava 사용비중이 적기 때문에 생각보다 작업사항이 적었음


 ## 실제 변경후 느낀점

 - 기대했던 만큼 성능차이가 나지않는다..!
 - RxJava에 비해 부족한 점이 많다.
 - flow의 catch에서 예외를 잡아내지 못하는 이슈가 있다. [[이슈 보러가기]](https://stackoverflow.com/questions/64411586/exception-in-flow-is-not-caught)
   - 제가 겪은 사항은 DB Server의 데이터 반환에서 생긴 문제였지만, flow catch에서 NullPointerException 이 발생하는걸 잡아내지 못한 경우였습니다.
 - RxJava 와 별개로 Glide를 Coil로 마이그레이션 했을때 성능차이가 크게 보였다. [[성능차이 보러가기]](https://jizard.tistory.com/224)

---

# 결론

 - 기존의 RxJava 사용중에 문제가 없다면, 굳이 급하게 바꿀 필요는 없다.
 - 코루틴에 관심이 있다면, 세미나 영상을 찾아보는것도 좋다.
 - 한글로 된 책은 "코틀린 동시성 프로그래밍" 이라는 책이 유일하지만, 현재는 구하기 힘들다.


## 참고사항

[강남언니 블로그](https://blog.gangnamunni.com/post/android-coroutine/)
