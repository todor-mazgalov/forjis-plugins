---
name: android
description: >
  Android development skill. Produces clean, modern, idiomatic Android code
  following current Jetpack and Material Design 3 best practices. Enforces
  Kotlin-first development, Jetpack Compose for UI, Architecture Components
  (ViewModel, StateFlow, Room, Navigation), coroutines for async, and Hilt
  for dependency injection. Strongly prefers Jetpack libraries over deprecated
  platform APIs and manual implementations.
  TRIGGER when: project has build.gradle(.kts) with android plugin,
  AndroidManifest.xml, app/src/ directory structure, or user asks to build
  Android features, implement Android UI, fix Android bugs, or work with
  Activities, Fragments, Services, BroadcastReceivers, ContentProviders.
  Also trigger when user mentions Jetpack Compose, Room, ViewModel, Hilt,
  Navigation component, Material Design on Android, Android Gradle Plugin,
  or any Android-specific API (Intent, Bundle, SharedPreferences, etc.).
  DO NOT TRIGGER when: project is pure Kotlin/JVM without Android
  (no AndroidManifest.xml), Flutter/Dart project, React Native project,
  or Kotlin Multiplatform targeting only non-Android platforms.
---

# Android Development

Write clean, modern, idiomatic Android code. Prioritize Jetpack libraries,
Compose-first UI, and Architecture Components.

When the project also contains Java source files, the `java` skill applies
for pure Java language concerns (records, sealed classes, pattern matching,
JDK built-ins). This skill focuses on Android-specific patterns.

## Before Writing Code

1. **Read existing code first.** Search the project for existing screens, ViewModels,
   repositories, and shared modules before creating anything new. Reuse what exists.
2. **Check SDK versions.** Look at `build.gradle(.kts)` for `compileSdk`, `targetSdk`,
   `minSdk` to know which APIs are available. Check the Compose BOM version and AGP version.
3. **Check Kotlin vs Java.** Prefer Kotlin for all new code. If the project is Java-only,
   follow Java conventions but still use Jetpack libraries.
4. **Follow project conventions.** Match existing architecture (MVVM, MVI), DI framework,
   navigation approach, and testing style already in use.

## Modern Android Development

Use Jetpack and modern patterns. They exist because the old alternatives had real problems —
lifecycle bugs, boilerplate, thread-safety issues, configuration-change crashes.

### Jetpack Compose over XML layouts

```kotlin
// WRONG — XML layout for a new screen
// res/layout/fragment_profile.xml + ProfileFragment.kt with ViewBinding
class ProfileFragment : Fragment(R.layout.fragment_profile) {
    private var _binding: FragmentProfileBinding? = null
    private val binding get() = _binding!!

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        _binding = FragmentProfileBinding.bind(view)
        binding.nameText.text = viewModel.name.value
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}

// CORRECT — Composable function
@Composable
fun ProfileScreen(viewModel: ProfileViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    Column(modifier = Modifier.padding(16.dp)) {
        Text(text = uiState.name, style = MaterialTheme.typography.headlineMedium)
        Text(text = uiState.email, style = MaterialTheme.typography.bodyLarge)
    }
}
```

Compose eliminates the Fragment lifecycle dance, null-binding guards, and XML/code
split. New screens should always use Compose unless the project explicitly standardizes
on the View system.

### StateFlow over LiveData

```kotlin
// WRONG — LiveData in ViewModel, observe() in Fragment
class ProfileViewModel : ViewModel() {
    private val _name = MutableLiveData<String>()
    val name: LiveData<String> = _name
}
// In Fragment:
viewModel.name.observe(viewLifecycleOwner) { name -> binding.nameText.text = name }

// CORRECT — StateFlow in ViewModel, collectAsStateWithLifecycle in Compose
class ProfileViewModel @Inject constructor(
    private val userRepo: UserRepository,
) : ViewModel() {
    private val _uiState = MutableStateFlow(ProfileUiState())
    val uiState: StateFlow<ProfileUiState> = _uiState.asStateFlow()

    init {
        viewModelScope.launch {
            userRepo.getUser().collect { user ->
                _uiState.update { it.copy(name = user.name, email = user.email) }
            }
        }
    }
}

data class ProfileUiState(
    val name: String = "",
    val email: String = "",
)
```

StateFlow is Kotlin-native, works naturally with coroutines, and
`collectAsStateWithLifecycle()` handles lifecycle automatically in Compose.

### ViewModel with SavedStateHandle

```kotlin
// WRONG — storing state in Activity fields
class DetailActivity : AppCompatActivity() {
    private var itemId: String = ""  // Lost on process death

    override fun onCreate(savedInstanceState: Bundle?) {
        itemId = intent.getStringExtra("id") ?: ""
    }
}

// CORRECT — ViewModel + SavedStateHandle
@HiltViewModel
class DetailViewModel @Inject constructor(
    savedStateHandle: SavedStateHandle,
    private val itemRepo: ItemRepository,
) : ViewModel() {
    private val itemId: String = checkNotNull(savedStateHandle["id"])

    val item = itemRepo.getItem(itemId)
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), null)
}
```

SavedStateHandle survives process death. ViewModel survives configuration changes.
Together they cover both scenarios.

### Navigation Component over manual Fragment transactions

```kotlin
// WRONG — manual fragment management
supportFragmentManager.beginTransaction()
    .replace(R.id.container, DetailFragment.newInstance(id))
    .addToBackStack(null)
    .commit()

// CORRECT — Compose Navigation
@Composable
fun AppNavHost(navController: NavHostController = rememberNavController()) {
    NavHost(navController, startDestination = "home") {
        composable("home") { HomeScreen(onItemClick = { id -> navController.navigate("detail/$id") }) }
        composable("detail/{id}") { backStackEntry ->
            DetailScreen(id = backStackEntry.arguments?.getString("id") ?: "")
        }
    }
}
```

Navigation Component handles back stack, deep links, arguments, and transitions.
Type-safe navigation with Kotlin serialization is available in recent versions.

### Coroutines over AsyncTask / HandlerThread

```kotlin
// WRONG — AsyncTask (deprecated), raw Thread, HandlerThread
object : AsyncTask<Void, Void, List<User>>() {
    override fun doInBackground(vararg params: Void?): List<User> = api.getUsers()
    override fun onPostExecute(result: List<User>) { adapter.submitList(result) }
}.execute()

// CORRECT — coroutines with structured concurrency
viewModelScope.launch {
    val users = withContext(Dispatchers.IO) { api.getUsers() }
    _uiState.update { it.copy(users = users) }
}
```

`viewModelScope` is lifecycle-aware — coroutines cancel automatically when the
ViewModel clears. Use `Dispatchers.IO` for disk/network, never block the main thread.

### Hilt over manual DI

```kotlin
// WRONG — manual singleton pattern
object ApiClient {
    val retrofit: Retrofit = Retrofit.Builder().baseUrl(BASE_URL).build()
    val userService: UserService = retrofit.create(UserService::class.java)
}

// CORRECT — Hilt modules
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit = Retrofit.Builder().baseUrl(BASE_URL).build()

    @Provides
    @Singleton
    fun provideUserService(retrofit: Retrofit): UserService =
        retrofit.create(UserService::class.java)
}

// ViewModel gets dependencies automatically
@HiltViewModel
class UserViewModel @Inject constructor(
    private val userService: UserService,
) : ViewModel()
```

Hilt provides compile-time verified DI with minimal boilerplate. Use `@Singleton`
for app-wide singletons, `@ViewModelScoped` for ViewModel-scoped dependencies.

### Room over raw SQLite

```kotlin
// WRONG — raw SQLiteOpenHelper
class DbHelper(context: Context) : SQLiteOpenHelper(context, "app.db", null, 1) {
    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)")
    }
    override fun onUpgrade(db: SQLiteDatabase, old: Int, new: Int) {}
}

// CORRECT — Room with Flow
@Entity
data class UserEntity(
    @PrimaryKey val id: Int,
    val name: String,
)

@Dao
interface UserDao {
    @Query("SELECT * FROM UserEntity")
    fun getAll(): Flow<List<UserEntity>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(users: List<UserEntity>)
}

@Database(entities = [UserEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

Room provides compile-time SQL verification, reactive queries via Flow, and
automatic migration support. Return `Flow` for observable queries, `suspend`
for one-shot operations.

## Prefer Jetpack Over Deprecated APIs

| Deprecated / Old API | Modern Replacement | Notes |
|----------------------|--------------------|-------|
| `AsyncTask` | Coroutines + `viewModelScope` | Deprecated API 30 |
| `Loader` / `CursorLoader` | Room + Flow | Reactive, lifecycle-aware |
| `SharedPreferences` | Jetpack DataStore | Async, type-safe, no ANR risk |
| `LocalBroadcastManager` | `SharedFlow` / callbacks | Deprecated in AndroidX |
| `startActivityForResult` | Activity Result API | `registerForActivityResult()` |
| `ViewPager` | `ViewPager2` or `HorizontalPager` (Compose) | Better RTL, vertical support |
| Platform `AlertDialog` | Material 3 `AlertDialog` (Compose) | Themed, accessible |
| `PreferenceFragment` | Jetpack `PreferenceFragmentCompat` | AndroidX, consistent |
| `IntentService` | `WorkManager` | Survives process death, constraints |
| XML menus | Compose `TopAppBar` actions | For Compose screens |
| Manual JSON parsing | kotlinx.serialization / Moshi | Type-safe, less error-prone |

Before using any deprecated API, check if Jetpack has a modern replacement. It usually does.

## Architecture

### Unidirectional Data Flow

```
UI → Events → ViewModel → Repository → Data Sources
UI ← State  ← ViewModel ← Repository ← Data Sources
```

Model UI state as a single data class. ViewModel exposes `StateFlow<UiState>`,
UI collects it. Events flow up as method calls, state flows down as data.

```kotlin
// UI State — single source of truth for the screen
data class OrderListUiState(
    val orders: List<Order> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null,
)

// ViewModel — transforms data into UI state
@HiltViewModel
class OrderListViewModel @Inject constructor(
    private val orderRepo: OrderRepository,
) : ViewModel() {
    private val _uiState = MutableStateFlow(OrderListUiState())
    val uiState: StateFlow<OrderListUiState> = _uiState.asStateFlow()

    init { loadOrders() }

    fun loadOrders() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, error = null) }
            orderRepo.getOrders()
                .onSuccess { orders -> _uiState.update { it.copy(orders = orders, isLoading = false) } }
                .onFailure { e -> _uiState.update { it.copy(error = e.message, isLoading = false) } }
        }
    }
}
```

### Repository Pattern

```kotlin
// Repository abstracts data sources — ViewModel never touches Retrofit/Room directly
class OrderRepository @Inject constructor(
    private val api: OrderApi,
    private val dao: OrderDao,
) {
    fun getOrders(): Flow<List<Order>> = dao.getAll().map { entities ->
        entities.map { it.toDomain() }
    }

    suspend fun refreshOrders(): Result<Unit> = runCatching {
        val orders = api.getOrders()
        dao.insertAll(orders.map { it.toEntity() })
    }
}
```

Room is the single source of truth, not the network response. Network updates
flow through Room so the UI always reflects consistent state.

## Jetpack Compose

### State Hoisting

```kotlin
// WRONG — stateful composable that's hard to test and reuse
@Composable
fun SearchBar() {
    var query by remember { mutableStateOf("") }
    TextField(value = query, onValueChange = { query = it })
}

// CORRECT — state hoisted, composable is stateless
@Composable
fun SearchBar(query: String, onQueryChange: (String) -> Unit) {
    TextField(value = query, onValueChange = onQueryChange)
}
```

Stateless composables are reusable, testable, and previewable. Hoist state to
the caller that needs to control it.

### Side Effects

```kotlin
// LaunchedEffect — runs when key changes, cancels on leave/recomposition
LaunchedEffect(userId) {
    viewModel.loadUser(userId)
}

// rememberCoroutineScope — for event-driven coroutines (button clicks)
val scope = rememberCoroutineScope()
Button(onClick = { scope.launch { viewModel.save() } }) { Text("Save") }

// DisposableEffect — cleanup on leave
DisposableEffect(lifecycleOwner) {
    val observer = LifecycleEventObserver { _, event -> /* ... */ }
    lifecycleOwner.lifecycle.addObserver(observer)
    onDispose { lifecycleOwner.lifecycle.removeObserver(observer) }
}
```

Never do network calls or database reads directly in a `@Composable` function.
Use `LaunchedEffect` or trigger from ViewModel.

### Material Design 3

```kotlin
// Use MaterialTheme tokens, not hardcoded values
Text(
    text = "Title",
    style = MaterialTheme.typography.headlineMedium,
    color = MaterialTheme.colorScheme.onSurface,
)

// Dynamic color (Material You) — adapts to user's wallpaper
val colorScheme = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
    if (isSystemInDarkTheme()) dynamicDarkColorScheme(context)
    else dynamicLightColorScheme(context)
} else {
    if (isSystemInDarkTheme()) darkColorScheme() else lightColorScheme()
}

MaterialTheme(colorScheme = colorScheme) { /* app content */ }
```

### Compose Performance

```kotlin
// Use key in LazyColumn to help Compose track items efficiently
LazyColumn {
    items(orders, key = { it.id }) { order ->
        OrderCard(order)
    }
}

// Use derivedStateOf to reduce unnecessary recompositions
val showScrollToTop by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 5 }
}

// Mark stable types to help the Compose compiler skip recomposition
@Immutable
data class OrderUiModel(val id: String, val title: String, val total: Double)
```

## Gradle Build Configuration

### Version Catalog

```toml
# gradle/libs.versions.toml
[versions]
agp = "8.7.0"
kotlin = "2.0.21"
compose-bom = "2024.12.01"
hilt = "2.53.1"
room = "2.6.1"

[libraries]
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version = "2.0.21-1.0.28" }
```

Use version catalogs for all dependencies. Pin the Compose BOM to get consistent
Compose library versions. Use KSP instead of kapt for annotation processing.

### Build Variants and Signing

```kotlin
// app/build.gradle.kts
android {
    compileSdk = 35

    defaultConfig {
        minSdk = 26
        targetSdk = 35
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
    }
}
```

Keep `minSdk` as high as reasonable (26+ covers 95%+ of devices). Enable
minification for release builds — R8 removes unused code and obfuscates.

## Lifecycle Awareness

```kotlin
// WRONG — collecting Flow without lifecycle awareness
lifecycleScope.launch {
    viewModel.uiState.collect { state -> /* updates even when app is backgrounded */ }
}

// CORRECT — lifecycle-aware collection
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { state -> updateUi(state) }
    }
}

// In Compose — collectAsStateWithLifecycle handles this automatically
val uiState by viewModel.uiState.collectAsStateWithLifecycle()
```

Never collect Flows without lifecycle awareness in Activities/Fragments — it wastes
resources and can cause crashes when updating a stopped UI. In Compose, use
`collectAsStateWithLifecycle()` which stops collection when the lifecycle drops
below STARTED.

Handle configuration changes via ViewModel, not `android:configChanges` in the manifest.
The manifest hack hides lifecycle bugs and breaks accessibility features.

## Permissions and Security

```kotlin
// WRONG — requesting all permissions at startup
// CORRECT — request at point of use with Activity Result API
val cameraPermission = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.RequestPermission(),
) { granted ->
    if (granted) openCamera() else showPermissionRationale()
}

Button(onClick = { cameraPermission.launch(Manifest.permission.CAMERA) }) {
    Text("Take Photo")
}
```

- Request permissions when the user triggers the feature, not at app startup
- Degrade gracefully when permission is denied
- No sensitive data in logs — guard with `BuildConfig.DEBUG`
- HTTPS only; use network security config for dev exceptions
- Use `EncryptedSharedPreferences` or encrypted DataStore for sensitive data

## Testing

| Layer | Framework | What to Test |
|-------|-----------|--------------|
| ViewModel / business logic | JUnit + Turbine | State transitions, data mapping |
| Compose UI | `compose-ui-test` | Rendering, interactions, accessibility |
| Integration | Robolectric | Android framework without device |
| End-to-end | Espresso / UI Automator | Full user flows on device/emulator |

```kotlin
// ViewModel test with Turbine
@Test
fun `loadOrders emits loading then success`() = runTest {
    val fakeRepo = FakeOrderRepository(orders = listOf(testOrder))
    val viewModel = OrderListViewModel(fakeRepo)

    viewModel.uiState.test {
        assertThat(awaitItem().isLoading).isTrue()
        val success = awaitItem()
        assertThat(success.isLoading).isFalse()
        assertThat(success.orders).containsExactly(testOrder)
    }
}

// Compose UI test
@get:Rule val composeTestRule = createComposeRule()

@Test
fun profileScreen_displaysUserName() {
    composeTestRule.setContent {
        ProfileScreen(uiState = ProfileUiState(name = "Alice"))
    }
    composeTestRule.onNodeWithText("Alice").assertIsDisplayed()
}
```

Use fake repositories (not mocks) for ViewModel tests — they're simpler and
test real behavior. Use `TestDispatcher` for coroutine testing.

## Performance

### Avoid ANR

- No disk or network I/O on the main thread
- Use `viewModelScope` + `Dispatchers.IO` for blocking work
- Use `WorkManager` for long-running background tasks
- Enable `StrictMode` in debug builds to catch violations

### Memory Management

- No static references to `Activity` or `Context` — this leaks the entire view hierarchy
- Use `applicationContext` for long-lived objects
- Prefer lifecycle-aware components over `WeakReference`
- Profile with LeakCanary in debug builds

### UI Performance

- Use `key` parameter in `LazyColumn` / `LazyRow` items
- Mark data classes as `@Immutable` or `@Stable` when safe
- Use `derivedStateOf` to reduce recomposition scope
- Profile with Layout Inspector and Compose composition tracing
- Add Baseline Profiles for startup optimization

## Common Mistakes to Avoid

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| XML layouts for new screens | Compose is the modern standard | Use `@Composable` functions |
| `LiveData` in new code | StateFlow is more Kotlin-idiomatic | `MutableStateFlow` + `collectAsStateWithLifecycle` |
| Network call on main thread | Causes ANR after 5 seconds | `viewModelScope.launch` + `withContext(IO)` |
| Static `Activity` reference | Memory leak of entire view hierarchy | Use `applicationContext` or lifecycle-aware components |
| `configChanges` in manifest | Hides lifecycle bugs, breaks a11y | ViewModel + SavedStateHandle |
| `SharedPreferences` on main thread | Disk I/O causes jank | DataStore (async by design) |
| Hardcoded strings in Composables | Breaks i18n, no preview isolation | `stringResource(R.string.xxx)` |
| God Activity with all logic | Untestable, unmaintainable | MVVM: ViewModel + Repository |
| Manual Fragment transactions | Error-prone, no type safety | Navigation Component |
| Ignoring process death | State lost on low-memory kill | SavedStateHandle in ViewModel |
| Not testing Compose UI | Regressions slip through | `ComposeTestRule` + semantic matchers |
| `runBlocking` on main thread | Defeats coroutines, freezes UI | Use `suspend` or `viewModelScope.launch` |
| Destructuring props in Compose | Breaks reactivity tracking | Pass the full state object |
| `kapt` for annotation processing | Slow, deprecated | Use KSP instead |

## Project Structure Conventions

Follow whatever structure the project already uses. If starting fresh, prefer:

```
app/
├── src/main/
│   ├── java/com/example/myapp/
│   │   ├── di/                # Hilt modules (@Module, @InstallIn)
│   │   ├── data/
│   │   │   ├── local/         # Room entities, DAOs, database
│   │   │   ├── remote/        # Retrofit services, API DTOs
│   │   │   └── repository/    # Repository implementations
│   │   ├── domain/            # Domain models, use cases (optional layer)
│   │   ├── ui/
│   │   │   ├── theme/         # Material 3 theme, colors, typography
│   │   │   ├── navigation/    # NavHost, route definitions
│   │   │   ├── components/    # Shared composables (buttons, cards, dialogs)
│   │   │   └── screens/
│   │   │       └── home/      # HomeScreen.kt, HomeViewModel.kt, HomeUiState.kt
│   │   └── MyApplication.kt   # @HiltAndroidApp
│   ├── res/
│   │   ├── values/            # strings.xml, themes.xml, dimens.xml
│   │   ├── values-night/      # Dark theme overrides
│   │   └── drawable/          # Vector drawables preferred over raster
│   └── AndroidManifest.xml
├── src/test/                   # Unit tests (JUnit, Turbine)
├── src/androidTest/            # Instrumented tests (Espresso, Compose)
├── build.gradle.kts
└── proguard-rules.pro

gradle/
└── libs.versions.toml          # Version catalog for all dependencies
```

Group by feature (screens/) rather than by type (all ViewModels together).
Each screen folder contains its composable, ViewModel, and UI state class.
