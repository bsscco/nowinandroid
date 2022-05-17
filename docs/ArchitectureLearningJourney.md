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


아래 표는 각 스탭에서 무슨 일이 일어나는지 보여줍니다. 연관된 코드를 찾는 가장 쉬운 방법은 안드로이드 스튜디오에서 프로젝트를 불러오는 것입니다. 그리고 아래 코드 열의 텍스트를 찾는 것이죠.


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
   <td>WorkManager는 동기화 job을 실행합니다. job은 원격 데이터소스로 데이터 동기화를 시작하기 위해 <code>OfflineFirstNewsRepository</code>를 호출합니다.
   </td>
   <td><code>SyncWorker.doWork</code>
   </td>
  </tr>
  <tr>
   <td>4
   </td>
   <td><code>OfflineFirstNewsRepository</code>는 실제 API 요청을 실행하기 위해 <code>RetrofitNiaNetwork</code>를 호출합니다. API 요청은 <a href="https://square.github.io/retrofit/">Retrofit</a>을 통해 수행됩니다.
   </td>
   <td><code>OfflineFirstNewsRepository.syncWith</code>
   </td>
  </tr>
  <tr>
   <td>5
   </td>
   <td><code>RetrofitNiaNetwork</code>는 원격서버에 REST API를 호출합니다.
   </td>
   <td><code>RetrofitNiaNetwork.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>6
   </td>
   <td><code>RetrofitNiaNetwork</code>는 원격서버로부터 응답을 받습니다.
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
   <td><code>NewsResourceDao</code>의 데이터가 바뀌었을 때 Dao는 <a href="https://developer.android.com/kotlin/flow">Flow</a> 스트림을 통해 뉴스 데이터를 방출합니다.
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
   <td><code>When ForYouViewModel</code>이 뉴스 리소스를 받았을 때 피드 상태를 <code>Success</code>로 갱신합니다. <code>ForYouScreen</code>은 뉴스 리소스를 사용해 화면을 그립니다.
<p>
화면은 뉴스 리소스를 보여줍니다.
   </td>
   <td><code>ForYouFeedState.Success</code>
   </td>
  </tr>
</table>



## Data layer

The data layer is implemented as an offline-first source of app data and business logic. It is the source of truth for all data in the app.



![Diagram showing the data layer architecture](images/architecture-3-data-layer.png "Diagram showing the data layer architecture")


Each repository has its own models. For example, the `TopicsRepository` has a `Topic` model and the `NewsRepository` has a `NewsResource` model.

Repositories are the public API for other layers, they provide the _only_ way to access the app data. The repositories typically offer one or more methods for reading and writing data.


### Reading data

Data is exposed as data streams. This means each client of the repository must be prepared to react to data changes. Data is not exposed as a snapshot (e.g. `getModel`) because there's no guarantee that it will still be valid by the time it is used.

Reads are performed from local storage as the source of truth, therefore errors are not expected when reading from `Repository` instances. However, errors may occur when trying to reconcile data in local storage with remote sources. For more on error reconciliation, check the data synchronization section below.

_Example: Read a list of authors_

A list of Authors can be obtained by subscribing to `AuthorsRepository::getAuthorsStream` flow which emits `List<Authors>`.

Whenever the list of authors changes (for example, when a new author is added), the updated `List<Author>` is emitted into the stream.


### Writing data

To write data, the repository provides suspend functions. It is up to the caller to ensure that their execution is suitably scoped.

_Example: Follow a topic _

Simply call `TopicsRepository.setFollowedTopicId` with the ID of the topic which the user wishes to follow.


### Data sources

A repository may depend on one or more data sources. For example, the `OfflineFirstTopicsRepository` depends on the following data sources:


<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Backed by</strong>
   </td>
   <td><strong>Purpose</strong>
   </td>
  </tr>
  <tr>
   <td>TopicsDao
   </td>
   <td><a href="https://developer.android.com/training/data-storage/room">Room/SQLite</a>
   </td>
   <td>Persistent relational data associated with Topics
   </td>
  </tr>
  <tr>
   <td>NiaPreferences
   </td>
   <td><a href="https://developer.android.com/topic/libraries/architecture/datastore">Proto DataStore</a>
   </td>
   <td>Persistent unstructured data associated with user preferences, specifically which Topics the user is interested in. This is defined and modeled in a .proto file, using the protobuf syntax.
   </td>
  </tr>
  <tr>
   <td>NiANetwork
   </td>
   <td>Remote API accessed using Retrofit
   </td>
   <td>Data for topics, provided through REST API endpoints as JSON.
   </td>
  </tr>
</table>



### Data synchronization

Repositories are responsible for reconciling data in local storage with remote sources. Once data is obtained from a remote data source it is immediately written to local storage. The  updated data is emitted from local storage (Room) into the relevant data stream and received by any listening clients.

This approach ensures that the read and write concerns of the app are separate and do not interfere with each other.

In the case of errors during data synchronization, an exponential backoff strategy is employed. This is delegated to `WorkManager` via the `SyncWorker`, an implementation of the `Synchronizer` interface.

See the `OfflineFirstNewsRepository.syncWith` for an example of data synchronization.


## UI Layer

The [UI layer](https://developer.android.com/topic/architecture/ui-layer) comprises:



*   UI elements built using [Jetpack Compose](https://developer.android.com/jetpack/compose)
*   [Android ViewModels](https://developer.android.com/topic/libraries/architecture/viewmodel)

The ViewModels receive streams of data from repositories and transform them into UI state. The UI elements reflect this state, and provide ways for the user to interact with the app. These interactions are passed as events to the view model where they are processed.


![Diagram showing the UI layer architecture](images/architecture-4-ui-layer.png "Diagram showing the UI layer architecture")


### Modeling UI state

UI state is modeled as a sealed hierarchy using interfaces and immutable data classes. State objects are only ever emitted through the transform of data streams. This approach ensures that:



*   the UI state always represents the underlying app data - the app data is the source-of-truth.
*   the UI elements handle all possible states.

**Example: News feed on For You screen**

The feed (a list) of news resources on the For You screen is modeled using `ForYouFeedState`. This is a sealed interface which creates a hierarchy of two possible states:



*   `Loading` indicates that the data is loading
*   `Success` indicates that the data was loaded successfully. The Success state contains the list of news resources.

The `feedState` is passed to the `ForYouScreen` composable, which handles both of these states.


### Transforming streams into UI state

View models receive streams of data as cold [flows](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html) from one or more repositories. These are [combined](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html) together to produce a single flow of UI state. This single flow is then converted to a hot flow using [stateIn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html). The conversion to a state flow enables UI elements to read the last known state from the flow.

**Example: Displaying followed topics and authors**

The `FollowingViewModel` exposes `uiState` as a `StateFlow<FollowingUiState>`. This hot flow is created by combining four data streams:



*   List of authors (getAuthorsStream)
*   List of author IDs which the current user is following
*   List of topics
*   List of topic IDs which the current user is following

The list of `Author`s is mapped to a new list of `FollowableAuthor`s. `FollowableAuthor` is a wrapper for `Author` which also indicates whether the current user is following that author. The same transformation is applied for the list of `Topic`s.

The two new lists are used to create a `FollowingUiState.Interests` state which is exposed to the UI.


### Processing user interactions

User actions are communicated from UI elements to view models using regular method invocations. These methods are passed to the UI elements as lambda expressions.

**Example: Following a topic**

The `FollowingScreen` takes a lambda expression named `followTopic` which is supplied from `FollowingViewModel.followTopic`. Each time the user taps on a topic to follow this method is called. The view model then processes this action by informing the topics repository.


## Further reading

[Guide to app architecture](https://developer.android.com/topic/architecture)

[Jetpack Compose](https://developer.android.com/jetpack/compose)
