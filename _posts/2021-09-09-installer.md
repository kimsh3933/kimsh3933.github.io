---

date: 2021-08-24
title: "Installer Package Name"
categories: Android

---



[PackageManager](https://developer.android.com/reference/android/content/pm/PackageManager) 의 [getInstallSourceInfo](https://developer.android.com/reference/android/content/pm/PackageManager) (API30 이상), [getInstallerPackageName](https://developer.android.com/reference/android/content/pm/PackageManager#getInstallerPackageName(java.lang.String)) (API5 ~ API30미만) 을 통해서 installer 패키지 정보를 알 수 있습니다. 즉 어느 앱을 통해서 설치했느냐를 알 수 있습니다.



adb에서 

```
adb shell pm list packages -i [패키지 이름]
```

을 통해서 검색을 하면 installer package를 알 수 있습니다.

installer package의 특징은 다음과 같습니다.

- installer의 패키지 명에 따라 값이 변화 합니다
  - 구글 플레이 스토어를 통해서 설치한 경우는 `com.android.vending` 
  - 다른 개별스토어의 경우에는 스토어앱의 패키지 명
  -  브라우저나 탐색기 앱에서 실행시켜 설치한 경우에는 브라우저나 탐색기앱
  - 안드로이드 스튜디오로 직접 빌드해서 Run한 경우에는 null
- 해당 값은 앱의 속성이므로 앱의 데이터를 모두 비운다고 해도 삭제되지 않는 값 입니다.
- 만약 다른 installer를 통해서 업데이트 한 경우에는 업데이트 시킨 installer의 패키지로 변경 됩니다.


