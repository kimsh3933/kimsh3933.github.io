---

date: 2021-03-29
title: "Jetpack Compose With ViewModel"
categories: Android

---

## Composable에서 ViewModel 사용법

원문 : https://developer.android.com/jetpack/compose/libraries#viewmodel

MVVM 구조인 경우 Compose에서 viewmodel을 어떻게 사용하면 될까요?

먼저 각 composable 함수에 viewmodel을 초기화 할 수 있습니다.

이를 위해 먼저 `androidx.lifecycle:lifecycle-viewmodel-compose:$latestVersion` 를 gradle.app에  추가하도록 합시다. [참고](https://developer.android.com/jetpack/androidx/releases/lifecycle#lifecycle_viewmodel_compose_2)

```kotlin
@Composable
fun sampleFunction() {
	val viewModel: ExamleViewModel = viewModel()
}
```

@Composable 함수 내부에서 호출이 가능한 viewModel() 함수를 가지고 viewmodel을 가져올 수 있습니다. Composable이 활동하는 한 어디에서 가져오든 동일한 instance의 viewmodel을 가져 옵니다.



그리고 viewmodel의 liveData를 observe해서 데이터를 얻었었는데 이를 compose에 적용 시켜 봅시다.

liveData를 composable에서 사용할 수 있는 state로 변환하여 가져와 봅시다. 그러기 위해서는 먼저 `androidx.compose.runtime:runtime-livedata:$composeVersion` 를 gradle.app 에 추가해 봅시다.

그러고 나면 observeAsState 함수를 가지고 liveData의 값을 compose에 사용 할 수 있는 state로 가져올 수 있습니다. observeAsState에는 parameter로 초기값을 지정할 수 있습니다. 초기값을 지정하지 않는다면 별도의 null처리가 필요합니다.



```kotlin
@Composable
fun textFromApi() {
    val viewModel: TempViewModel = viewModel()
      // exampleState.value의 null 처리 필요. observeAsState의 parameter 로 초기값 지정 가능.
    val exampleState = viewModel.liveDataSample.observeAsState()

    showValue(exampleState)

    viewModel.getValueFromApi()
}

@Composable
fun showValue(name: State<String?>) {
    Text(text = "${name.value}")
}
```



liveDataSample이라는 liveData를 State로 가져왔는데요, textFromApi메소드에서 exampleState.value를 호출하게 되면 textFromApi 전체를 다시 호출하게 되어 Text만 새로 고침 하면 되는 것을 다른 로직까지 새로 호출시켜서 낭비를 가져올 수 있습니다. 그래서 텍스트 표시하는 부분을 별도의 Composable로 분리하여 필요한 부분만 갱신 되도록 해야 합니다.



