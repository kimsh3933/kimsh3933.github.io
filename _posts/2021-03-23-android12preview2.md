---

date: 2021-03-23
title: "안드로이드 12 개발자 프리뷰2 공개"
categories: Android

---

# 안드로이드 12 개발자 프리뷰2 공개



지난 게시글에서 [TheAndroidShow에서 안드로이드 12의 프리뷰가 공개되었다고](https://kimsh3933.github.io/android/qna-compose/#interviews) 언급했습니다. 그때는 그렇게 많은 기능들이 공개되지 못했었는데요, [지난주 3월 17일에 안드로이드 프리뷰 2 버전이 공개 되었습니다](https://android-developers.googleblog.com/2021/03/android-12-developer-preview-2.html). 뷰에 블러를 주거나 PIP기능이 개선되는 등 눈에 띄는 기능들이 추가 되었습니다.



### [앱 오버레이 컨트롤](https://developer.android.com/about/versions/12/features#hide-application-overlay-windows)

- 시스템 경고창 같은 오버레이 콘텐츠들은 사용자를 방해할 수 있으므로 앱을 표시하기 전에 권한을 요청해야 합니다.
- Android 12 부터는 오버레이를 콘텐츠 위에 표시할 수 있는지 여부를 제어할 수 있습니다.
- [Manifest 를 설정하고](https://developer.android.com/reference/android/Manifest.permission#HIDE_OVERLAY_WINDOWS) [Window.setHideOverlayWindows()](https://developer.android.com/reference/android/view/Window#setHideOverlayWindows(boolean)) 를 호출하여 앱의 창이 표시될 때 모든 오버레이가 숨겨져야 함을 나타낼 수 있습니다. 오버레이를 숨기는 것에 관한 작업 입니다.

### [잠금 화면 알림 작업에 대한 보안 강화](https://developer.android.com/about/versions/12/features#notification-secure)

- Android 12에서는 기기 잠금 화면에 표시되는 알림에 대해 세분화된 보안 제어를 추가 하였습니다.
- [setAuthenticationRequired](https://developer.android.com/reference/android/app/Notification.Action.Builder#setAuthenticationRequired(boolean)) 를 설정하여 잠긴 화면에서 notification의 action을 호출하면 항상 인증 요청이 발생합니다. false인 경우에는 OS가 인증이 필요한지 하지 않은지 결정을 합니다.

### [앱 다이제스트에 대한 액세스](https://developer.android.com/about/versions/12/behavior-changes-12)

- 설치된 앱 패키지의 무결성을 확인해야 하는 앱의 경우 설치된 앱의 체크섬을 플랫폼에 직접 쿼리할 수 있는 새로운 API를 도입 하였습니다.
- SHA256, SHA512, Merkle Root 등의 여러 다이제스트 알고리즘 중에서 선택할 수 있습니다.



### [둥근 모서리 지원](https://developer.android.com/about/versions/12/features#rounded_corner_apis)

- 많은 최신 장치는 모서리가 둥근 화면을 사용하고 있습니다. 
- 이러한 장치에서 뛰어난 UX를 제공하려면 개발자는 둥근 모서리를 고려하고 주변 UI요소를 조정하여 잘리지 않도록 해야 합니다.
- [RoundedCorner](https://developer.android.com/reference/android/view/RoundedCorner?hl=en)는 반지름, 중심점 등 코너에 대한 정보를 가지고 있습니다.
- [Display.getRoundedCorner()](https://developer.android.com/reference/android/view/Display#getRoundedCorner(int)) 를 호출하여 각 둥근 모서리에 대한 세부 정보를 가져올 수 있습니다.
- [WindowInsets.getRoundedCorner()](https://developer.android.com/reference/android/view/WindowInsets#getRoundedCorner(int)) 를 호출하여 앱 경계와 관련된 모서리 정보를 가져올 수 있습니다.



### [Picture In Picture(PIP) 개선](https://developer.android.com/about/versions/12/features/pip-improvements)

- 제스처 네비게에션을 사용하는 사람들을 위해 홈으로 스와이프 할 때 앱이 PIP모드로 전환 되는 방식을 개선 하였습니다.
- [앱이 자동으로 PIP를 사용하도록 설정하면](https://developer.android.com/reference/android/app/PictureInPictureParams.Builder#setAutoEnterEnabled(boolean)) 시스템은 이제 앱을 홈에서 PIP모드로 직접 전환하도록 합니다.
  - up-to-home 애니메이션을 기다리지 않고서 바로 PIP가 실행됩니다.
  - Activity#enterPictureInPictureMode 를 호출하지 않고서도 자동으로 PIP 모드를 설정합니다.
- 탭 액션에 관한 동작도 개선 하였습니다.
  - 한번 탭하여 PIP 창을 확장하고 컨트롤 표시 -> 확장 하지 않고 컨트롤만 표시
  - 두번 탭하여 PIP를 전체화면 모드로 전환 -> 현재 PIP크기와 최대 크기 PIP 전환
- 오른쪽이나 왼쪽 가장자리로 드래그 하여 숨길 수 있습니다.



### [컴패니언 장치 앱 깨우기](https://developer.android.com/about/versions/12/features#keep-awake)

- 스마트 워치 및 피트니스 트래커와 같은 컴패니언 장치를 관리하는 앱의 경우 연결된 장치가 근처에 있을 때 마다 앱이 실행되고 연결되어 있는지 확인하는 것이 어려울 수 있습니다.
- 이를 쉽게 하기 위해 [CompanionDeviceService](https://developer.android.com/reference/android/companion/CompanionDeviceService) 가 추가되어 [Companion Device Manager](https://developer.android.com/reference/android/companion/CompanionDeviceManager) 를 확장하고 있습니다.
- 컴패니언 장치가 근처에 있을 때 마다 시스템이 앱을 깨우도록 구현할 수 있습니다.
- [새로운 컴패니언 장치 프로필을 사용할 수 있습니다.](https://developer.android.com/reference/android/companion/AssociationRequest.Builder#setDeviceProfile(java.lang.String))

### [Bandwidth Estimator 개선](https://developer.android.com/about/versions/11/features/5g#estimator)

- 각 사용자가 사용할 수 있는 일반적인 대역폭을 알아야 하는 개발자를 위해 Bandwidth Estimator을 개선 하였습니다.
  - 이동 통신사별 총 처리량, Wi-Fi SSID, 네트워크 유형 및 신호수준의 추정치를 검색할 수 있도록 하였습니다.

### [더 쉬워진 블러, 색상 필터 및 기타 효과](https://developer.android.com/about/versions/12/features#rendereffect)

- Android 12에서는 뷰 및 렌더링 계층에 그래픽 효과를 더 쉽게 적용할 수 있습니다.

- [RenderEffect](https://developer.android.com/reference/android/graphics/RenderEffect) 를 활용하여 뷰에 흐림, 컬러 필터 등을 적용할 수 있습니다. 체인으로 결합할 수 있습니다.
- [view.setRenderEffect](https://developer.android.com/reference/android/view/View#setRenderEffect(android.graphics.RenderEffect))로 뷰에 효과를 적용할 수 있습니다.

```kotlin
view.setRenderEffect(RenderEffect.createBlurEffect(radiusX, radiusY, SHADER_TILE_MODE))
```

- Dialog에 흐림 표현을 줄 수 있으며, ImageView에 별도의 추가 처리 없이 블러처리를 할 수 있습니다.
- 또한 [Window.setBackgroundBlurRadius()](https://developer.android.com/reference/android/view/Window#setBackgroundBlurRadius(int)) 를 활용하면 배경에 있는 화면에 블러 효과를 줄 수 있습니다.



### 앱 호환성

[개발자 옵션에서 옵션을 켜고 끄면서 현재 테스트 하는 앱의 targetSDK를 올리지 않고서도 앱이 제대로 실행이 되는지 테스트를 할 수 있습니다.](https://developer.android.com/about/versions/12/reference/compat-framework-changes) 이를 통해 현재 앱들의 호환성 테스트를 좀 더 수월하게 할 수 있습니다.



### Android 12 테스트 하기

[일부 구글 레퍼런스 기기에 설치 할 수 있는 Android 12 이미지를 제공하고 있습니다.](https://developer.android.com/about/versions/12/download)

# Reference

- https://android-developers.googleblog.com/2021/03/android-12-developer-preview-2.html

- https://developer.android.com/about/versions/12
- https://developer.android.com/about/versions/12/behavior-changes-all
- https://developer.android.com/about/versions/12/features