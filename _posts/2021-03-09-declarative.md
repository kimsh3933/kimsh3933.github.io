---

date: 2021-03-09
title: "선언형 프로그래밍은 무엇인가?"
categories: Android

---



[최근 Jetpack Compose의 beta가 출시 되었습니다.](https://www.youtube.com/watch?v=9US35jaiWIw) 코틀린 코드 만으로도 UI를 작성하여 코드 관리가 쉬워지고 성능이 더 좋아질 것으로 기대되는 툴킷 입니다. 지금 Compose에 대해서 알아보고 있는 중이고요, 알아보고 정리해서 업로드 하도록 하겠습니다. 그 전에 Compose를 살펴보면 Declarative 라는 단어가 나옵니다. Declare 선언하다, 알리다 라는 뜻이란걸 보아 선언적이라는 말이 되겠네요. 선언적이라는 말이 나왔다면, 이전에는 안드로이드 UI를 선언적으로 작성하지 않았다는 말이 될거 같네요. 그렇다면 선언형은 무엇인지 이전 방식과 비교해보면서 간단히 알아보았으면 합니다.



# 선언형 프로그래밍이란?

선언형(Declarative) 프로그래밍은 말그대로 선언하는 프로그래밍 입니다. 그리고 그 반댓말은 명령형(Imperative) 프로그래밍 입니다.

예시를 한번 볼까요?

>선언형 손님 : 스테이크 하나 주세요!
>
>명령형 손님 : 호주산 살치살을 70도 온도에 4시간 동안 수비드를 하고 나서 팬 위에서 구워주세요.

선언형 손님은 "스테이크" 자체를 원합니다. 반면에 명령형 손님은 호주산 살치살을 어떻게 가공하느냐에 더 집중하고 있습니다.

이렇게 선언형은 "무엇을" 할 것인가, 명령형은 "어떻게" 할 것인가에 더 집중하고 있습니다.

위키피디아에서는 `간단히 말하여, 명령형 프로그램은 알고리즘을 명시하고 목표는 명시하지 않는데 반해 선얺여 프로그램은 목표를 명시하고 알고리즘을 명시하지 않는 것이다.` 라고 설명하고 있습니다.



# 선언형 프로그래밍의 예시

이미 이전에 iOS에서 나왔던 SwiftUI가 선언형 프레임워크 입니다.

안드로이드를 기준으로 설명 드리자면 [DataBinding](https://developer.android.com/topic/libraries/data-binding)이 일종의 선언형 프로그램입니다. 

기존 방식을 보면 xml에서 정의한 뷰를 java/kotlin 코드에서 가져오기 위해서 findViewById 같은걸 해서 가져오고, 연산을 하고 뷰를 변화 시켰었습니다.

```kotlin
val textView = findViewById<TextView>(R.id.text)
val currentTime = parseToFormat(System.currentTimeMills())
textView.text = currentTime
```

textView의 텍스트를 변화 시키기 위해서 xml에서 텍스트 뷰를 찾아오고 연산을 하고 나서 결과를 text에 넣게 됩니다. 그러면 텍스트뷰를 변화시키는 부분은 이런 중간 알고리즘을 다 알게 되는거죠.



반면 dataBinding에서는 텍스트를 이렇게 변화 시킵니다.

```xml
<TextView
	android:text="@{viewModel.currentTime}"        
/>
```

텍스트 뷰를 변화시키는 레이아웃 xml파일에서는 viewModel.currentTime라는 값(라이브데이터)이 어떤 과정을 거쳐서 텍스트 뷰까지 왔는지 중요하지 않습니다. 그저 currentTime이라는 값만 달라고 할 뿐 입니다.

그리고 기존 java/kotlin 에서 캡슐화, ViewModel, 옵져버 패턴을 활용해서 선언형으로 작성 할 수도 있습니다.

내부 로직은 ViewModel에 숨겨두고 viewModel의 LiveData를 observe하여 값만 쏙 빼서 뷰 상태를 변화시킬 수 있습니다.

> 변수에 observer을 달아둔다 -> 연산을 요청한다 -> 캡슐화로 꽁꽁 숨겨진 곳에서 연산을 한다 -> 연산의 결과로 변수가 변한다 -> 뷰에 변수만 넣어주면 된다.

그리고 데이터 흐름에 따라 데이터를 처리하는 Rx계통의 라이브러리를 활용해서도 선언형 프로그래밍을 할 수 있습니다. subscribe하는 부분에서는 데이터가 어떻게 필터 되는지 알 수 없고 주는 데이터만 받아서 먹으면 되니깐요.

```java
Observable.just(timeStamp)
          .map(timeStamp->parseToFormat(timeStamp))
          .subScribe(result -> textView.setText(result));
```

subscribe 내부에 있는 메소드는 just나 map에서 어떤 일이 일어났는지 알 수 없이 result라는 결과만 받아서 처리를 하고 있습니다.

이런 방식들을 정확하는 반응형 프로그래밍이라고 하지만, 반응형 프로그래밍은 선언형 프로그래밍을 지향하고 있습니다.



혹시 언어 자체가 선언적인것은 있을까요?

SQL과 HTML 입니다. SQL은 쿼리문에 내가 무엇을 원하는지 적고 있습니다. 무엇을 원하는지 도출하기 위해서는 어떤 과정을 거쳤는지 알 필요가 없는거죠.

```sql
SELECT * FROM THE_TABLE WHERE COST > 100
```

COST가 100 이상인 column들을 찾기 위한 쿼리문인데요, 해당 결과를 찾는 알고리즘을 우리는 모릅니다.

HTML은 프로그래밍 언어... 는 아니지만 HTML만 있으면 완전히 선언형이 됩니다. 작성한 HTML을 어떻게 처리하는지 모르며, 그저 우리는 뭘 표현하고 싶다만 작성하면 됩니다. 그림을 보여 주고 싶다면 img태그, 테이블을 보여주고 싶다면 table 태그 처럼 말이죠.



# 선언형의 이점

선언형 프로그래밍이 어떤건지 살펴 보았습니다. 그렇다면 왜 다들 선언형 프로그램을 쓰거나 선언형 프로그램을 지향하는 방식을 사용하고 있을까요?

우선 코드의 유지보수가 수월해 집니다. 선언형은 결과에만 집중하면 되기 때문에 결과를 처리하는 부분만 손을 보면 됩니다.

그리고 실제 알고리즘은 뒤에 감추어져 있고 이는 메소드 명이나 클래스 명으로 나와 있기 때문에 즉, 사람의 언어에 가깝게 작성 되어 있기 때문에 코드를 추론하기가 더 쉬워집니다. 쉽게 말하면 코드를 잘 읽을 수 있게 되는 것이죠. 코드를 잘 읽을 수 있다는건 문제를 고치기 쉬워지고, 뭔가를 더하기 쉬워진다는 것입니다. 

또한 바꾸기로 한 무엇만 받으면 되고 중간에 뭐가 일어났는지 알지 않아도 되기 때문에 전체적인 애플리케이션의 흐름을 바꾸지 않고도 바꾸기로 한 무엇을 수정할 수 있습니다.



# Compose 와 선언형?

지금까지 선언형은 무엇이고 이점은 무엇인지에 대해서 알아 보았습니다. 앞에서 jetpack compose를 언급했었는데요, compose에는 선언형과 연관된 어떠한 이점이 있길래 compose에 관심을 가질까요? 자세한 것은 이후 포스트에서 다루겠지만 간단하게 짚고 넘어가 봅시다.

텍스트 뷰 하나에 "Hello World"를 입력해 봅시다

```kotlin
fun onCreate(saveStateInstance: Bundle?) {
	super.onCreate(saveStateInstance)
	setContentView(R.layout.main)
	val textView = findViewById<TextView>(R.id.text)
	textView.text = "Hello World"
}
```

텍스트 뷰 하나의 값을 바꾸기 위해 xml을 설정하고 xml에서 텍스트 뷰를 찾아야 합니다. 텍스트에 그저 Hello World를 넣고 싶을뿐인데 말이죠.



Compose에서는 이렇게 처리 합니다.

```kotlin
fun onCreate(saveStateInstance: Bundle?) {
	super.onCreate(saveStateInstance)
	setContent {
    helloWorld()
  }
}

@Composable
fun helloWorld() {
  Text("Hello World")
}
```

helloWorld 메소드 안에서 아무것도 할 필요 없이 Hello World라는 문자열을 가진 텍스트 뷰를 선언 합니다.

이렇게 보시면 위의 코드 보다 아래에 있는 코드가 정확히 뭘 이야기 하는지 알기 쉽습니다.



# 결론

단순한 작업을 하더라도 선언형으로 작업을 하면 나중에 코드 읽기가 더 수월해 지는 것 같습니다. 그렇기 때문에 지금까지 선언형을 추구하면서 개발을 해왔습니다. LiveData, 디자인패턴 같은 것을 사용해왔다면 이런 것들이 선언형, 반응형으로 동작하며 더 보기 좋은 코드, 유지보수하기 좋은 코드를 만들기 위해서 사용해왔다는 것을 아실겁니다.(대다수의 새로운 아키텍쳐들은 더 보기 좋은 코드를 지향하면서 만들어 지는 것 같습니다.)  과연 Compose에서는 어떤 방식으로 보기 좋은 코드를 만들 수 있을지 어떻게 활용하면 될지 기대가 많이 됩니다.



# Reference

- https://boxfoxs.tistory.com/430
- https://developer.android.com/jetpack/compose/mental-model

- [https://medium.com/@hongseongho/선언형-프로그래밍-알아보기-1d8247342f17](https://medium.com/@hongseongho/선언형-프로그래밍-알아보기-1d8247342f17)
- [https://ko.wikipedia.org/wiki/선언형_프로그래밍](https://ko.wikipedia.org/wiki/선언형_프로그래밍)
- https://developer.android.com/topic/libraries/data-binding

- https://brunch.co.kr/@yudong/35
- https://developer.android.com/jetpack/compose/tutorial