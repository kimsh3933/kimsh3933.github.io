---

date: 2021-06-20
title: "Google IO 2021 : Using Jetpack Libraries In Compose"
categories: Android

---

Using Jetpack Libraries In Compose



해당 게시글은 Google IO 2021의 세션 중 "Using Jetpack Libraries In Compose"의 내용을 정리한 글 입니다.

https://www.youtube.com/watch?v=0z_dwBGQQWQ



읽어 보기 전에 What's new in Jetpack Compose를 먼저 읽어 보시는 것을 권장합니다.

새로운 UI 도구 키트인 Jetpack Compose를 사용하는데 가장 중요한 부분 중 하나는 처음부터 앱을 다시 작성하지 않고도 기존 앱을 활용하여 점진적으로 도입할 수 있어야 하는 것입니다. 다르게 말하자면 UI 레이어 아래에 깔린 비즈니스 및 데이터 레이어를 새로운 Compose UI와 함께 사용할 수 있다는 것 입니다.



![001](https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-07-05-usingjetpack/001.png?raw=true)

가정 및 원예 쇼핑 앱인 Bloom이라는 앱을 예시로 들었습니다.(하단 설명에 보면 mock앱, 가상의 앱이라고 되어 있습니다.) 

이 앱은 사용자에게 방대한 양의 식물을 검색하고 사전 구축한 동류 식물 컬렉션을 살펴보는 기능을 제공합니다.

여기서는 Bloom 앱을 뷰와 XML을 사용하는 대신에 Compose로 화면을 작성하는 방법에 대해서 알아 봅니다.



![002](https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-07-05-usingjetpack/002.png?raw=true)

Bloom은 여러 Jetpack 라이브러를 활용 하였습니다. 앱의 아키텍처를 살펴보면 Bloom은 Room을 사용하여 식물 정보를 데이터베이스에 저장합니다. 

메모리 효율적인 청크로 로드하기 위한 페이징과 UI로직을 처리하기 위한 ViewModels, Kotlin Coroutine Flow를 통해 각 레이어와 Hilt간의 상태 및 데이터를 의존성 주입 솔루션으로 노출합니다.

```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
	private val plantsRepository: PlantRepository
) : ViewModel() {
  private val _uiState = MutableStateFlow(HomeUiState(loading = true))
  val uiState: StateFlow<HomeUiState> = _uiState
  
  val pagedPlants: Flow<PagingData<Plant>> = plantsRepository.plants
  
  init {
    viewModelScope.launch {
      val collections = plantsRepository.getCollections()
      _uiState.value = HomeUiState(plantCollections = collections)
    }
  }
}
```

Bloom의 HomeViewModel은 두 가지 용도로 사용됩니다.

첫째로 UI에 노출되는 상태와 해당 상태가 생성되는 방식을 구분합니다. 홈 UI의 비즈니스 로직의 처리를 전담하는 것이 바로 ViewModel입니다.

둘째, Jetpack ViewModel 클래스를 확장하면서 구성이 변경되어도 불구하고 살아남아 최신 데이터를 즉시 사용할 수 있게 합니다. 호출된 식물 목록은 저장소 레이어에서 노출 됩니다.

```kotlin
data class HomeUiState(
	val plantCollections: List<Collection<Plant>> = emptyList(),
  val loading: Boolean = false,
  val refreshError: Boolean = false
)
```

화면의 나머지 UI 상태는 동류 식물 컬렉션 및 페이지가 로드중이거나 오류가 표시되어야 하는 경우에 뜨는 신호로 구성되어 있습니다.

![003](https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-07-05-usingjetpack/003.png?raw=true)

HomeScreen을 보면 UI의 각 부분에 대한 여러개의 개별 컴포저블로 나눌 수 있으며, 각 컴포저블은 필요한 데이터만 담을 수 있습니다.



```kotlin
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun HomeScreen() {
	val homeViewModel = viewModel<HomeViewModel>()
}
```



HomeScreen 구현을 하게 되면 viewModel() 메소드를 사용하여 컴포저블에서 HomeViewModel의 인스턴스를 얻을 수 있습니다.

이 메서드는 두가지 작업을 수행합니다.

첫째로 ViewModel의 범위를 가장 가까운 ViewModelStoreOwner로 자동 지정 합니다. HomeScreen을 액티비티에 바로 입력하면 사용할 수 있습니다.

두번째로 Factory를 사용할 수 있습니다. 

AndroidEntryPoint Annotation이 달린 액티비티나 Fragment의 경우 Hilt가 이미 기본 팩토리로 되어 있어서 Hilt 기반 ViewModel은 즉시 작동합니다. Hilt나 다른 의존성 주입 솔루션을 사용하지 않는 경우 ViewModel Factory를 직접 만들어 viewModel 함수에 전달하게 됩니다.

Compose에서는 재사용 및 테스트 가능성을 높이기 위해 상태를 유지하지 않는 컴포저블을 선호합니다. 이를 Stateless 컴포저블이라고 부르며, 이는 내부 상태를 유지하는 Stateful과 반대 됩니다.

여기서 HomeScreen은 함수 내에서 HomeViewModel의 인스턴스를 검색하여 Stateful 컴포저블로 만들 수 있습니다. 

```kotlin
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun HomeScreen(
	homeViewModel: HomeViewModel = viewModel()
) {
	val uiState by homeViewModel.uiState.collectAsState()
  
  if (uiState.loading) {
    // ...
  } else {
    // ...
  }
}
```

Stateless로 만들려면 ViewModel을 Parameter로 넣어야 합니다. 이런식으로 ViewModel획득에 대한 책임을 상위 요소에 위임하여 컴포저블이 UI 작성에 중점을 두도록 합니다.

ViewModel의 인스턴스가 있다면 collectAsState 함수를 사용하여 uiState 플로우를 수집할 수 있습니다. 이는 상태를 읽은 컴포저블을 다시 실행하고 이에 따라 UI를 새로 업데이트 합니다.

이는 LiveData(observeAsState 사용)나 RxJava(subscribeAsState)로도 가능합니다.



```kotlin
import androidx.paging.compose.collectAsLazyPagingItems

@Composable
fun PlantList(plants: Flow<PagingData<Plant>>) {
  val pagedPlantItems = plants.collectAsLazyPagingItems()
  LazyColumn {
    itemsIndexed(pagedPlantItems) { index, plant ->
      if (plant != null) {
        PlantItem(plant)
      } else {
        PlantPlaceHolder()
      }
    }
  }
}
```

Bloom에는 방대한 식물 데이터가 있으므로 페이징을 활용하는 것이 좋습니다. Paging Compose 아티팩으로 PagingData의 플로우를 collectAsLazyPagingItems() 메소드를 통해 Compose에서 사용할 수 있는 상태로 변환합니다.

이렇게 하면 LazyColumn을 사용하여 쉽게 표현할 수 있습니다.

```kotlin
@Composable
fun PlantList(plants: Flow<PagingData<Plant>>) {
  val pagedPlantItems = plants.collectAsLazyPagingItems()
  LazyColumn {
    if (pagedPlantItems.loadState.refresh == LoadState.Loading) {
      item { LoadingIndicator() }
    }
    
    itemsIndexed(pagedPlantItems) { /* ... */ }
    
    if (pagedPlantItems.loadState.append == LoadState.Loading) {
      item { LoadingIndicator() }
    }
  }
}
```

또한 loadState를 보면 데이터가 새로고침한 시점과 더 많은 데이터가 목록에 추가 되는 동안 화면 하단에 다다른 시점 까지도 표시되는 자체 로딩 컴포져블을 쉽게 추가할 수 있습니다.

```kotlin
@Composable
fun HomeScreen(
	homeViewModel: HomeViewModel = viewModel()
) {
  // ...
  PlantList(homeViewModel.pagedPlants)
  // ...
}
```

이제 HomeScreen에서 ViewModel에서 가져온 데이터로 PlantList를 불러올 수 있습니다. 



ViewModel을 수정하거나 앱의 다른 레이어를 변경하지 않고도 Compose UI 에서 Jetpack 라이브러리를 많이 사용하고 있다고 합니다.



![004](https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-07-05-usingjetpack/004.png?raw=true)

앱의 디자인을 보면 다른 곳에서 재사용 하고 싶은 UI가 있을겁니다. 



![005](https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-07-05-usingjetpack/005.png?raw=true)

예시를 보면 PlantDetailScreen의 추천 사용으로 재사용 할 수 있는 것이 바로 Browse Collections Carousel입니다. 여기의 아이템을 탭하면 항목이 확장되어서 해당 컬렉션과 관련된 식물이 나타납니다. 캐러셀 구현 시 캐러셀 자체를 확장하거나 축소하는 로직과 내부 상태 뿐만 아니라 사용자가 식물을 탭할 때 하나의 컬렉션만 확장되고 PlantDetailScreen으로 이동하는 로직도 필요합니다. 그렇다면 해당 로직은 어디에 있어야 할까요? ViewModel 내부일까요? 재사용 가능한 UI 구성요소로는 불가능 합니다.



```kotlin
@Composable
fun CollectionCarousel(
	// State in
  // Events out
) {
  // ...
}
```



CollectionsCarousel 을 재사용 할 수 있게 하려면 상태를 전달하고 이벤트를 노출하는것이 좋습니다. 논리를 캡슐화 하고 클래스에서 컴포저블의 상태를 제어하는 것은 일관성 없는 UI가 표시되지 않도록 컴포저블을 보호하고 다른 장소에서 동일한 코드가 반복하지 않도록 하는 좋은 방법 입니다.

클래스에서 컴포저블의 상태를 제어하는 것은 일관성 없는 UI가 표시되지 않도록 컴포저블을 보호하고 다른 장소에서 동일한 코드가 반복하지 않도록 하는 좋은 방법 입니다.



```kotlin
data class PlantCollection(
	val name: String,
	@IdRes val asset: Int,
	val plants: List<Plant>
)

class CollectionCarouselState(
	private val collections: List<PlantCollection>
) {
	// ...
}
```



CollectionsCarousel 상태의 경우 컬렉션을 매개변수로 사용하는 CollectionsCarouselState라는 일반 클래스를 생성할 수 있습니다. 

```kotlin
class CollectionCarouselState(
	private val collections: List<PlantCollection>
) {
	var selectedIndex: Int? by mutableStateOf(null)
		private set
  val isExpanded: Boolean
  	get() = selectedIndex != null
  var plants by mutableStateOf(emptyList<Plant>())
  	private set
  	
  // ...
}
```

캐러셀이 선택한 컬렉션을 확장하는지에 대한 여부 및 표시할 식물에 대한 정보를 표시합니다. 



```kotlin
class CollectionsCarouselState(
	private val collections: List<PlantCollection>
) {
  // ...
  fun onCollectionClick(index: Int) {
    if (index >= collections.size || index < 0) return
    if (index == selectedIndex) {
      selectedIndex = null
    } else {
      plants = collections[index].plants
      selectedIndex = index
    }
  }
}
```

그리고 onCollectionClick에서 컬렉션을 클릭할 때 상태를 업데이트 하는 로직을 캡슐화 합니다.



```kotlin
@Composable
fun CollectionsCarousel(
	carouselState: CollectionsCarouselState
  onPlantClick: (Plant) -> Unit
) {
  // ...
}
```

이 상태는 이제 CollectionsCarousel의 발신자가 생성하고 관리해야 하는 매개변수로 컴포저블에 제공됩니다. 이벤트의 경우 CollectionsCarousel은 사용자가 식물을 탭 할때마다 호출되는 람다 형식의 추가 매개변수를 사용합니다.



하지만 ViewModel에서 모든 것을 사용하는 대신 CollectionsCarouselState를 사용자 상호 작용을 표시하는 이벤트와 일반 클래스로 사용하는지 알고 싶을 것 입니다.

컴포저블이 루트 상태에 가까울 때 상태를 호스팅 하고 로직을 처리하며 이벤트를 처리하는 ViewModel을 생성합니다. 예를 들자면 전체 화면을 구성하는 activity나 fragment의 화면을 구성하는 레이아웃 역할을 하는 컴포저블 입니다.

캐러샐과 같은 재사용 가능한 컴포저블 처럼 그렇지 않은 경우에는 ViewModel을 사용하지 말아야 합니다. 대신 상태를 관리하고 함수를 호출하는 이벤트를 노출하는 일반 상태 소유자 클래스를 만들어야 합니다.

왜냐하면 ViewModel이 activity, fragment나 탐색 그래프의 목적지로 지정되기 때문입니다. 따라서 해당 범위에서 ViewModel유형의 인스턴스를 하나만 사용 가능합니다.

선별 수준의 컴포저블에서 ViewModel의 이점은 configuration change 가 될 때 값이 살아날 수 있으며, SavedStateHandle 를 통해 프로세스가 종료 되어도 데이터가 살아남을 수 있다는 것 입니다. 또한 Lifecycle이 화면이 일치하는 상태에 더 적합하며 내장 코루틴 스코프로(viewModelScope?) 비즈니스 로직을 조정할 수 있습니다.



```kotlin
@Composable
fun HomeScreen(
	homeViewModel: HomeViewModel = viewModel()
) {
	val uiState by homeViewModel.uiState.collectAsState()
	
	CollectionsCarousel(
		carouselState = homeViewModel.uiState.carouselState,
		onPlantClick = {/* Navigate to Plant Detail screen*/}
	)
}
```

HomeScreen의 상태는 HomeViewModel로 관리되므로 carouselState는 ViewModel이 표시하는 UI상태에 포함되어야 합니다. 그리고 CollectionsCarousel로 전달 됩니다.



```kotlin
@Composable
fun HomeScreen(
	uiState: HomeUiState,
  plants: Flow<PagedData<Plant>>,
  onPlantClick: (Plant) -> Unit
) {
  // ...
  SearchPlants()
  CollectionsCarousel(
  	carouselState = uiState.carouselState,
    onPlantClick = onPlantClick
  )
  PlantList(plants)
  // ...
}
```



우리는 ViewModel과 상태 수집에 대한 걱정 없이 HomeScreen에 대한 테스트를 작성하고 가짜 데이터를 미리보기로 전달할 수 있기를 원합니다. HomeScreen을 변경하는 것은 쉽습니다. UI상태, 호출된 Plant의 Flow와 매개변수로 onPlantClick Lambda 를 추가하기만 하면 됩니다. 



```kotlin
val homeViewModel: HomeViewModel = viewModel()
val uiState by homeViewModel.uiState.collectAsState()
val plantList = homeViewModel.pagedPlants

// HomeScreen은 HomeViewModel로 부터 완전히 독립적이다.
HomeScreen(uiState, plantList, onPlantClick = {
  // TODO : Plant 상세 화면으로 Navigate하기.
})
```

ViewModel에 호출을 푸시하고 collectAsState를 한 단계 올렸습니다. 이는 앱의 다른 모든 것과 비교하여 홈 화면이 어디로 이동될지 생각해 볼 때 큰 차이가 있습니다. 



```kotlin
import androidx.activity.compose.setContent

@AndroidEntryPoint
class BloomActivity: ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    setContent {
      val homeViewModel: HomeViewModel = viewModel()
      val uiState by homeViewModel.uiState.collectAsState()
      val plantList = homeViewModel.pagedPlants

      HomeScreen(uiState, plantList, onPlantClick = {
        // TODO : Plant 상세 화면으로 Navigate하기.
      }) 
    }
  }
}
```

Bloom이 이 화면 하나로만 구성된 앱 이라면 activity에서 직접 composable을 설정하는 setContent메소드를 사용할 수 있습니다. 

하지만 Bloom앱에서는 훨씬 더 복잡한 것을 보여주기 때문에 컴포저블을 activity에 직접 추가하는 것은 올바른 방식이 아닐 수 있습니다. 

```kotlin
@AndroidEntryPoint
class HomeFragment: Fragment() {
  override fun onCreateView(inflater: LayoutInflater,
                            container: ViewGroup?, savedInstanceState: Bundle?
                           ) = ComposeView(requireContext()).apply {
    setContent {
      val homeViewModel: HomeViewModel = viewModel()
      val uiState by homeViewModel.uiState.collectAsState()
      val plantList = homeViewModel.pagedPlants

      HomeScreen(uiState, plantList, onPlantClick = {
        // TODO : Plant 상세 화면으로 Navigate하기.
      }) 
    }
  }
}
```



Compose를 채택하여 시작할 경우 기존 앱을 위한 자연 스러운 migration방법은 한번에 한 화면을 이전하는 것 입니다. 예를 들어 Fragment를 사용한다면 이 HomeScreen은 onCreateView()에서 ComposeView를 바로 생성하는 HomeFragment의 유일한 콘텐츠 일 수 있습니다. 이러한 유형의 접근 방식에서는 FragmentScenario와 같은 API로 프래그먼트를 검사합니다.

![006](https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-07-05-usingjetpack/006.png?raw=true)

앱의 모든 화면이 재사용 및 테스트 가능한 컴포저블을 둘러싼 래퍼일 뿐이라면 다음 단계는 Navigation Compose와 모든 화면을 하나로 묶는 것 입니다.

![007](https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-07-05-usingjetpack/007.png?raw=true)

Jetpack Navigation Component는 처음 부터 모든 유형의 목적지에 대해 아는 것이 없는 일반 런타임으로 구축 되었습니다. 이 개념은 Navigation Fragment 아티팩트를 구축하는데 사용 되었으며 최근에는 Navigation Compose 아티팩트를 구축 하였습니다.

이 공유 런타임은 각 구현이 Kotlin DSL, ViewModel의 수명주기 관련된 자동 scoping, Deep Linking, 결과 반환, 시스템 뒤로가기 버튼과의 통합을 통해 앱에 있는 화면 또는 목적지에 대한 탐색 그래프를 구축하기 위해 즉시 지원되는 것을 의미합니다.



```kotlin
setContent {
  val navController = remberNavController()
  Scaffold(
  	bottomBar = {/* NavController 사용 가능 */}
  ) { innerPadding -> 
     	NavHost(
     		navController,
     		startDestination = "home"
        modifier = Modifier.padding(innerPadding)
    	) {
        // ...
      }
   
  }
}
```

Navigation Compose에는 다음 2가지 주요 부분이 필요합니다. NavController와 NavHost입니다. NavController는 NavController을 입력으로 사용하는 NavHost와는 별도의 객체로 Stateful NavController를 생성할 수 있어 CollectionsCarousel 상태에서 사용된 것과 동일한 패턴을 따릅니다.

이렇게 하면 NavHost 외부의 기타 구성 요소가 NavController를 상태에 대한 단일 공급원으로 사용하여 현재 목적지 변경에 반응할 수 있습니다. 예를 들면 이는 하단 탐색 메뉴에서 선택한 항목을 선택할때 유용합니다.

NavHost Composable은 Composable목적지를 Compose 계층 구조에 추가하는 역할을 합니다. 여기서 Kotlin DSL 을 통해 탐색 그래프를 정의할 수 있습니다. 이는 앱에서 가능한 목적지를 모두 표시한 지도입니다. 따라서 HomeScreen의 경우 구성 가능한 홈 목적지를 만들 수 있습니다.

```kotlin
import android.hilt.navigation.compose.hiltNavGraphViewModel

NavHost(navController, startDestination = "home") {
  composable(route = "home") {
    val homeViewModel: HomeViewModel = hiltNavGraphViewModel()
    val uiState by homeVioewModel.uiState.collectAsState()
    val plantList = homeViewModel.pagedPlants
    
    HomeScreen(uiState, plantList, onPlantClick = {
      /* Plant Detail 화면으로 이동*/
    })
  }
}
```



HomeScreen에서 선언된 경로는 해당 목적지로 이동하는 데 사용되는 고유 경로 입니다. 여기는 앱의 메인 화면이므로 그래프의 시작 목적지로 설정합니다. 다만 코드에서 변경해야 할 부분이 있습니다.

여기서 ViewModel대신 Hilt Navigation Compose의 hiltNavGraphViewModel() 메서드를 사용합니다. Navigation 은 ViewModel을 개별 목적지로 자동 범위 지정 합니다.

하지만 목적지가 @AndroidEntryPoint Activity또는 Fragment가 아니기 때문에 기본 팩토리는 Hilt 팩토리가 아닙니다.

hiltNavGraphViewModel() 메서드는 viewModel()을 주변의 편리한 래퍼로 항상 올바른 팩토리가 설정되도록 합니다.

HomeScreen을 변경할 필요가 없었다는 사실을 알아 두어야 합니다.



Navigation Compose 내에서 사용되고 있고 여전히 별도의 테스트가 가능하다는 사실은 알려져 있지 않습니다.

물론 Navigation Compose의 큰 이점은 화면이 두 개 이상일 때 드러납니다. 이 때 savedState 및 ViewModel의 범위를 개별 목적지로 지정할 수 있으므로 필요한 경우에만 상태가 생성되고, 백 스택에서 목적지를 없앴을 때에만 제거되도록 합니다.

![008](https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-07-05-usingjetpack/008.png?raw=true)

Bloom의 경우 PlantList 또는 확장된 캐러셀의 작은 이미지가 상세 화면의 미리 보기가 되어야 합니다. 그 이미지로 사용자는 정원을 위해 살 수 있는 것을 파악하게 됩니다. PlantDetailsScreen을 빌드하는 것이 첫 단계 입니다. 

![009](https://github.com/kimsh3933/kimsh3933.github.io/blob/master/_posts/attach/2021-07-05-usingjetpack/009.png?raw=true)

HomeScreen처럼 미리보기를 사용하여 처음 부터 개별로 테스트할 수 있도록 PlantDetailScreen을 StateLess 구성요소로 빌드해야 합니다. 

재 사용 가능한 구성요소를 빌드하는 것은 앱 전체에서 사용자에게 보다 일관된 환경을 제공할 뿐만 아니라 새 화면을 구축할 때에도 도움이 됩니다.

```kotlin
NavHost(navController, startDestination = "home") {
  composable(route = "home") {
    // ...
  }
  composable(route = "plant/{id}", arguments = 
            listOf(navArgument(id) { type = NavType.IntType })
            ) { backStackEntry ->
    	val plantViewModel: PlantViewModel = hiltNavGraphViewModel()
      val plant by plantViewModel.plantDetails.collectAsState()
      PlantDetailScreen(plant)
  }
}
```

다음으로 PlantDetailScreen을 탐색 그래프에 추가해야 합니다. 이는 단지 또 다른 구성 가능한 목적지에 불과하지만 HomeScreen과 비교했을 때 한 가지 다른 점이 있습니다. 사용자가 선택한 식물을 알아야 합니다. Navigation의 경로와 웹 URL은 공통점이 많이 있습니다. 경로가 단순히 `plant/` 가 아니라 보여주고 싶은 식물의 ID를 포함한 `plant/{id}` 라는 것을 의미합니다. Plant클래스의 ID는 정수이므로 IntType로 선언합니다.



```kotlin
@HiltViewModel
class PlantViewModel @Inject constructor(
	plantRepository: PlantsRepository,
  savedStateHandle: SavedStateHandle
) : ViewModel() {
  val plantDetails: Flow<Plant> = plantsRepository.getPlantDetails(
  	savedStateHandle.get<Int>("id")
  )
}
```



이제 NavHost가 식물 상세 목적지로 전달되는 백 스택 항목에서 직접 ID를 추출할 수 있지만 이 곳이 아니라 PlantViewModel에서 해당 ID를 가져오고자 합니다. 이를 위해 SavedStateHandle를 사용할 수 있습니다. 경로에 입력한 인수가 자동으로 채워진다는 것이 중요합니다. 이를 통해 ViewModel이 해당 ID를 저장소로 바로 전달하여 식물 상세 정보를 검색하고 관찰 가능한 플로우를 UI에 제공하게 됩니다. 즉 백그라운드 로딩으로 식물 가격을 업데이트 하거나 단일 진실 공급원을 변경한다면 PlantDetailScreen은 새 데이터로 자동 재구성 됩니다. 

```kotlin
NavHost(navController, startDestination = "home") {
  composable(route = "home") {
    // ...
    HomeScreen(uiState, plantList, onPlantClick = { plant -> 
    	navController.navigate("plant/${plant.id}")
    }) 
  }
  
  composable(route = "plant/{id}") {
    // ...
  }
}
```

HomeScreen을 StateLess로 만든 덕분에 사용자가 식물을 탭할 때 작동되는 Lambda를 이미 노출했습니다. 그런 다음 HomeScreen 목적지가 PlantDetailScreen의 경로로 navigate()를 호출하고 ID를 입력하여 해당 Lambda를 구한합니다. NavController이 나머지 부분을 처리하여 HomeScreen의 상태를 저장하고 PlantDetailScreen으로 교체합니다.

시스템 뒤로 버튼을 누르면 자동으로 사용자가 종료한 상태와 동일한 HomeScreen으로 돌아갑니다. Jetpack Compose로 수행할 수 있는 작업 및 기타 JetPack 라이브러리와 통합 하는 부분을 간단하게 살펴 보았습니다.