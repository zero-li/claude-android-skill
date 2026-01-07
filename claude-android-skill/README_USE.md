# Android Development Skill - Usage Guide

A production-ready skill for building Android apps with Kotlin, Jetpack Compose, and Koin dependency injection.

## When to Use

Use this skill when working on Android development tasks:

- Creating new Android projects or modules
- Building screens with Jetpack Compose
- Setting up MVVM architecture
- Implementing repositories and data sources
- Configuring dependency injection with Koin
- Working with Room database
- Setting up navigation with type-safe routes

## Quick Reference

### Create a New Feature Module

```
Hey Claude, create a new feature module for user settings in my Android project.
Package: com.example.app
Path: /path/to/project
```

Result: Generates `feature/settings/api` and `feature/settings/impl` with:
- Navigation route and key
- Compose Screen
- ViewModel with Koin injection
- Koin module for DI

### Build a Compose Screen

```
Hey Claude, build a profile screen with MVVM pattern.
Include user name, email, and avatar components.
```

Result: Creates Screen composable, ViewModel, UiState with proper state management.

### Set Up Repository Pattern

```
Hey Claude, create an offline-first repository for tasks.
Use Room for local storage and expose Flow<T>.
```

Result: Creates repository interface, implementation with Room DAO, data sources.

### Configure Dependency Injection

```
Hey Claude, add a new Koin module for my data layer.
I need to inject Repository and NetworkApi.
```

Result: Creates Koin module with proper `viewModel` and `single` declarations.

## Usage Examples

### Example 1: Basic Feature Module

```
Hey Claude—I just added the "android-development" skill.
Can you create a new feature module for user authentication?
Package: com.myapp.auth
Path: /Users/me/projects/MyApp
```

**Expected Output:**
```
✅ Feature module 'authentication' generated successfully!

Next steps:
1. Add to settings.gradle.kts:
   include(":feature:authentication:api")
   include(":feature:authentication:impl")
2. Add dependency in app/build.gradle.kts:
   implementation(projects.feature.authentication.impl)
3. Add Koin module in your App module:
   importKoinModules(listOf(authenticationModule))
4. Add navigation in NiaNavHost
```

### Example 2: Compose UI Screen

```
Hey Claude—I need help building a task list screen.
Use Jetpack Compose with MVVM pattern.
Include loading, success, and error states.
```

**Expected Output:**
```kotlin
// TaskListRoute.kt
@Composable
internal fun TaskListRoute(
    viewModel: TaskListViewModel = koinViewModel(),
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    TaskListScreen(
        uiState = uiState,
        onAction = viewModel::onAction,
    )
}

// TaskListViewModel.kt
class TaskListViewModel(
    private val taskRepository: TaskRepository,
) : ViewModel() {
    val uiState: StateFlow<TaskListUiState> = taskRepository
        .getTasks()
        .map { TaskListUiState.Success(it) }
        .stateIn(...)
}
```

### Example 3: Repository Implementation

```
Hey Claude—create an offline-first repository for articles.
Use Room for caching and expose reactive streams.
```

**Expected Output:**
```kotlin
// ArticlesRepository.kt
interface ArticlesRepository {
    fun getArticles(): Flow<List<Article>>
    suspend fun refreshArticles()
}

// OfflineFirstArticlesRepository.kt
internal class OfflineFirstArticlesRepository(
    private val articleDao: ArticleDao,
    private val networkApi: ArticleApi,
) : ArticlesRepository {

    override fun getArticles(): Flow<List<Article>> =
        articleDao.getAll().map { entities ->
            entities.map { it.toArticle() }
        }

    override suspend fun refreshArticles() {
        // Network fetch + Room cache
    }
}
```

### Example 4: Koin DI Setup

```
Hey Claude—I need to set up Koin dependency injection.
Create modules for my data and domain layers.
```

**Expected Output:**
```kotlin
// dataModule.kt
val dataModule = module {
    single<ArticleDao> { database.articleDao() }
    single<ArticleApi> { retrofit.create(ArticleApi::class.java) }
    single<ArticlesRepository> { OfflineFirstArticlesRepository(get(), get()) }
}

// domainModule.kt
val domainModule = module {
    factory { GetArticlesUseCase(get()) }
}

// ViewModel declaration
val articleModule = module {
    viewModelOf(::ArticleListViewModel)
}

// App initialization
startKoin {
    androidLogger(Level.DEBUG)
    androidContext(this@App)
    modules(dataModule, domainModule, articleModule)
}
```

### Example 5: Navigation Setup

```
Hey Claude—add navigation to my new settings screen.
Use type-safe navigation with Navigation Compose.
```

**Expected Output:**
```kotlin
// SettingsRoute.kt (api module)
@Serializable
data class SettingsRoute(val userId: String? = null)

fun NavController.navigateToSettings(userId: String? = null) {
    navigate(SettingsRoute(userId))
}

// NavGraph setup (impl module)
fun NavGraphBuilder.settingsScreen(
    onBackClick: () -> Unit,
) {
    composable<SettingsRoute> {
        SettingsRoute(onBackClick = onBackClick)
    }
}

// In NiaNavHost
settingsScreen(
    onBackClick = navController::popBackStack,
)
```

## Technology Stack

| Component | Library |
|-----------|---------|
| Language | Kotlin |
| UI | Jetpack Compose |
| Architecture | MVVM with UDF |
| DI | Koin 3.5.6 |
| Database | Room |
| Network | Retrofit + Kotlinx Serialization |
| Async | Coroutines + Flow |
| Navigation | Navigation Compose (type-safe) |

## File Templates

The skill uses these templates:

- `scripts/generate_feature.py` - Feature module generator
- `assets/templates/libs.versions.toml.template` - Gradle version catalog
- `assets/templates/settings.gradle.kts.template` - Project settings

## See Also

- [SKILL.md](SKILL.md) - Full skill documentation
- [use_koin.md](use_koin.md) - Hilt to Koin migration guide
- [references/](references/) - Detailed architecture guides
