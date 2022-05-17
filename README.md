![Now in Android](docs/images/nia-splash.jpg "Now in Android")

Now in Android App [Work in progress 🚧]
==================

This is the repository for the [Now in Android](https://developer.android.com/series/now-in-android)
app.

Now in Android is a fully functional Android app built entirely with Kotlin and Jetpack Compose. It
follows Android design and development best practices and is intended to be a useful reference
for developers. As a running app, it's intended to help developers keep up-to-date with the world
of Android development by providing regular news updates.

-> Kotlin과 Jetpack Compose으로 구현했습니다. Android 디자인, 개발의 Best practice를 따릅니다.

The app is currently in early stage development and is not yet available on the Play Store.


# Features

Now in Android displays content from the
[Now in Android](https://developer.android.com/series/now-in-android) series. Users can browse for
links to recent videos, articles and other content. Users can also follow topics they are interested
in or follow specific authors.

-> Now in Android 시리즈 목록을 보여줍니다. 사용자는 주제 또는 저자를 팔로우 할 수 있습니다.

## Screenshots

![Screenshot showing For You screen](docs/images/screenshot-1-foryou.png "Screenshot showing For You screen") 
![Screenshot showing Interests screen](docs/images/screenshot-2-interests.png "Screenshot showing Interests screen") 
![Screenshot showing Topic detail screen](docs/images/screenshot-3-topicdetail.png "Screenshot showing Topic detail screen")


# 개발 환경

Now in Android uses the Gradle build system and can be imported directly into the latest stable
version of Android Studio (available [here](https://developer.android.com/studio)). The `debug`
build can be built and run using the default configuration.

Once you're up and running, you can refer to the learning journeys below to get a better
understanding of which libraries and tools are being used, the reasoning behind the approaches to
UI, testing, architecture and more, and how all of these different pieces of the project fit
together to create a complete app.

-> 앱을 실행해보면 아래의 학습 과정을 참고하여 라이브러리, 도구, UI, 테스트, 아키텍쳐 등 이러한 접근법들의 이유를 학습할 수 있습니다.

NOTE: Building the app using an M1 Mac will require the use of
[Rosetta](https://support.apple.com/en-gb/HT211861). See
[the following bug](https://github.com/protocolbuffers/protobuf/issues/9397#issuecomment-1086138036)
for more details.

# 아키텍쳐

The Now in Android app follows the
[official architecture guidance](https://developer.android.com/topic/architecture) 
and is described in detail in the
[architecture learning journey](docs/ArchitectureLearningJourney.md).

-> Now in Android 앱은 공식 아키텍쳐 가이드를 따르며, architecture learning journey에 자세하게 설명되어있습니다.

# 빌드

The `debug` variant of `app` uses local data to allow immediate building and exploring the UI.

-> debug 빌드에선 로컬 데이터를 사용합니다.

The `staging` and `release` variants of `app` make real network calls to a backend server, providing
up-to-date data as new episodes of Now in Android are released. At this time, there is not a
public backend available.

The `benchmark` variant of `app` is used to test startup performance and generate a baseline profile
(see below for more information).

-> benchmark 빌드는 실행시간 성능 테스트를 위해 사용됩니다.

`app-nia-catalog` is a standalone app that displays the list of components that are stylized for
Now in Android.

-> app-nia-catalog 는 Now in Android를 위해 제작된 컴포넌트 목록을 보여주는 개별 앱입니다.

# 테스트

To facilitate testing of components, Now in Android uses dependency injection with
[Hilt](https://developer.android.com/training/dependency-injection/hilt-android).

-> 컴포넌트들을 테스트 하기 이ㅜ해 Now in Android는 DI(Hilt)를 사용합니다.

Most data layer components are defined as interfaces.
Then, concrete implementations (with various dependencies) are bound to provide those interfaces to
other components in the app.
In tests, Now in Android notably does _not_ use any mocking libraries.
Instead, the production implementations can be replaced with test doubles using Hilt's testing APIs
(or via manual constructor injection for `ViewModel` tests).

-> 대부분의 데이터 레이어 컴포넌트들은 인터페이스로 정의됩니다. 그리고 구현체들은 DI를 통해 바인딩 됩니다. 테스트에서 다른 Mocking 라이브러리를 사용하지 않습니다. 대신에 Hilt의 테스팅 API를 사용해 테스트 더블로 대체합니다.

These test doubles implement the same interface as the production implementations, and generally
provide a simplified (but still realistic) implementation with additional testing hooks.
This results in less brittle tests that may exercise more production code, instead of just verifying
specific calls against mocks.

-> 이런 테스트 더블들은 프로덕션 구현체처럼 같은 인터페이스를 구현합니다. 그리고 테스팅 훅이 추가된 단순하게 구현을 제공합니다. 이 결과로 다루기 힘든 테스트들은 더 적어지고, 프로덕션 코드에 더 집중할 수 있습니다.

Examples:
- In instrumentation tests, a temporary folder is used to store the user's preferences, which is
  wiped after reach test.
  This allows using the real `DataStore` and exercising all related code, instead of mocking the 
  flow of data updates.
  
  -> 계측 테스트에서 임시 폴더는 사용자의 설정을 저장하는 데에 쓰입니다. 이것은 테스트 이후에 지워집니다. 이것은 DataStore와 관련된 코드들을 테스트할 수 있게 해줍니다.

- There are `Test` implementations of each repository, which implement the normal, full repository
  interface and also provide test-only hooks.
  `ViewModel` tests use these `Test` repositories, and thus can use the test-only hooks to
  manipulate the the state of the `Test` repository and verify the resulting behavior, instead of
  checking that specific repository methods were called.
  
  -> 각 레포지토리의 테스트 구현들은 레포지토리 인터페이스를 구현하며 테스트 전용 훅을 제공합니다. ViewModel 테스트는 이런 테스트 레포지토리들을 사용합니다.

# UI

UI components are designed according to [Material 3 guidelines](https://m3.material.io/) and built
entirely using [Jetpack Compose](https://developer.android.com/jetpack/compose). 

-> UI 컴포넌트들은 Material 3 가이드를 따르도록 설계되었습니다. 그리고 Jetpack Compose를 사용해 구현되었습니다.

The app has two themes: 

- Dynamic color - uses colors based on the [user's current color theme](https://material.io/blog/announcing-material-you) (if supported)
- Default theme - uses predefined colors when dynamic color is not supported

Each theme also supports dark mode. 

The app uses adaptive layouts to
[support different screen sizes](https://developer.android.com/guide/topics/large-screens/support-different-screen-sizes).

Find out more about the [UI architecture here](docs/ArchitectureLearningJourney.md#ui-layer).

# Baseline profiles

The baseline profile for this app is located at `app/src/main/baseline-prof.txt`.
It contains rules that enable AOT compilation of the critical user path taken during app launch.
For more information on baseline profiles, read [this document](https://developer.android.com/studio/profile/baselineprofiles).

| Note: The baseline profile needs to be re-generated for release builds that touched code which
| changes app startup.

To generate the baseline profile, select the `benchmark` build variant and run the
`BaselineProfileGenerator` benchmark test on an AOSP Android Emulator.
Then copy the resulting baseline profile from the emulator to `app/src/main/baseline-prof.txt`.

# License

Now in Android is distributed under the terms of the Apache License (Version 2.0). See the
[license](LICENSE) for more information.
