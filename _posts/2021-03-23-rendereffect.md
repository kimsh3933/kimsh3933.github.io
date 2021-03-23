---

date: 2021-03-23
title: "Android 12의 RenderEffect 사용해보기"
categories: Android

---

# Android 12의 RenderEffect 사용해보기

<img src="https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-03-23-rendereffect/1.png?raw=true" alt="1" style="zoom:50%;" />

Android Preview 12에 있는 RenderEffect 중 BlurEffect를 적용한 결과 입니다.

제일 상단 왼쪽 부터

원본, Shader.TileMode.MIRROR

Shader.TileMode.CLAMP, Shader.TileMode.DECAL

Shader.TileMode.REPEAT, Shader.TileMode.CLAMP

순으로 tileMode를 설정했습니다.

제일 하단 아래쪽은 radius값을 100으로 설정했으며 나머지는 10으로 설정 하였습니다.

```kotlin
RenderEffect.createBlurEffect(radiusX, radiusY, Shader.TileMode.MIRROR)
```

위 코드에서 볼 수 있듯이 X축 Y축 별도로 blur 처리를 할 수 있습니다.



코드를 작성하다가 샘플 코드를 작성할때 보통 onCreate에 바로 작성을 하게 되는데 계속 안되어서 에뮬레이터를 여러번 생성해보고 gradle설정도 건드려 보고도 하다가 view.post를 하고 나니 effect가 적용 되었습니다. 

```kotlin
val effect = RenderEffect.createBlurEffect(radiusX, radiusY, Shader.TileMode.MIRROR)
view.post {
	view.setRenderEffect(effect)
}
```

이런식으로 작성을 하거나 onStart 이후 시점에 작성하거나 아니면 view가 attach된 이후에 작성 하는게 좋을 것 같습니다.

그리고 한가지 문제가 더 있었는데요, 홈화면에 갔다가 돌아오면 effect가 사라져 있다는 것 이었습니다. 그렇다면 일일이 onStart, onResume 에서 처리를 해야 한다는 말인데 해당 사항은 좀 더 지켜봐야 할 것 같습니다.



현재는 targetSdkVersion을 "S"로 할 때 Android S가 아닌 기기에서는 실행이 되지 않고 있습니다. 아직은 하위 버전 단말에서는 확인할 수 없습니다. 추후에 확인해봐야 할 것 같습니다.