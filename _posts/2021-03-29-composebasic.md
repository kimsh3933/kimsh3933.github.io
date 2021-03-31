---

date: 2021-03-29
title: "Jetpack Compose Basics"
categories: Android

---

# Jetpack Compose Basics

원문 : https://developer.android.com/codelabs/jetpack-compose-basics



Jetpack Compose는 UI개발을 단순화하도록 설계된 최신 도구 키트 입니다. 반응형 프로그래밍 모델과 Kotlin 프로그래밍 언어의 간결함 및 사용 용이성이 합쳐 졌습니다. 선언적입니다. 데이터를 UI계층 구조로 변환하는 일련의 함수를 호출하여 UI를 표현합니다. 기본 데이터가 변경 되면 프레임 워크가 이러한 함수를 자동으로 호출하여 뷰 계층을 업데이트 합니다.

Compose앱은 Composable Function으로 구성됩니다. `@Composable` annotation으로 표시된 일반 함수로 더 많은 composable function을 불러올 수 있습니다. Compose를 사용하면 코드를 작은 덩어리로 구성할 수 있습니다.

재사용 가능한 작은 composable를 만들어 앱에 사용되는. UI 라이브러리를 쉽게 구축할 수 있습니다. 각각은 화면의 한 부분을 담당하며 독립적으로 편집 가능합니다.



## Composable 시작해보기

Composable Function은 `@Composable` annotation이 달린 "일반 함수"입니다. 내부에 있는 다른 `@Composable`을 호출할 수 있습니다.



```kotlin
@Composable
fun Greeting(name: String) {
   Text(text = "Hello $name!")
}
```

이 메소드는 주어진 String을 표시하는 UI를 생성합니다. Text는 라이브러리에서 제공하는 기능 입니다.



기존에는 XML 레이아웃을 호출하여서 UI를 작업했지만 Composable를 사용하는 경우 activity안에서 composable function을 사용합니다.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            BasicsCodelabTheme {
                // A surface container using the 'background' color from the theme
                Surface(color = MaterialTheme.colors.background) {
                    Greeting("Android")
                }
            }
        }
    }
}
```

여기서 BasicCodelabTheme는 Composable function의 스타일을 지정하는 방법입니다. 텍스트가 화면에 어떻게 표시되는지 확인 하려면 에뮬레이터 또는 기기에서 앱을 실행하거나 Android Studio 미리보기에서 볼 수 있습니다.

setContent는 activity의 view를 생성하는 부분입니다. setContent 는 content라는 function을 parameter로 받고 있습니다. @Composable은 다른 @Composable에서만 호출이 가능합니다.([Composable](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Composable#) functions can only ever be called from within another [Composable](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Composable#) function.) [참고](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Composable) 

그렇기 때문에 @Composable인 content를 parameter로 받는 setContent 와 연결되어야지만 Composable들을 실행시킬 수 있습니다.

```kotlin
public fun ComponentActivity.setContent(
    parent: CompositionContext? = null,
    content: @Composable () -> Unit
)
```



Android Studio의 미리보기를 사용하려면 `@Preview` annotation을 사용하여 parameter가 없는 함수 혹은 기본 파라미터가 있는 함수를 @Preview 와 함께 사용하여 프로젝트를 빌드 합니다. 동일한 파일에 여러개의 preview를 포함하고 이름을 지정할 수 있습니다. Preview 함수는 실제 실행되는 앱과는 별도로 동작 됩니다. 기존 xml에서 tools와 기능이 유사합니다.

```kotlin
@Preview(showBackground = true, name = "Text preview")
@Composable
fun DefaultPreview() {
    BasicsCodelabTheme {
        Greeting(name = "Android")
    }
}
```

Preview annotation에 name값을 지정해 주어야 Preview를 볼 수 있습니다.
미리보기가 보이지 않으면  ![bcf00530a220eea9.png](https://raw.githubusercontent.com/kimsh3933/kimsh3933.github.io/master/_posts/attach/2021-03-29-composebasic/code.png) (code)가 아닌  ![aadde7eea0921d0f.png](https://raw.githubusercontent.com/kimsh3933/kimsh3933.github.io/master/_posts/attach/2021-03-29-composebasic/split.png) (split)을 클릭하면 됩니다.


Preview코드 작성 후 preview화면에서 Build&Refresh를 클릭하여 빌드 합니다. value가 바뀔때는 실시간으로 반영되지만 UI 속성이 바뀔 때에는 새로 빌드가 필요합니다.



## 선언적 UI

다른 배경색을 설정하기 위해서는 Surface에 색을 정의 하면 됩니다.

```kotlin
@Composable
fun Greeting(name: String) {
    Surface(color = Color.Yellow) { // color에는 Color 객체를 넣어야 합니다. Int값이 아닙니다.
        Text (text = "Hello $name!")
    }
}
```



대부분 Compose UI 요소는 선택적으로 속성을 넣을 수 있습니다. Modifier객체를 활용하면 됩니다. Modifier객체는 변수로 만들어서 재사용이 가능합니다. Padding을 사용한 예시 입니다. 양 옆으로 24dp만큼 Padding을 줍시다.

```kotlin
@Composable
fun Greeting(name: String) {
    Surface(color = Color.Yellow) { // color에는 Color 객체를 넣어야 합니다. Int값이 아닙니다.
        Text (text = "Hello $name!", modifier = Modifier.padding(24.dp))
    }
}
```



### 재사용 가능한 Composable

UI에 더 많은 구성 요소를 추가할수록 코드 베이스의 다른 함수와 마찬가지로 더 많은 수준의 중첩을 만들 수 있습니다. 함수가 커지면 가독성에 영향을 끼칠 수 있습니다. 재사용 가능한 작은 구성요소를 만들어 앱에서 사용하는 UI요소를 쉽게 구축할 수 있습니다.



Composable한 `MyApp`이라는 함수를 만듭니다. 

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp()
        }
    }
}

@Composable
fun MyApp() {
    BasicsCodelabTheme {
        Surface(color = Color.Yellow) {
            Greeting(name = "Android")
        }
    }
}

@Composable
fun Greeting(name: String) {
    Text(text = "Hello $name!", modifier = Modifier.padding(24.dp))
}

@Preview
@Composable
fun DefaultPreview() {
    MyApp()
}
```



다른 composable 함수도 BasicCodelabTheme, Surface를 적용받도록 하고 싶은데요, 그러면 MyApp함수를 다른 곳에서 활용 해야 할 것입니다. 그렇지만 이미 Greeting 함수가 안에 있어서 활용하기 곤란한 상황인데요, Greeting함수 대신 원하는 함수를 parameter로 가져오게 해 봅시다. Composable한 High-Order 함수를 활용해 봅시다. (컨테이너 함수)

```kotlin
@Composable
fun MyApp(content: @Composable () -> Unit) {
    BasicsCodelabTheme {
        Surface(color = Color.Yellow) {
            content()
        }
    }
}
```

High-Order function의 앞에 @Composable annotation을 넣으면 composable한 함수를 파라미터로 넣을 수 있습니다. 이를 container function이라고 합니다. 컨테이너 함수인 MyApp을 다른 곳에서도 사용하면 해당 UI요소도 BasicCodelabTheme, Surface속성을 적용 받을 수 있습니다.



```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp {
                Greeting("Android")
            }
        }
    }
}

@Preview("Text preview")
@Composable
fun DefaultPreview() {
    MyApp {
        Greeting("Android")
    }
}
```



### 레이아웃으로 여러 Composable 호출하기

```kotlin
@Composable
fun MyScreenContent() {
    Column {
        Greeting("Android")
        Divider(color = Color.Black)
        Greeting("there")
    }
}
```



Column은 Vertical LinearLayout와 유사 합니다. 항목을 수직으로 표시할 수 있습니다.

Divider는 가로 구분선을 제공합니다.



Compose는 Kotlin의 다른 함수들 처럼 호출 할 수 있습니다. 예를 들면 Column에 list를 가져와서 for loop를 사용하여 UI를 나타낼 수 있습니다.



```kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
    Column {
        for (name in names) {
            Greeting(name = name)
            Divider(color = Color.Black)
        }
    }
}
```



## Compose의 State

상태 변경에 대한 반응은 Compose의 핵심 입니다. Compose 앱은 Composable 함수를 호출하여 UI로 변환합니다. 데이터가 변경 되면 새 데이터로 이러한 함수를 호출하여 업데이트 된 UI를 만듭니다. Compose는 앱 데이터의 변경 사항을 관찰할 수 있는 도구를 제공합니다. 이 도구는 자동으로 함수를 호출합니다. 이를 recomposing이라고 합니다. compose는 또한 데이터가 변경된 구성 요소만 재구성 하고 영향 받지 않는건 건너 뛸 수 있도록 합니다.

mutableStateOf 라는 Composable한 가변 메모리를 제공하는 함수를 사용하여 내부 상태를 추가합니다.

remember을 활용하여 현재 상태를 기억하게 합니다. 그리고 해당 값을 사용할 때 자동으로 observe됩니다. 만약 count값이 변경이 생기면 해당 범위가 한번 더 호출되는 식 입니다. 값이 observe 되어 한번 더 호출되는 범위는 해당 state의 value를 호출한 Composable 함수 입니다. 최소로만 재구성을 하게 하려면 state의 value보다는 state를 다른 composable함수로 넘겨서 최종적으로 처리하는 곳에서 value를 쓰도록 하면 됩니다. 

```kotlin
@Composable
fun Counter() {

    val count = remember { mutableStateOf(0) }

    Button(onClick = { count.value++ }) {
				// 여기는 @Composable한 RowScope라 Counter 함수 전체가 호출되지 않고 이 부분만 다시 호출 됩니다. 
        Text("I've been clicked ${count.value} times")
    }
}
```

```kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
    Column {
        for (name in names) {
            Greeting(name = name)
            Divider(color = Color.Black)
        }
        Divider(color = Color.Transparent, thickness = 32.dp)
        Counter()
    }
}
```







```kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
    val counterState = remember { mutableStateOf(0) }

    Column {
        for (name in names) {
            Greeting(name = name)
            Divider(color = Color.Black)
        }
        Divider(color = Color.Transparent, thickness = 32.dp)
        Counter(
            count = counterState.value,
            updateCount = { newCount ->
                counterState.value = newCount
            }
        )
    }
}

@Composable
fun Counter(count: Int, updateCount: (Int) -> Unit) {
    Button(onClick = { updateCount(count+1) }) {
        Text("I've been clicked $count times")
    }
}
```

호출한 함수에 의해 내부 상태를 제어하게 하는 방법을 hoisting이라고 합니다. (자바스크립트 에서는 선언을 유효범위의 최상단에서 한번에 다하는 방법이라고 합니다.) 

MyScreenContent에서 Counter함수를 호출하며 관련된 상태, 값들을 MyScreenContent가 가지고 있는 것이죠. 그렇게 되면 다른 카운트, 다른 방식으로 새로운 카운트를 조작하는 등 counter 내부에 state를 가지고 있을 때 보다 재사용성이 높아 졌습니다.



## 유연한 레이아웃

Column은 항목을 수직 순서대로 배치하는데 사용됩니다.

같은 방식으로 Row는 가로로 배치하는데 사용됩니다. 특정 가중치로 화면을 차지하도록 weight 속성을 사용할 수도 있습니다.

```kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
    val counterState = remember { mutableStateOf(0) }

    Column(modifier = Modifier.fillMaxHeight()) {
        Column(modifier = Modifier.weight(1f)) {
            for (name in names) {
                Greeting(name = name)
                Divider(color = Color.Black)
            }
        }
        Counter(
            count = counterState.value,
            updateCount = { newCount ->
                counterState.value = newCount
            }
        )
    }
}
```

상위 Column에 fillMaxHeight 속성을 넣었습니다. 해당 속성은 컨텐츠가 들어갈 수 있는 최대 높이를 잡으라는 의미 입니다.

하위 Column에는 weight를 넣었습니다. 



또한 if문을 가지고 count값에 따라서 배경색을 변경할 수도 있습니다.

```kotlin
@Composable
fun Counter(count: Int, updateCount: (Int) -> Unit) {
    Button(
        onClick = { updateCount(count+1) },
        colors = ButtonDefaults.buttonColors(
            backgroundColor = if (count > 5) Color.Green else Color.White
        )
    ) {
        Text("I've been clicked $count times")
    }
}
```



위 예시에서는 두개의 항목을 한 Column에 표시 했지만, 수천개를 처리한다면 어떻게 해야 할까요? 우선 크게 두 가지 방법이 있습니다.

- ScrollableColumn 은 수직 스크롤할 수 있는 Column입니다. ScrollView와 유사합니다.

- LazyColumn은 보이는 뷰만 렌더링 합니다. 그렇게 때문에 큰 리스트가 있을 때 좋은 퍼포먼스를 발휘합니다. RecyclerView와 유사합니다.
  - 다만 ReclyerView와는 다르게 childView를 재활용 하지는 않습니다. 스크롤 할 때 새로운 Composable를 방출하며 이것은 Android View를 인스턴스화 하는 것 보다 비용이 덜 들기 때문에 성능이 좋습니다.

1000개 이상 리스트를 사용하려 하는데요, 여기서는 LazyColumn을 사용해봅시다.

```kotlin
@Composable
fun NameList(names: List<String>, modifier: Modifier = Modifier) {
   LazyColumn(modifier = modifier) {
       items(items = names) { name ->
           Greeting(name = name)
           Divider(color = Color.Black)
       }
   }
}
```

```kotlin
names: List<String> = List(1000) { "Hello Android #$it" }
```



## 목록 애니메이션

목록을 클릭 한 후 배경색을 변경하고 싶다고 가정해 봅시다. 버튼을 이용하여 바로 바뀌는건 해봤지만 이번에는 애니메이션을 넣어 자연스럽게 해봅시다.

우선 버튼에 isSelected라는 버튼 선택 상태를 기억하는 remember을 만들어 봅시다. 그리고 animateColorAsState를 사용하여 isSelected값에 따라 배경 값이 바뀌게 해 봅시다.

```kotlin
@Composable
fun Greeting(name: String) {
   var isSelected by remember { mutableStateOf(false) }
   val backgroundColor by animateColorAsState(if (isSelected) Color.Red else Color.Transparent)

   Text(
       text = "Hello $name!",
       modifier = Modifier
           .padding(24.dp)
           .background(color = backgroundColor)
           .clickable(onClick = { isSelected = !isSelected })
   )
}
```





## 앱 테마 지정

현재 Android Studio에서 empty compose activity 프로젝트를 만들면 BasicsCodelabTheme같은 테마 파일이 kotlin으로 작성되어 있습니다. xml대신 kotlin으로 작성된 것입니다.



해당 테마로 Composable Function을 감싸게 되면 해당 함수는 해당 테마의 영향을 받게 됩니다.

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            BasicsCodelabTheme {
                Greeting(name = "Android")
            }
        }
    }
}
```



또한 style속성에 MaterialTheme에 있는 속성을 불러낼 수 있습니다.

```kotlin
@Composable
fun Greeting(name: String) {
    Text (
        text = "Hello $name!",
        modifier = Modifier.padding(24.dp),
        style = MaterialTheme.typography.h1
    )
}
```



ui.theme 패키지에 Theme.kt, Color.kt 가 생성되어 있습니다. 이 코드들을 바탕으로 다른 테마들도 구현이 가능합니다.

##### color.kt

```kotlin
val Purple200 = Color(0xFFBB86FC)
val Purple500 = Color(0xFF6200EE)
val Purple700 = Color(0xFF3700B3)
val Teal200 = Color(0xFF03DAC5)
```



##### Theme.kt

```kotlin
private val DarkColorPalette = darkColors(
    primary = Purple200,
    primaryVariant = Purple700,
    secondary = Teal200
)

private val LightColorPalette = lightColors(
    primary = Purple500,
    primaryVariant = Purple700,
    secondary = Teal200

    /* Other default colors to override
    background = Color.White,
    surface = Color.White,
    onPrimary = Color.White,
    onSecondary = Color.Black,
    onBackground = Color.Black,
    onSurface = Color.Black,
    */
)

@Composable
fun BasicsCodelabTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable() () -> Unit
) {
    val colors = if (darkTheme) {
        DarkColorPalette
    } else {
        LightColorPalette
    }

    MaterialTheme(
        colors = colors,
        typography = Typography,
        shapes = Shapes,
        content = content
    )
}
```

