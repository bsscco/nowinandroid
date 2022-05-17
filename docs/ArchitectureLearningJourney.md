# 아키텍쳐 학습 여행

이 학습여행에서 당신은 New in Android App 아키텍쳐에 대해 배울 겁니다. 이 아키텍쳐는 레이어와 핵심 클래스, 그리고 이들 사이의 상호작용에 대한 것을 말합니다.


## 목표와 요구사항

앱 아키텍쳐를 위한 목표:
*   가능한 한 [공식 아키텍쳐 가이드](https://developer.android.com/jetpack/guide)를 따릅니다.
*   개발자가 이해하기 쉽도록 너무 실험적인 것을 다루지 않습니다.
*   같은 코드베이스에서 여러 개발자가 일할 수 있도록 돕습니다.
*   CI를 통해 로컬 및 Instrumented 테스트를 촉진합니다.
*   빌드타임을 최소화합니다.


## 아키텍쳐 개요

이 앱의 아키텍쳐는 2가지 레이어를 가집니다. [데이터 레이어](https://developer.android.com/jetpack/guide/data-layer), [UI 레이어](https://developer.android.com/jetpack/guide/ui-layer) (추가적으로, [도메인 레이어](https://developer.android.com/jetpack/guide/domain-layer)).


<center>
<img src="images/architecture-1-overall.png" width="600px" alt="Diagram showing overall app architecture" />
</center>

이 아키텍쳐는 [단방향 데이터 흐름](https://developer.android.com/jetpack/guide/ui-layer#udf)과 반응형 프로그래밍 모델을 따릅니다. 아래는 두 레이어의 핵심 개념입니다:
*   상위 레이어는 하위 레이어의 변화에 반응합니다.
*   이벤트는 아래로 흐릅니다.
*   데이터는 위로 흐릅니다.

데이터는 [코틀린 Flows](https://developer.android.com/kotlin/flow)를 통해 구현된 스트림을 통해 흐릅니다.


### 예제: 당신의 화면 위에 뉴스 보여주기

앱이 처음 실행됐을 때 앱은 뉴스 리소스를 원격 서버로부터 불러옵니다. 원격 서버는 빌드 variant가 staging, realese일 때 사용되고, debug일 때는 로컬 데이터가 사용됩니다. 불러온 뉴스 리소스는 사용자의 관심사에 따라 필터링 되어 표시됩니다.

아래 다이어그램은 이벤트가 어떻게 발생하고, 이벤트를 처리하기 위해 관련된 객체에서 데이터가 어떻게 흐르는지 보여줍니다.


![Diagram showing how news resources are displayed on the For You screen](images/architecture-2-example.png "Diagram showing how news resources are displayed on the For You screen")


아래 표는 각 스탭에서 무슨 일이 일어나는지 보여줍니다. 연관된 코드를 찾는 가장 쉬운 방법은 안드로이드 스튜디오에서 프로젝트를 불러오는 것입니다. 그리고 아래 코드 열의 텍스트를 찾습니다.


<table>
  <tr>
   <td><strong>스탭</strong>
   </td>
   <td><strong>설명</strong>
   </td>
   <td><strong>코드 </strong>
   </td>
  </tr>
  <tr>
   <td>1
   </td>
   <td>앱이 실행됐을 때 모든 레포지토리를 동기화 하기 위해 <a href="https://developer.android.com/topic/libraries/architecture/workmanager">WorkManager</a> job이 스케줄에 추가됩니다.
   </td>
   <td><code>SyncInitializer.create</code>
   </td>
  </tr>
  <tr>
   <td>2
   </td>
   <td>뉴스 피드의 첫 상태는 <code>Loading</code>입니다. 로딩 스피너를 화면에 보여줍니다.
   </td>
   <td><code>ForYouFeedState.Loading</code>
   </td>
  </tr>
  <tr>
   <td>3
   </td>
   <td>WorkManager는 동기화 job을 실행합니다. job은 원격 데이터소스와 데이터를 동기화 하기 위해 <code>OfflineFirstNewsRepository</code>를 호출합니다.
   </td>
   <td><code>SyncWorker.doWork</code>
   </td>
  </tr>
  <tr>
   <td>4
   </td>
   <td><code>OfflineFirstNewsRepository</code>는 실제 API 요청을 위해 <code>RetrofitNiaNetwork</code>를 호출합니다. API 요청은 <a href="https://square.github.io/retrofit/">Retrofit</a>을 통해 수행됩니다.
   </td>
   <td><code>OfflineFirstNewsRepository.syncWith</code>
   </td>
  </tr>
  <tr>
   <td>5
   </td>
   <td><code>RetrofitNiaNetwork</code>는 REST API를 원격서버로 호출합니다.
   </td>
   <td><code>RetrofitNiaNetwork.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>6
   </td>
   <td><code>RetrofitNiaNetwork</code>가 원격서버로부터 응답을 받습니다.
   </td>
   <td><code>RetrofitNiaNetwork.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>7
   </td>
   <td><code>OfflineFirstNewsRepository</code>는 원격 데이터를 <code>NewsResourceDao</code>의 삽입, 수정, 삭제 기능을 사용해 로컬 <a href="https://developer.android.com/training/data-storage/room">Room database</a>로 동기화 합니다.
   </td>
   <td><code>OfflineFirstNewsRepository.syncWith</code>
   </td>
  </tr>
  <tr>
   <td>8
   </td>
   <td><code>NewsResourceDao</code>의 데이터가 바뀌었을 때 <code>NewsResourceDao</code>는 <a href="https://developer.android.com/kotlin/flow">Flow</a> 스트림을 통해 뉴스 데이터를 방출합니다.
   </td>
   <td><code>NewsResourceDao.getNewsResourcesStream</code>
   </td>
  </tr>
  <tr>
   <td>9
   </td>
   <td><code>OfflineFirstNewsRepository</code>는 이 스트림의 <a href="https://developer.android.com/kotlin/flow#modify">중간 연산자</a>로써 데이터 레이어 내부 데이터 모델인 <code>PopulatedNewsResource</code>를 공개된 <code>NewsResource</code> 모델로 변경합니다. <code>NewsResource</code>는 다른 레이어들에서 소비됩니다.
   </td>
   <td><code>OfflineFirstNewsRepository.getNewsResourcesStream</code>
   </td>
  </tr>
  <tr>
   <td>10
   </td>
   <td><code>When ForYouViewModel</code>이 뉴스 리소스를 받았을 때 뉴스 피드 상태를 <code>Success</code>로 갱신합니다. <code>ForYouScreen</code>은 뉴스 리소스를 사용해 화면을 그립니다.
<p>
화면은 뉴스 리소스를 사용자의 관심사에 따라 필터링 하여 보여줍니다.
   </td>
   <td><code>ForYouFeedState.Success</code>
   </td>
  </tr>
</table>



## 데이터 레이어

데이터 레이어는 오프라인 소스 기준으로 앱 데이터와 비지니스 로직을 구현합니다. 이것은 앱의 모든 데이터에 source of truth 원칙을 적용하기 위함입니다.



![데이터 레이어 아키텍쳐를 보여주는 다이어그램](images/architecture-3-data-layer.png "Diagram showing the data layer architecture")


각 레포지토리는 자신의 모델들을 가집니다. 예를 들어, `TopicsRepository`는 `Topic` 모델을, `NewsRepository`는 `NewsResource` 모델을 가집니다.

레포지토리는 다른 레이어들을 위한 공개 API이며 앱 데이터에 접근할 수 있는 유일한 방법입니다. 레포지토리에는 일반적으로 데이터를 읽고 쓸 수 있는 하나 이상의 메소드가 있습니다.


### 데이터 읽기

데이터는 데이터 스트림을 통해 노출됩니다. 이것은 레포지토리의 사용자가 데이터 변경에 대해 반응할 준비가 되어야 한다는 것을 의미합니다. 데이터는 스냅샷(예_ `getModal`)으로 노출되기 않습니다. 데이터가 사용될 때 여전히 유효하리라는 보장이 없기 때문입니다.

읽기는 source of truth로서 로컬 저장소로부터 읽습니다. 그러므로 `Repository` 인스턴스로부터 읽어올 때 오류를 기대하진 않습니다. 그러나 로컬 저장소의 데이터를 원격 소스에 저장하려할 때 오류가 발생할 순 있습니다. 원격 소스 저장 오류에 대해선 아래 synchronization 섹션을 참고하세요.

예: 저자 목록 읽기

저자 목록은 `AuthorsRepository::getAuthorsStream` flow로 구독될 수 있습니다. 이 flow는 `List<Authors>`를 방출합니다.

저자 목록이 변경될 때마다, 예를 들어 새로운 저자가 추가될 때, 변경된 `List<Author>`가 스트림에 방출될 것입니다.


### 데이터 쓰기

데이터를 쓰기 위해 레포지토리는 suspend 함수를 제공합니다. suspend 함수의 호출자는 함수가 실행되는 scope를 선택할 수 있습니다.

예: 주제를 팔로우 하기

간단하게 팔로우 하길 원하는 주제의 id를 넣어 `TopicsRepository.setFollowedTopicId`를 호출합니다.


### 데이터소스

레포지토리는 하나 이상의 데이터소스를 의존합니다. 예를 들어, `OfflineFirstTopicsRepository`는 다음의 데이터소스들을 의존합니다:


<table>
  <tr>
   <td><strong>이름</strong>
   </td>
   <td><strong>저장 방법</strong>
   </td>
   <td><strong>목적</strong>
   </td>
  </tr>
  <tr>
   <td>TopicsDao
   </td>
   <td><a href="https://developer.android.com/training/data-storage/room">Room/SQLite</a>
   </td>
   <td>주제와 관련있는 영속적 관계 데이터
   </td>
  </tr>
  <tr>
   <td>NiaPreferences
   </td>
   <td><a href="https://developer.android.com/topic/libraries/architecture/datastore">Proto DataStore</a>
   </td>
   <td>특히 유저가 관심있는 주제와 관련있는 영속적 비구조적 설정 데이터. 이것은 protobuf 구문을 사용한 .proto 파일로 모델링 되고 정의됩니다.
   </td>
  </tr>
  <tr>
   <td>NiANetwork
   </td>
   <td>Retrofit을 사용한 원격 API 접근
   </td>
   <td>JSON으로 REST API에 제공하는, 토픽 데이터
   </td>
  </tr>
</table>



### 데이터 동기화

레포지토리는 로컬 저장소 데이터를 원격 소스에 저장하는 책임을 집니다. 원격 데이터소스로부터 데이터를 내려받으면 즉시 로컬 저장소에 저장합니다. 변경된 데이터가 있다면 이 데이터는 로컬 저장소로부터 관련된 데이터 스트림을 통해 구독하고 있는 위치로 방출됩니다.

이 접근법은 앱의 읽기와 쓰기 관심사를 확실하게 분리시켜줍니다. (읽기는 구독을 통해 다른 위치에서 읽어질 수 있으므로)

데이터 동기화 중에 오류가 발생한다면 지수 백오프 전략이 사용됩니다. 이것은 `Synchronizer` 인터페이스의 구현체인 `SyncWorker`를 통해 `WorkManger`에 위임됩니다.

데이터 동기화 예시를 보려면 `OfflineFirstNewsRepository.syncWith`를 참고하세요.


## UI 레이어

[UI 레이어](https://developer.android.com/topic/architecture/ui-layer) 구성:



*   UI 요스들은 [Jetpack Compose](https://developer.android.com/jetpack/compose)를 사용해 빌드됩니다.
*   [Android ViewModels](https://developer.android.com/topic/libraries/architecture/viewmodel)

ViewModel은 레포지토리로부터 데이터 스트림을 받아 데이터를 UI 상태로 변환합니다. UI 요소들은 이 상태를 반영하고 사용자와 앱이 상호작용할 수 있는 방법을 제공합니다. 이러한 상호작용은 이벤트로써 ViewModel로 전달되어 처리됩니다.


![UI 레이어 아키텍쳐를 보여주는 다이어그램](images/architecture-4-ui-layer.png "Diagram showing the UI layer architecture")


### UI 상태 모델링하기

UI 상태는 sealed 인터페이스로 계층화 되며 immutable data 클래스로 모델링 됩니다. 상태 객체들은 오직 데이터 스트림의 변형을 통해 방출됩니다. 이 접근법은 다음의 이점이 있습니다:



*   UI 상태는 항상 source-of-truth를 기반으로 한 데이터를 대표합니다.
*   UI 요소들은 모든 가능한 상태를 다룹니다.

**예: ForYou 화면 위의 뉴스 피드**

ForYou 화면 위의 뉴스 피드 리소스는 `ForYouFeedState`를 사용해 모델링 됩니다. 이것은 2개의 가능한 상태로 구성된 sealed interface입니다. 



*   `Loading`는 데이터가 준비 중임을 의미합니다.
*   `Success`는 데이터를 성공적으로 불러왔음을 의미합니다. Success 상태는 뉴스 목록을 포함합니다.

`feedState`는 `ForYouScreen` composable로 전달됩니다.


### 스트림을 UI 상태로 변환하기

ViewModel은 한 개 이상의 레포지토리로부터 cold [flows](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html)로 된 데이터 스트림을 전달받습니다. 이 flow들은 [combined](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html) 연산자를 사용해 하나의 UI 상태 flow로 변환됩니다. 이 단일 flow는 [stateIn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html) 연산자를 통해 hot flow로 변환됩니다. 이러한 과정은 UI 요소들이 flow로부터 마지막 상태를 읽을 수 있게 만들어줍니다.

**예: 팔로잉 중인 주제와 저자를 보여주기**

`FollowingViewModel`은 `StateFlow<FollowingUiState>`를 통해 `uiState`를 내보냅니다. 이 hot flow는 4개의 데이터 스트림에 의해 만들어집니다:



*   저자 목록 (getAuthorsStream)
*   팔로잉 중인 저자들의 ID 목록
*   주제 목록
*   팔로잉 중인 주제들의 ID 목록

`Author` 목록은 팔로잉 중인지 아닌지를 담고 있는 `FollowableAuthor` 목록으로 매핑 됩니다. `Topic` 목록에도 같은 변형이 적용됩니다.

두 새로운 목록은 `FollowingUiState.Interests` 상태를 만드는 데에 사용됩니다.


### 사용자 상호작용 처리하기

사용자 액션은 UI 요소들로부터 ViewModel의 메소드를 호출함으로써 처리됩니다. 이러한 메소드는 람다로써 UI 요소에 전달됩니다.

**예: 주제 팔로잉 하기**

`FollowingScreen`은 `FollowingViewModel.followTopic`으로부터 `followTopic`이라는 람다를 가집니다. 사용자가 주제를 탭할 때마다 이 메소드가 호출됩니다. ViewModel은 topics 레포지토리를 사용해 이 액션을 처리합니다.


## 더 읽어보기

[Guide to app architecture](https://developer.android.com/topic/architecture)

[Jetpack Compose](https://developer.android.com/jetpack/compose)
