# Android 开发 Skill - 使用指南

一个用于构建 Android 应用的生产级 Skill，支持 Kotlin、Jetpack Compose 和 Koin 依赖注入。

## 使用场景

在处理以下 Android 开发任务时使用此 Skill：

- 创建新的 Android 项目或模块
- 使用 Jetpack Compose 构建屏幕
- 设置 MVVM 架构
- 实现 Repository 和数据源
- 使用 Koin 配置依赖注入
- 使用 Room 数据库
- 设置类型安全的路由导航

## 快速参考

### 创建新功能模块

```
嘿 Claude，创建一个用户设置的新功能模块。
Package: com.example.app
Path: /path/to/project
```

**结果**: 生成 `feature/settings/api` 和 `feature/settings/impl`，包含：
- 导航路由和键
- Compose 屏幕
- ViewModel（带 Koin 注入）
- Koin DI 模块

### 构建 Compose 屏幕

```
嘿 Claude，构建一个带 MVVM 模式的任务列表屏幕。
包含加载、成功和错误状态。
```

**结果**: 创建屏幕 composable、ViewModel、UiState 和状态管理。

### 设置 Repository 模式

```
嘿 Claude，为文章创建一个离线优先的 Repository。
使用 Room 缓存并暴露 Flow<T>。
```

**结果**: 创建 Repository 接口、使用 Room DAO 的实现、数据源。

### 配置依赖注入

```
嘿 Claude，为我的数据层添加新的 Koin 模块。
需要注入 Repository 和 NetworkApi。
```

**结果**: 创建包含正确 `viewModel` 和 `single` 声明的 Koin 模块。

## 使用示例

### 示例 1：基础功能模块

```
嘿 Claude，我刚添加了 "android-development" skill。
能否为用户认证创建一个新功能模块？
Package: com.myapp.auth
Path: /Users/me/projects/MyApp
```

**预期输出：**
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

### 示例 2：Compose UI 屏幕

```
嘿 Claude，我需要帮助构建任务列表屏幕。
使用 Jetpack Compose 和 MVVM 模式。
```

**预期输出：**
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

### 示例 3：Repository 实现

```
嘿 Claude，为文章创建一个离线优先的 Repository。
使用 Room 缓存并暴露响应式流。
```

**预期输出：**
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
        // 网络获取 + Room 缓存
    }
}
```

### 示例 4：Koin DI 配置

```
嘿 Claude，我需要设置 Koin 依赖注入。
为我的数据层和领域层创建模块。
```

**预期输出：**
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

// ViewModel 声明
val articleModule = module {
    viewModelOf(::ArticleListViewModel)
}

// App 初始化
startKoin {
    androidLogger(Level.DEBUG)
    androidContext(this@App)
    modules(dataModule, domainModule, articleModule)
}
```

### 示例 5：导航设置

```
嘿 Claude，为我的新设置屏幕添加导航。
使用 Navigation Compose 的类型安全导航。
```

**预期输出：**
```kotlin
// SettingsRoute.kt (api 模块)
@Serializable
data class SettingsRoute(val userId: String? = null)

fun NavController.navigateToSettings(userId: String? = null) {
    navigate(SettingsRoute(userId))
}

// NavGraph 设置 (impl 模块)
fun NavGraphBuilder.settingsScreen(
    onBackClick: () -> Unit,
) {
    composable<SettingsRoute> {
        SettingsRoute(onBackClick = onBackClick)
    }
}

// 在 NiaNavHost 中
settingsScreen(
    onBackClick = navController::popBackStack,
)
```

## 技术栈

| 组件 | 库 |
|------|-----|
| 语言 | Kotlin |
| UI | Jetpack Compose |
| 架构 | MVVM + UDF |
| DI | Koin 3.5.6 |
| 数据库 | Room |
| 网络 | Retrofit + Kotlinx Serialization |
| 异步 | Coroutines + Flow |
| 导航 | Navigation Compose (类型安全) |

## 文件模板

此 Skill 使用以下模板：

- `scripts/generate_feature.py` - 功能模块生成器
- `assets/templates/libs.versions.toml.template` - Gradle 版本目录
- `assets/templates/settings.gradle.kts.template` - 项目设置

## 参见

- [SKILL.md](SKILL.md) - 完整 Skill 文档
- [use_koin.md](use_koin.md) - Hilt 到 Koin 迁移指南
- [README_USE.md](README_USE.md) - 英文版使用指南
- [references/](references/) - 详细架构指南
