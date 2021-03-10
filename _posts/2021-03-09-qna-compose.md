---

date: 2021-03-10
title: "The Android Show(Jetpack Compose Beta)"
categories: Android

---

# The Android Show (Jetpack Compose Beta)



얼마전에 Android Jetpack Compose의 베타가 발표 되었습니다. 

거기에 맞춰 구글에서 라이브쇼를 열었는데요 https://www.youtube.com/watch?v=vRjJAWh6JPE

이번에 베타가 발표된 Jetpack Compose의 시연은 물론이고 실시간으로 사용자들의 질문을 받아 Q&A를 하는 시간도 있었습니다.

시연과 Q&A를 확인해보니 기존에 복잡했던 안드로이드 UI 작성과 확인이 더 편리해질 것으로 기대가 됩니다.

Jetpack Compose의 beta가 발표된 이번 TheAndroidShow의 주요 내용들을 요약해 보았습니다.



## Introduction

진행자의 오프닝에 이어 실제 사용 하고 있는 개발자의 인터뷰 영상이 나옵니다. Lyft의 개발자는 Compose를 활용하여 디자인 시스템을 다시 개발하고 있다고 합니다. 그리고 Compose에서 애니메이션 기능을 가장 기대하고 있다고 합니다. 기존 뷰에 비하면 정말 쉽게 할 수 있다고 합니다. 

그 다음 Jetpack compose를 담당한 디렉터가 나와 jetpack compose를 개발한 이유를 설명합니다. 적은 코드로 멋진 앱을 빠르고 간단하게 만들기 위해서 라고 합니다. 모든 것이 선언 방식으므로 UI를 설명하기만 하면 Compose가 나머지를 처리한다고 합니다. 앱 변화 상태에 따라 UI가 저절로 업데이트 되기 때문에 UI구축 시간이 단축된다고 합니다.

Compose는 Kotlin API를 기반으로 한다고 소개하고 있습니다. 또한 기존의 안드로이드 코드와 완벽히 호환된다고 소개하고 있습니다.

마지막으로 시작하는데 도움이 되도록 여러 자료들을 준비해두었다고 합니다.

- 각 단계별로 배울 수 있는 교육과정 : https://developer.android.com/courses/pathways/compose
- 샘플앱 : https://github.com/android/compose-samples



## How Compose Works

그 다음은 Compose를 시연해보는 세션입니다.

시연자가 채팅 예시앱 JetChat을 구현 하면서 Compose가 어떻게 작동 되는지 보여 주었습니다.

https://youtu.be/vRjJAWh6JPE?t=431

텍스트를 화면에 텍스트를 추가 하기 위해서 텍스트 Composable을 추가 하였습니다. Message객체를 파라미터로 받는 composable method입니다. 해당 메소드를 작성하는 것 만으로 화면에 뷰가 하나 생겼습니다. xml없이 생겼습니다.

```kotlin
@Composable
fun Message(
		message: Message
) {
		Text(text = message.content)
}
```



https://youtu.be/vRjJAWh6JPE?t=472

compose를 활용하면 코드에 미리보기를 추가할 수 있습니다. @Preview annotation을 달면 Android Studio 화면에서 확인할 수 있습니다.

데모용 긴 글을 가져왔는데요, 좌우 간격이 좁아 padding을 써야하는데 text설정 할 때 padding을 추가했더니 미리보기의 padding도 반영되어서 나왔습니다. 에뮬레이터도, 설치도, 현재 작업화면으로 이동할 필요 없이 바로 코드 옆에서 뷰를 확인할 수있습니다.

```kotlin
@Preview
@Composable
fun PreviewMessage() {
		JetchatTheme {
				Message(message = demoMessages[0])
		}
}

@Composable
fun Message(
		message: Message
) {
		Text(text = message.content, Modifier.padding(8.dp))
}
```

https://youtu.be/vRjJAWh6JPE?t=526

리스트를 나타내기 위해서는 지금까지 RecyclerView를 사용 했습니다. 그리고 Adapter, ViewHolder, DiffUtils를 같이 사용해야 했습니다. 하지만 Compose에서는 LazyColumn 컴포넌트를 사용해도 됩니다. 메시지 목록을 items 메소드로 넘기면 목록에서 각 항목의 내용을 정의하게 됩니다.

```kotlin
@Composable
fun Messages(
		messages: List<Message>,
		modifier: Modifier = Modifier
) {
		LazyColumn(modifier) {
				items(messages) { message ->
						MessageItem(message = message, Modifier.fillParentMaxWidth())
				}
		}
}
```



https://youtu.be/vRjJAWh6JPE?t=576

그 다음은 애니메이션 입니다. AnimatiedVisibility 컴포저블을 활용해서 필요한 애니메이션을 만들었습니다. 바로 옆 프리뷰 화면에서 애니메이션 작동을 확인할 수 있습니다. Animator 객체는 필요 없이 모든 작업이 자동화 되어 있습니다.

https://youtu.be/vRjJAWh6JPE?t=659

기존에는 이미 컴파일된 xml리소스를 기반으로 테마가 설정 되었기 때문에 동적인 테마 업데이트가 어려웠습니다. 하지만 compose에서는 테마 업데이트도 동적으로 바뀌었습니다. 또한 테마가 바뀌는 것을 애니메이션 화 시킬 수 있습니다.



그리고 위에서 시연했던 코드들은 샘플앱에서 확인 가능합니다. (https://github.com/android/compose-samples)



## Q&A Session

https://youtu.be/vRjJAWh6JPE?t=732

그 다음은 Q&A 세션 입니다. Q&A 세션에서 나온 질문과 답변을 정리해 보았습니다. 



질문 1. Compose의 렌더링 성능은 기존 Views와 비슷한 수준에 도달했나요? 현재 뷰 성능은 어떤가요?

- 전반적으로 Compose의 성능은 View보다 우수
- Composition과 업데이트 프로세스를 분리하고 간소화하여 성능을 향상
- 실험적인 멀티 쓰레딩 API
  
  - Composition을 메인스레드에서 분리. 여러 쓰레드에서 동시에 진행 가능
- LazyList와 스크롤 성능은 아직 연구 중
- 툴킷을 플랫폼으로 부터 분리
  - 플랫폼과 분리되어 있어 성능을 높이는 작업 가능
  - 툴킷이 플랫폼 외부에 있다는 문제 때문에 런타임에서 더 많은 클래스를 로드해야 하는 문제
    - 해당 사항을 개선하기 위한 노력을 아끼지 않고 있다.

- 아직 할 일은 많지만 전망이 좋다.

  

질문 2. XML레이아웃이 있는 기존 프로젝트에 적합한가요?

- 개발자들이 갑자기 모든 앱코드를 Compose로 바꾸지는 않을것
- xml 레이아웃이 있는 기존 프로젝트를 유지할 수 있음
- 다만 compose가 제공하는 UI에 xml 레이아웃은 부적절함
  - compose로 움직이는. UI와 xml 레이아웃 간에는 상호 운용이 안됨
- string, color, dimen 등 다른 xml리소스는 가져올 수 있음.



질문 3. Compose 개발도구에 대한 업데이트가 있나요?

- (행사 당일) Compose Beta가 출시 됨
- [Android Studio의 최신 Canary 버전인 Arctic Fox](https://developer.android.com/studio/preview)에 Compose를 위한 도구가 여럿 포함됨.
- 레이아웃 편집기에서 Compose를 지원, Compose를 위한 미리보기 창 추가.
  - 미리보기를 통한 빠른 확인으로 개발 속도 향상 가능
  - 라이브 기능으로 프리뷰와 기기에서 변경 사항을 확인 할 수 있음.
- 앱 전체를 다시 컴파일 하지 않아도 즉시 확인이 가능해 개선 속도가 대폭 향상.
- ConstraintLayout 지원
  - 코드를 기반으로 작성. Compose 에 어울리는 방식 사용.



질문 4. Redux와 같은 상태 관리 개념/라이브러리가 제공되나요?

- 기존 솔루션에서 Compose를 사용하는게 너무 어려워서는 안된다고 생각한다.
- 최근 Android 생태계는 MVI(Model - View - Intent) 구조로 움직임
  - MVI 패턴은 Compose와 잘 맞음.
- Redux를 예로 들자면 Redux와 MVI는 원리적으로 비슷함.
- Compose는 개발자의 선택을 특정 방향으로 강요하지 않으므로 외부에서 만들어진 Redux같은걸 가져올 수있음



질문 5. Compose를 시작하기 위한 튜토리얼이 있나요?

- Compose Pathway라고 검색하면 여러 학습 자료를 찾아볼 수 있다.
  - https://developer.android.com/courses/pathways/compose
- Compose 베타를 공개하면서 새로운 문서, 샘플, 코드 실습들도 준비되어 있다.



질문 6. MotionLayout과 어떻게 호환되나요?

- MotionLayout은 Compose와 호환되지 않음
- Compose는 기존 View 시스템과는 애니메이션 사용 방법이 다르기 때문
- Compose는 애니메이션을 완전히 새롭게 바꾸었다.
- 다만 어떻게 통합하면 좋을지 방법을 찾고는 있다.



질문 7. 프로덕션에 배포할 수 있나요?

- 베타가 출시되었지만 베타는 안정적이지 않다.
- 아직 해야할 작업들이 남아있다.
- 올해 중에 안정화 단계에 이를 때 까지는 많은 작업이 필요하다.
- Compose를 프로덕션에 배포할 수 있다.
- 빠른 적용에는 장단점이 있으니 그 점을 충분히 알고 미리 준비하기를 권한다.



질문 8. Compose가 액티비티와 프래그먼트를 대체할 수 있나요?

- 대체 가능하다.
- 액티비티와 프래그먼트는 플랫폼에 속해 있으므로 교체할 필요 없다.
- Compose에서 앱 코드를 작성하다 보면 액티비티나 프래그먼트 간의 상호작용은 현저히 줄어든다.
  - 특히 프래그먼트와의 상호작용이 줄 것이다.
- Compose는 프래그먼트이든 어디든 Compose 계층을 만들어 내도록 설계 되어 있다.
- 처음부터 개발해야 한다면 액티비티나 프래그먼트를 그렇게 많이 사용하지는 않을 것이다.



질문 9. Jetpack Compose는 재미있나요?

- Composable 자체가 재미 있다.



## Android Developer Challenge

https://youtu.be/vRjJAWh6JPE?t=1613

Jetpack compose의 기능을 활용한 [Android Developer Challenge가 진행중에 있습니다.](https://developer.android.com/dev-challenge) 

총 4주간의 Challenge인데요, 이미 글을 작성한 시점(2021.3.10.) 기준으로는 2주차가 지나갔습니다.

방식을 보니 먼저 완성한 몇명에게 선물을 주는 방식으로 보입니다.

https://github.com/android/android-dev-challenge-compose 에서 기본적인 틀을 포크해서 README에 있는 사항을 따르면 된다고 합니다. 다음 주차때는 한번 도전을 해봐야 겠습니다.



## Interviews

https://youtu.be/vRjJAWh6JPE?t=1797

안드로이드 개발자의 인터뷰 영상이 나옵니다. 개발하는 환경에 대한 기대치가 높아지고 개발자의 작업 효율은 더 중요해졌다고 합니다. 그래서 Jetpack Compose 베타를 출시하고 Android 12 프리뷰 버전도 출시했다고 합니다. 그리고 Jetpack Compose를 발판삼아 더 나은 앱을 개발하길 기대한다고 합니다. 그리고 Android 12에서는 기기 호환성을 개선하고 성능을 향상하기 위한 새로운 기능을 추가했다고 합니다.



그 다음으로는 Android 엔지니어링 부문 부사장과의 인터뷰가 있었습니다.

왜 새로운 UI 툴킷을 출시했냐는 질문에 기존 툴킷은 출시된 지 10년이 되었으며 여러번 고쳐서 다시 써오면서 발전했지만, 처음 부터 다시 개발한다면 어떤 툴킷을 만들까 생각하면서 개발되었다고 합니다.

Compose는 선언적이고 간결하며 Compose를 몰라도 금방 내용을 파악할 수 있다고 합니다.

Jetpack Compose는 매우 유연한 툴킷이며 10년 뒤를 내다본 제품입니다. 휴대전화 뿐만 아니라 다른 디바이스에서도 활용될 것 이라고 하였습니다. 

Android 12에 관해서는 현재는 잘돌아가는 정도로만 프리뷰를 출시 했으며 앞으로 여러번의 릴리즈와 업데이트가 남아있다고 합니다. 베타 1이 되어야 본격적인 변화가 생기며 그 때가 기대된다고 하였습니다.

물론 첫번째 프리뷰에서 새로운 기능들이 추가 되었습니다. 복사 붙여넣기 강화라던지 영상을 공유했는데 상대방 앱이 코덱을 지원하지 않는다면 최신 코덱을 지원하지 않는 앱들에게도 보일 수 있도록 동적으로 변환하는 기능을 추가했다고 합니다.

또한 롤러블, 폴더블 디스플레이가 중요할 것이라고 의견을 내었습니다. 안드로이드의 역할은 이런 동적인 화면을 유용하게 활용하고 연속성을 유지할 수 있게 하는 것 이라고 하였습니다. 그리고 그렇게 하는 개발자의 작업을 도우려면 어떤 도구를 제공해야 할지  고민 한다고 하였습니다. 

Jetpack Compose가 좋은예라고 하면서 다양한 화면 크기와 동적인 화면 크기를 가정하고 개발하였다고 합니다. 코로나 시국에 화상회의/교육을 위해서 태블릿 구매가 많이 늘어났다고 합니다. 그런 점에서 대형화면이나 동적인 화면을 지원하는 도구를 더 많이 개발할 예정이라고 합니다.



## Outro

Jetpack Compose beta를 소개한 TheAndroidShow를 살펴 보았습니다. 개발자가 코드에만 집중할 수 있고, 더 간결하고 직관적으로 그리고 빠르게 개발할 수 있게하는 좋은 도구라고 말할 수 있습니다. 그리고 사용하는 것이 많이 기대가 됩니다. 선언형 프로그래밍으로써 코드 읽기, 유지보수의 편의만을 제공하는 것이 아니라, 부사장 인터뷰 처럼 요즘 코로나 시국에 늘어나는 태블릿 구매 그리고 갤럭시 폴드로 대표되는 접는 스마트폰, 롤러블폰과 같은 동적인 화면 크기를 가지는 기기 등 최근의 트렌드를 생각하면서 개발할 수 있도록 도와줄 것이라는 점이 인상 깊었습니다. 한 작업이 더 쉬워졌으니 가서 더 많은 것을 해보라 라는 뜻으로도 읽혀집니다.



# Reference

https://www.youtube.com/watch?v=vRjJAWh6JPE