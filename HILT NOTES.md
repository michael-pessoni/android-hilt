# Hilt

### Gradle dependencies

build.gradle (root)

```groovy
buildscript {
    ...
    ext.hilt_version = '2.28-alpha'
    dependencies {
        ...
        classpath "com.google.dagger:hilt-android-gradle-plugin:$hilt_version"
    }
}
```

build.gradle (app)

```groovy
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
    ...
}
```

```
dependencies {
    ...
    implementation "com.google.dagger:hilt-android:$hilt_version"
    kapt "com.google.dagger:hilt-android-compiler:$hilt_version"
}
```

### Annotations

#### @HiltAndroidApp

Add @HiltAndroidApp annotation to the Application class. It triggers Hilt's code generation, including a base class for your application that can use dependency injection.

#### @AndroidEntryPoint

Adds a DI container to the Android class annotaded with it. Allows to use members injection in Android classes which you don't have access to contructor.You can use @AndroidEntryPoint on the following types:

    Activity
    Fragment
    View
    Service
    BroadcastReceiver

#### @HiltViewModel

To enable injection of a ViewModel by Hilt use the @HiltViewModel annotation and @Inject before it's constructor.

```kotlin
@HiltViewModel
class FooViewModel @Inject constructor(
  val handle: SavedStateHandle,
  val foo: Foo
) : ViewModel
```

To instatiate the ViewModel the activity or fragment annotaded with @AndroidEntryPoint can cal `by viewModels()`

```kotlin
@AndroidEntryPoint
class MyActivity : AppCompatActivity() {
  private val fooViewModel: FooViewModel by viewModels()
}
```

#### @Module

Use @Module to annotate the class in which you will add bindings for types that cannot be constructor injected. You can use **@InstallIn** to determine in which Hilt component the module will be installed into.

Inside the module you can tell Hilt how to instantiate a type using **@Provides** and **@Binds** annotations.

#### @Provides

Adds a binding for a type that cannot be constructor injected:
- Return type is the binding type.

- Parameters are dependencies.

- Every time an instance is needed, the
  function body is executed if the type is
  not scoped.

  

```kotlin
@InstallIn(SingletonComponent::class)
@Module
class AnalyticsModule {
    @Provides
    fun providesAnalyticsService(
        converterFactory: GsonConverterFactory
        ): AnalyticsService {
            return Retrofit.Builder()
                .baseUrl("https://example.com")
                .addConverterFactory(converterFactory)
                .build()
                .create(AnalyticsService::class.java)
    }
}
```



#### @Binds

Shorthand for binding an interface type:
- Methods must be in a module.
- @Binds annotated methods must be
abstract.
- Return type is the binding type.
- Parameter is the implementation type.

```kotlin
@InstallIn(SingletonComponent::class)
@Module
abstract class AnalyticsModule {
    @Binds
    abstract fun bindsAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
    ): AnalyticsService
}
```

