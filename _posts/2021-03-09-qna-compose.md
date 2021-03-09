---

date: 2021-03-07
title: "Q&A of Jetpack Compose Beta"
categories: Android

---

# Q&A of  Jetpack Compose Beta



얼마전에 Android Jetpack Compose의 베타가 발표 되었습니다. 

거기에 맞춰 구글에서 라이브쇼를 열었는데요 https://www.youtube.com/watch?v=vRjJAWh6JPE

이번에 베타가 발표된 Jetpack Compose의 시연은 물론이고 실시간으로 사용자들의 질문을 받아 Q&A를 하는 시간도 있었습니다.

시연과 Q&A를 확인해보니 기존에 복잡했던 안드로이드 UI 작성과 확인이 더 편리해질 것으로 기대가 됩니다.

이 포스트 에서는 Q&A 세션에서 나온 질문과 답변을 정리해 보았습니다. 



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

