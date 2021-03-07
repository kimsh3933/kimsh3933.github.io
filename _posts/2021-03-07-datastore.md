---
date: 2021-03-07
title: "SharedPreference를 대체할 DataStore"
categories: Android
---


** 현재 DataStore는 알파 버전입니다. 변경사항이 있을 수 있으며, 적용 시 신중히 확인해보시고 적용 해야 합니다. 미리 파악해보시고 최소 베타가 나올때 적용하는 것이 좋을 것 같습니다. **

SharedPreference라는게 앱이 꺼지더라도 꼭 보존해야 하는 값들을 저장하는 데이터 저장소 입니다. 보통 변수 값들은 메모리에 저장되며, 앱이 메모리에서 내려가거나 기기가 꺼지면 메모리에 있던 변수 값들은 메모리에서 날아갑니다. 그래서 중요 데이터는 물리 저장소에 저장을 합니다. SharedPreference도 앱이 메모리에서 날아가버려도 지켜야 하는 값을 물리저장소에 xml파일로 집어 넣는 것이죠.

SharedPreference는 key-value 방식으로 저장을 합니다. 그리고 파일에 기록을 합니다.  이런 태생적인 특징으로 문제점도 있고 한계점도 있습니다. 그리고 이를 개선해보고자 구글에서 작년 중순에 DataStore라는 SharedPreference를 개선하는 데이터 저장소를 발표 하였습니다. DataStore는 어떤 특징을 가졌는지, SharedPreference와는 어떤 다른 점이 있는지 알아 봅시다.



# DataStore

- 프로토콜 버퍼를 사용하여 Key-Value 쌍으로 데이터를 저장하는 데이터 저장소 입니다.
  - 프로토콜 버퍼는 데이터 저장 등에 대해 직렬화 하여 사용할 수 있는 데이터 구조 입니다.
  - 구글에서 만든 json이나 xml같은거라고 생각하시면 됩니다.
  - [작성하는 방법](https://developers.google.com/protocol-buffers/docs/overview)
- 코틀린으로 라이브러리 코드로 작성되었습니다. 그리고 코루틴과 flow를 기반으로 작성되었습니다.
- 데이터는 비동기적으로, 트랜잭션 방식으로 저장이 됩니다.
- 크게 두 가지 방식으로 데이터를 저장 할 수 있습니다.
  - 기존 SharedPreference와 유사한 Preferences DataStore
  - 맞춤 데이터 유형으로 저장하는 Proto DataStore

[안드로이드 공식페이지](https://developer.android.com/topic/libraries/architecture/datastore) 에서는 SharedPreference를 사용하고 있다면 Datastore로 이전하도록 권유하고 있습니다.

많고 복잡한 데이터는 Room을 사용하도록 권유하고 있습니다.



## DataStore의 이점

- SharedPreference보다 안정적입니다.
  - 기존 SharedPreference는 크게 두 가지 방식으로 데이터를 업데이트 할 수 있습니다
    - commit, apply
  - commit은 동기적으로 데이터를 업데이트 합니다. 
    - 메모리에서 작업하는 것 보다 파일에 작업하는 것이기 때문에 필연적으로 느린편 입니다. 
    - 디스크 IO가 발생하며 파일을 다 쓸 때 까지 메소드를 block합니다.
    - 동기로 작업 하기 때문에 결과를 알 수 있습니다.
    - 코드들을 보면 메인 스레드에서 commit하는게 많던데 원래는 별도의 스레드에서 하는게 더 안정적입니다.
  - apply는 비동기적으로 데이터를 업데이트 합니다.
    - 결과를 알 수 없는 대신 commit보다 빠르게 값을 업데이트 할 수 있습니다.
  - SharedPreference를 사용하다 보면 Application Not Responding 에러, 런타임 에러가 종종 발생을 한다고 합니다.
    - 기본적으로 파일 I/O 작업을 하며 메모리에 로드될 때까지도 시간이 걸리기 때문에 UI스레드에서 호출하면 잠재적으로 ANR이 발생한다고 합니다.
  - DataStore는 런타임 에러가 없으며 코루틴의 Dispatchers.IO 아래에서 작동 되기 때문에 UI스레드에 안전하며 SharedPreference보다 안정적 입니다.
- SharedPreference 보다 Type-Safety합니다.
  - Proto DataStore의 경우 지정된 데이터 타입을 사용하기 때문에 type-safety합니다.
  - Preference Datastore의 경우 intPreferencesKey, stringPreferenceKey  등으로 타입을 명시 가능하고 해당 PreferenceKey를 string으로 작성된 key 대신 넣으면 되기 때문에 일일이 키를 넣고 타입을 지정해야 하는 sharedPreference보다 안정적 입니다.
- Proto datastore의 경우 다양한 타입을 넣을 수 있습니다.
  - 기존 SharedPreference의 경우 지정된 type만을 사용할 수 있었으며 여러개 중에 하나를 택하는 옵션을 구현해야 할 경우 int나 string등을 활용했어야 했습니다.
  - 옵션 같이 보기에는 한 묶음인 데이터들을 한값씩 일일이 관리했어야 했습니다. 그러면 해당 키를 관리하는 코드가 많이 늘어나는 문제가 있습니다.
  - Proto Datastore는 이를 해결하여 enum처럼 사용할 수 있게 하거나 한 객체가 들어가서 객체의 값을 바꾸는 느낌으로 dataStore를 사용할 수 있습니다.



## Gradle

DataStore를 사용하기 전에 gradle에 라이브러리를 등록 합니다.

```
implementation "androidx.datastore:datastore:1.0.0-alpha06"
implementation "androidx.datastore:datastore-preferences:1.0.0-alpha06"

// optional - Coroutine Support
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9"
// optional - LiveData Support
implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.3.0-alpha07"
implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"
    
// optional - RxJava2 support
implementation "androidx.datastore:datastore-preferences-rxjava2:1.0.0-alpha06"

  // optional - RxJava3 support
implementation "androidx.datastore:datastore-preferences-rxjava3:1.0.0-alpha06"
    
```



추가하는 라이브러리에 따라 LiveData나 RxJava와 조합이 가능합니다. 

## Preference DataStore

기존에 사용하던 SharedPreference와 유사한 DataStore입니다.

xml대신 프로토콜 버퍼를 사용하고 있습니다.

SharedPreference처럼 Key-value 를 사용하고 있습니다. 

단, 위에서 설명했듯이 key인 string값을 그대로 사용하는게 아닌 key에 타입을 명시할 수 있습니다.

```kotlin
val KEY = intPreferencesKey("firstKey")
```

이런식으로 key에 int라는 값을 명시할 수 있습니다. sharedPreference였다면 firstKey라는 문자열을 그대로 쓰고 사용할때마다 getInt, setInt를 써가면서 어떨땐 타입도 헷갈리고 하겠지만 dataStore에서는 그럴 일 없습니다.

DataStore의 데이터 저장소를 가져와 봅시다.

```kotlin
 val dataStore: DataStore<Preferences> = createDataStore(name="sample1")
```

SharedPreference와 동일하게 name을 지정해서 데이터 저장소를 가져올 수 있습니다.

그리고 dataStore는 이전에 있던 preference를 마이그레이션 할 수 있습니다.

```kotlin
val dataStoreMigrated: DataStore<Preferences> = createDataStore(name="sample1", migrations = listOf(SharedPreferencesMigration(this, "previous", setOf("keys"), deleteEmptyPreferences = false)))
```

SharedPreferenceMigration 을 활용하여 기존의 SharedPreference를 가져올 수 있습니다. 이름만 가져온다면 해당 SharedPreference 전부를 가져오게 되며, 특정 키만 가져온다고 하면 특정 키만 가져오게 됩니다. 가져온 특정 키의 값은 SharedPreference에서 삭제가 됩니다.

deleteEmptyPreferences 옵션은 migration하면서 키를 전부다 가져왔을 경우 sharedPreference를 지워 버릴것인지 묻는 옵션 입니다. 기본값은 true입니다.



이제 데이터 스토어에 값을 넣어 봅시다. 이전에 지정한 "firstKey"아니 KEY를 활용해 넣어 봅시다.

```kotlin
dataStore.edit { preference ->
	val current = preference[KEY] ?: 0
	preference[KEY] = current.plus(1)
}
```

dataStore.edit를 하고 preference를 비동기로 가져옵니다. 그리고 preference안에 KEY라는 값을 찾고 KEY자리에 값을 집어 넣습니다.

이제 값을 가져와 봅시다.

```kotlin
dataStore.data.map {it[KEY]}.first()
```

이렇게 가져올 수 있지만 메인스레드에서는 이렇게 가져올 수 없습니다.

메인스레드에서는 liveData를 활용하여 가져올 수 있는데요, gradle설정할 때 liveData support하는 라이브러리를 같이 넣어야 사용할 수 있습니다.

```kotlin
dataStore.data.map {it[KEY]}.asLiveData().observe(lifecycleOwner, Observer { value ->
	currentValue = value
})
```

 data를 라이브데이터화 시켜서 옵져버를 통해서 값을 가져올 수 있습니다.

위에 두 코드를 보면 map으로 값을 가져오는게 보입니다. map를 활용하면 가져온 값을 조작해서 조작한 값을 return하거나 여러 값을 가져와서 리스트화, 객체화 시킬 수 있습니다.



Preference DataStore만을 사용하여도 현재 사용하고 있는 SharedPreference를 어떻게 개선할까 생각이 드실 겁니다. 여기까지 해도 만족하실 분이 계시겠지만, 뭔가를 더 하고 싶으신 분들도 계실겁니다. sharedPreference에 enum을 넣는다던지, 클래스를 적용한다던지 등이죠. 이것을 proto dataStore를 통해서 해결 해봅시다.



## Proto DataStore

Proto DataStore는 프로토콜 버퍼라는 언어를 활용하여 데이터를 저장할 수 있게 하는 DataStore입니다. 



사용하기 전 프로토콜 버퍼 언어를 사용해서 만든 별도의 파일을 처리 하기 위해 gradle에 다음과 같은 항목을 추가 합니다.

```
plugins {
    ...
    id "com.google.protobuf" version "0.8.12"
}

dependencies {
    ...
    implementation  "com.google.protobuf:protobuf-javalite:3.10.0"
    ...
}

protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:3.10.0"
    }

    // Generates the java Protobuf-lite code for the Protobufs in this project. See
    // https://github.com/google/protobuf-gradle-plugin#customizing-protobuf-compilation
    // for more information.
    generateProtoTasks {
        all().each { task ->
            task.builtins {
                java {
                    option 'lite'
                }
            }
        }
    }
}
```

이렇게 추가를 해야지 proto datastore를 사용할 수 있습니다.



그리고 `app/src/main/proto`폴더 아래에 *.proto 파일을 만들어 봅시다.

```protobuf
syntax = "proto3"; 
option java_package = "com.package.your";
option java_multiple_files = true;
message MonitorSetting {
  int32 brightness = 1;
  string lauguage = 2;
  bool nightmode = 3;
  enum InputSource {
      HDMI = 0;
      DVI = 1;
      THUNDERBOLT = 2;
    }
   InputSource inputSource = 4;
}
```

여기서는 많이 쓰이는 int, string, boolean, enum을 사용해보도록 합시다.

만약 커스텀한 클래스를 사용하고 싶으시다면 하단에 별도의 클래스처럼 message라는 것을 만들어서 사용이 가능합니다. ([참고](https://developers.google.com/protocol-buffers/docs/proto3#other))

여기서 1, 2, 3 이라고 할당한 것은 값이 아니라 필드의 고유 번호 입니다. 필드를 식별하는데 사용됩니다. ([참고](https://developers.google.com/protocol-buffers/docs/proto3#assigning_field_numbers))

이렇게 작성하고 나서 컴파일을 하게 되면 MonitorSetting 이라는 클래스가 생깁니다. 그리고 proto에서 작성된 데이터 유형을 읽고 쓰기 위한 방법을 알려주기 위해 Serializer을 사용하게 됩니다.  defaultValue를 조작하면 기본값을 만들 수 있습니다.

```kotlin
object MonitorSettingSerializer : Serializer<MonitorSetting> {
    override val defaultValue: MonitorSetting = MonitorSetting.getDefaultInstance()
    override fun readFrom(input: InputStream): MonitorSetting {
        try {
            return MonitorSetting.parseFrom(input)
        } catch (exception: InvalidProtocolBufferException) {
            throw CorruptionException("Cannot read proto.", exception)
        }
    }

    override fun writeTo(t: MonitorSetting, output: OutputStream) = t.writeTo(output)
}
```

예를 들어 language 값을 "KOREAN"으로 하고 싶은 경우

```kotlin
override val defaultValue: MonitorSetting = MonitorSetting.getDefaultInstance().toBuilder().setLauguage("KOREAN").build()
```

이런 식으로 초기화 하면 됩니다. 메소드를 쓰셔도 되구요.



proto data store를 사용하기 위해서 데이터 저장소를 정의해볼까요?

```kotlin
val dataStore: DataStore<MonitorSetting> = dataStore = createDataStore(fileName = "monitor_settings", serializer = MonitorSettingSerializer)
```

DataStore에 generic 값을 방금 작성한 proto의 message 이름으로 생성된 클래스로 지정 하였습니다. 그리고 serializer에 방금 작성한 serializer을 사용 하였습니다.



이제 값을 집어 넣어 볼까요?

```kotlin
CoroutineScope(Dispatchers.IO).launch {
    dataStore.updateData { preferences ->
        preferences.toBuilder()
            .setBrightness(30)
            .setInputSource(MonitorSetting.InputSource.DVI)
            .setNightmode(false)
            .setLauguage("KOREAN")
            .build()
    }
}
```

외부에 파일을 쓰는거니 만큼 coroutine scope를 설정하도록 합시다.

각각 속성을 정해주고 난 다음에 build하면 값이 적용이 됩니다. 여기서는 값을 한번에 적용했지만, 한 값만 바꿔줘도 굅니다.



읽은 값을 확인하기 전에 파일을 읽고 쓰는거기 때문에 IO에 문제가 일어나면 값을 가져오지 못할 수 있습니다. 그래서 값을 조회하기 전에 catch를 하여 IOException이 일어난 경우 기본 값을 사용하도록 처리 합니다. DataStore는 뭔가 잘못된 경우 IOException을 낸다고 하네요.

```kotlin
val flow = dataStore.data.catch { exception ->
    if (exception is IOException) {
        emit(MonitorSetting.getDefaultInstance())
    } else {
        throw exception
    }
}
```



그리고 위에 Preference DataStore와 유사하게 map을 통해서 열면 됩니다.

```kotlin
flow.map{it}.first
```

처럼 다른 스레드에서 여는 방법이라던지



```
flow.map {
    it
}.asLiveData().observe(this, Observer {
    Log.i("Monitor", "Status = { Bright = ${it.brightness}, Language = ${it.lauguage}, InputSource = ${it.inputSource}}")
})
```

LiveData를 활용하여 값을 가져올 수 있는 방법이 있습니다.

여기서는 map에서 MonitorSetting 객체를 가져오도록 했는데요, 별도로 각 속성만을 가져와서 Observer을 부착할 수 있습니다.



Proto DataStore는 프로토콜 버퍼라는 새로운 형식을 배워야 하고 사용할 때 마다 일일이 작성해야 하는 불편함은 있습니다. 하지만 데이터 저장소를 작성 할 때 클래스를 작성하는 것과 비슷하게 특정 분류나 특정 기준으로 묶어서 데이터 저장소 키를 관리할 수 있어서 추후 유지보수에 용이할 것으로 보입니다. 



## 결론

지금까지 DataStore를 알아 보았습니다. SharedPreference를 사용하면서 불편하다고 느꼈던 것들을 DataStore를 통해서 어느정도 보완이 가능합니다.

DataStore에서 제공되는 key를 사용하면 해당 키를 사용하는 한 해당 키에 대한 타입을 틀리지 않을 것입니다. 물론 각 키가 사용하는 string 키 값이 중복될 수 있지만 고치는 것도 키를 교체할 필요 없이 string값만 바꾸면 되니깐 훨씬 쉽죠.

```kotlin
// AS-IS
val KEY_OLD = "KEY_OLD"
sharedPreferences.getInt(KEY_OLD)
val edit = sharedPreferences.edit()
edit.putString(KEY_OLD, 10)
edit.commit()

// TO-BE
val KEY_NEW = intPreferencesKey("KEY_NEW")
dataStore.data.map {
	it[KEY_NEW]
}.asLiveData().observe(this, Observer { value = it })
dataStore.edit {
  it[KEY_NEW] = 10
}
```

여기서도 KEY_OLD라는 키를 사용하는 sharedPreference를 보면 getInt, getString 값을 헷갈려버리면 빌드는 되지만, 사용할 때 오류가 나고 찾기도 힘듭니다.

하지만 KEY_NEW를 사용하는 dataStore는 KEY가 타입을 갖고 있어서 헷갈리거나 틀릴일이 없습니다. 틀린다 하더라도 해당 키만 수정하면 되구요.

그리고 Proto Datastore는 처음 만들기는 어렵지만 작성하면서 key에 저장된 값들을 의미 있게 묶을 수 있어 추후 관리에 용이할 것으로 보입니다.

앞에도 말씀드렸다시피 아직은 알파단계입니다. 아직 알파라서 사용하기 꺼려지시는 분들도 있을 수 있습니다. 하지만 바로 적용하지는 못하더라도 DataStore를 알아가고 베타나 안정된 버전이 나오는걸 기다리면서 마이그레이션을 하는 것도 나쁘지 않는 선택이라고 생각합니다.



# References

- https://developer.android.com/topic/libraries/architecture/datastore
- https://developers.google.com/protocol-buffers/docs/overview
- https://medium.com/scalereal/hello-datastore-bye-sharedpreferences-android-f46c610b81d5
- https://android-developers.googleblog.com/2020/09/prefer-storing-data-with-jetpack.html
- https://medium.com/@jurajkunier/migrating-sharedpreferences-to-jetpack-datastore-9deb8259063
- https://developer.android.com/codelabs/android-preferences-datastore#0
- https://ddolcat.tistory.com/184
- https://issuetracker.google.com/issues/117796731
- https://bcho.tistory.com/1182



