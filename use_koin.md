# 从 Hilt 迁移到 Koin 修改总结

本文档记录了将 Android 开发 Skill 模板的依赖注入框架从 Hilt 改为 Koin 的所有修改。

## 修改的文件

| 文件 | 修改内容 |
|------|----------|
| `assets/templates/libs.versions.toml.template` | Hilt 依赖改为 Koin |
| `SKILL.md` | ViewModel/Screen 模式改为 Koin 语法 |
| `scripts/generate_feature.py` | 新增 Koin 模块生成，移除 Hilt 注解 |
| `README.md` | 技术栈说明更新为 Koin |
| `references/gradle-setup.md` | 添加 Koin 初始化示例 |
| `references/architecture.md` | ViewModel/Repository 移除 Hilt 注解 |
| `references/compose-patterns.md` | ViewModel 获取方式改为 Koin |
| `references/modularization.md` | 模块结构说明更新 |
| `references/testing.md` | UI 测试改为 KoinTestRule |

## 主要变化

### 1. 依赖配置 (libs.versions.toml)

**Hilt (旧)**
```toml
hilt = "2.50"
androidxHiltNavigationCompose = "1.1.0"

hilt-android = { group = "com.google.dagger", name = "hilt-android" }
hilt-android-compiler = { group = "com.google.dagger", name = "hilt-android-compiler" }
hilt-android-testing = { group = "com.google.dagger", name = "hilt-android-testing" }

hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

**Koin (新)**
```toml
koin = "3.5.6"

koin-android = { group = "io.insert-koin", name = "koin-android" }
koin-androidx-compose = { group = "io.insert-koin", name = "koin-androidx-compose" }
koin-androidx-workmanager = { group = "io.insert-koin", name = "koin-androidx-workmanager" }

koin = { id = "io.insert-koin.koin", version.ref = "koin" }
```

### 2. ViewModel 声明

**Hilt (旧)**
```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: MyRepository,
) : ViewModel()
```

**Koin (新)**
```kotlin
class MyViewModel(
    private val repository: MyRepository,
) : ViewModel()
```

### 3. Compose 中获取 ViewModel

**Hilt (旧)**
```kotlin
@Composable
fun MyRoute(viewModel: MyViewModel = hiltViewModel()) { }
```

**Koin (新)**
```kotlin
@Composable
fun MyRoute(viewModel: MyViewModel = koinViewModel()) { }
```

### 4. Koin 模块声明

**generate_feature.py 生成的新模板**
```kotlin
import org.koin.androidx.viewmodel.dsl.viewModelOf
import org.koin.dsl.module

val featureModule = module {
    viewModelOf(::FeatureViewModel)
}
```

### 5. Koin 初始化

**App.kt**
```kotlin
class App : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidLogger(Level.DEBUG)
            androidContext(this@App)
            modules(appModule, featureModule)
        }
    }
}
```

### 6. AndroidManifest.xml
```xml
<application
    android:name=".App"
    ...>
</application>
```

### 7. ViewModel 与 SavedStateHandle

**Hilt (旧)**
```kotlin
@HiltViewModel
class DetailViewModel @Inject constructor(
    private val savedStateHandle: SavedStateHandle,
) : ViewModel()
```

**Koin (新)**
```kotlin
class DetailViewModel(
    private val savedStateHandle: SavedStateHandle,
) : ViewModel()

// Koin 模块
val detailModule = module {
    viewModel { DetailViewModel(get()) }
}
```

## 关键 API 对照

| 用途 | Hilt | Koin |
|------|------|------|
| ViewModel 注解 | `@HiltViewModel` | 无 (普通 class) |
| 构造函数注入 | `@Inject` | 无 |
| ViewModel 获取 | `hiltViewModel()` | `koinViewModel()` |
| 模块声明 | `@Module`, `@InstallIn` | `module { }` |
| ViewModel 绑定 | `@Binds`, `@IntoMap` | `viewModel { }` 或 `viewModelOf()` |

## 测试配置

**Hilt (旧)**
```kotlin
@HiltAndroidTest
@RunWith(AndroidJUnit4::class)
class MyTest {
    @get:Rule val hiltRule = HiltAndroidRule(this)
}
```

**Koin (新)**
```kotlin
@RunWith(AndroidJUnit4::class)
class MyTest {
    private val koinRule = KoinTestRule()

    @Before
    fun setup() {
        koinRule.startKoin { modules(testModule) }
    }

    @After
    fun teardown() {
        koinRule.stopKoin()
    }
}
```

## 版本信息

- **Koin 版本**: 3.5.6 (LTS 版本，支持至 2025 年 12 月)
- **Kotlin 版本**: 1.9.22
- **兼容性**: ✅ 完全兼容

## 后续步骤

1. 运行 `python scripts/generate_feature.py <feature-name> --package <package> --path <path>` 生成新功能模块
2. 在 App 的 `onCreate()` 中调用 `startKoin()` 初始化
3. 使用 `koinViewModel()` 在 Compose 中获取 ViewModel
4. 将模块添加到 `startKoin` 的 modules 列表中
