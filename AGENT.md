# AGENT.md

이 문서는 `README.md`에 명시된 "Apex Legend 정보 앱" 프로젝트를 성공적으로 수행하기 위한 개발 계획 및 가이드라인을 정의합니다.

---

## 1. 프로젝트 목표 (Project Goal)

Apex 레전드 게임의 파편화된 정보를 통합하여, 신규 및 복귀 사용자가 언제 어디서든 스마트폰 앱 하나로 **핵심 정보(레전드, 무기, 맵)를 쉽고 빠르게 확인할 수 있는** 안드로이드 애플리케이션을 개발합니다.

---

## 2. 개발 단계별 실행 계획 (Development Phases)

### Phase 1: 프로젝트 설정 및 기본 구조 설계

1.  **Android Studio 프로젝트 생성**
    * **Application Name:** ApexInfo
    * **Package Name:** (com.example.apexinfo)
    * **Language:** Kotlin
    * **Minimum SDK:** API 24 (Nougat) 이상
    * **Build Configuration:** Gradle with Kotlin DSL (build.gradle.kts)

2.  **아키텍처(MVVM) 설정**
    * `model`, `view`, `viewmodel` 패키지를 생성하여 기본 구조를 잡습니다.
    * 각 기능(레전드, 무기 등)별로 하위 패키지를 구성합니다.

3.  **라이브러리 의존성 추가**
    * `build.gradle.kts (app)` 파일에 아래 라이브러리를 추가합니다.
        * **UI:** Jetpack Compose, RecyclerView
        * **네트워크:** Retrofit2, OkHttp3, Gson/Moshi Converter
        * **이미지 로딩:** Coil
        * **비동기 처리:** Kotlin Coroutines
        * **ViewModel & LiveData:** `androidx.lifecycle` 관련 라이브러리

### Phase 2: 데이터 모델 및 API 연동

1.  **데이터 클래스(Model) 정의**
    * `Legend.kt`, `Weapon.kt`, `GameMap.kt` 등 API 응답에 맞춰 데이터 클래스를 생성합니다. (e.g., `data class Legend(val name: String, val passive: Skill, ...)` )

2.  **네트워크 모듈 구현 (`Retrofit2`)**
    * `ApiService.kt`: `@GET` 어노테이션을 사용하여 Apex 레전드 데이터 API 엔드포인트를 정의하는 인터페이스를 생성합니다.
    * `RetrofitClient.kt`: Retrofit 인스턴스를 생성하는 싱글톤 객체를 구현합니다.

3.  **Repository 패턴 구현**
    * `LegendRepository.kt`: `ApiService`를 호출하여 원격 데이터를 가져오는 함수를 구현합니다. (e.g., `suspend fun getLegends(): List<Legend>`)

### Phase 3: 화면 개발 (UI & ViewModel)

1.  **메인 화면 (Home Screen) & 네비게이션**
    * **Activity:** `MainActivity.kt` 파일 생성
    * **UI:** `BottomNavigationView`를 포함하는 기본 레이아웃을 Jetpack Compose로 구성합니다. (Home, Legends, Weapons, Maps 탭)
    * **기능:** 최신 공지나 시즌 정보를 보여줄 간단한 UI를 배치합니다.

2.  **리스트 화면 (List Screen)**
    * **ViewModel:** `LegendListViewModel.kt`
        * `LegendRepository`를 통해 레전드 목록을 가져와 `LiveData` 또는 `StateFlow`에 저장합니다.
    * **UI:** `LegendListScreen.kt` (Composable function)
        * `LazyColumn` (Compose의 RecyclerView)을 사용하여 ViewModel의 레전드 목록을 표시합니다.
        * 각 아이템 클릭 시, 상세 화면으로 네비게이션 이벤트를 처리합니다.
        * 클래스별 필터링 UI (버튼 또는 드롭다운)를 추가하고, 선택된 필터 값을 ViewModel로 전달합니다.

3.  **상세 정보 화면 (Detail Screen)**
    * **ViewModel:** `LegendDetailViewModel.kt`
        * 리스트 화면에서 전달받은 ID를 기반으로 특정 레전드의 상세 정보를 로드합니다.
    * **UI:** `LegendDetailScreen.kt` (Composable function)
        * `Coil`을 사용하여 레전드 이미지를 로드합니다.
        * 능력(패시브, 전술, 얼티밋)에 대한 상세 설명을 `Text` 컴포저블로 표시합니다.

### Phase 4: 기능 확장 및 테스트

1.  **무기 및 맵 정보 기능 구현**
    * `Phase 3`의 리스트/상세 화면 개발 프로세스를 무기 및 맵 정보 기능에 동일하게 반복 적용합니다.

2.  **유닛 테스트 (Unit Test)**
    * ViewModel과 Repository가 예상대로 동작하는지 JUnit, Mockito/MockK를 사용하여 테스트 코드를 작성합니다.

3.  **UI 테스트 (UI Test)**
    * Espresso 또는 Compose Test 라이브러리를 사용하여 사용자의 상호작용(클릭, 스크롤)이 올바른 UI 변화를 일으키는지 검증합니다.

---

## 3. 기술 스택 상세 활용 계획 (Tech Stack Implementation)

* **`Kotlin`**: 모든 소스 코드는 Kotlin을 주 언어로 사용하여 간결하고 안전하게 작성합니다.
* **`MVVM`**: View(Compose)는 ViewModel의 데이터를 관찰하고, ViewModel은 Repository를 통해 데이터를 처리합니다. 이를 통해 UI와 비즈니스 로직을 명확히 분리합니다.
* **`Jetpack Compose`**: XML 레이아웃 대신 선언형 UI 툴킷인 Compose를 사용하여 전체 UI를 구축합니다. `LazyColumn`을 활용하여 효율적인 목록을 구현합니다.
* **`RecyclerView`**: 만약 XML 기반으로 프로젝트를 확장할 경우, `RecyclerView`를 사용하여 대규모 데이터 목록을 효율적으로 처리합니다. (주 UI는 Compose로 진행)
* **`Retrofit2`**: 외부 API 서버와 HTTP 통신을 설정하고, JSON 데이터를 Kotlin 객체로 변환하는 작업을 자동화합니다.
* **`Coil`**: URL로부터 이미지를 비동기적으로 로드하고 캐싱하여 UI에 표시합니다. 코루틴 기반으로 동작하여 Kotlin과 잘 통합됩니다.