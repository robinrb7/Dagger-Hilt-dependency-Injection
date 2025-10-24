
# 🧠 Dependency Injection (DI) & Dagger Hilt – Notes

---

## 🚀 What is Dependency Injection?

### 💡 Definition
> Dependency Injection (DI) is a design pattern where the dependencies of a class are **provided (injected)** from outside rather than being created inside the class itself.

**Without DI (Tight Coupling):**
```kotlin
class Engine
class Car {
    private val engine = Engine() // Directly creates dependency
}


With DI (Loose Coupling):

class Engine
class Car(private val engine: Engine) // Inject dependency


Here, the Car doesn’t create its own Engine; someone else provides it.
This makes testing, scaling, and modifying dependencies much easier.

⚙️ Why Use Dagger Hilt?
❌ Manual DI

You can manually inject dependencies using constructors, but:

You must manually create and pass dependencies everywhere.

Difficult to manage shared instances (like singletons).

Hard to inject dependencies with different lifecycles (Activity, Fragment, etc.).

Tedious when the app grows (multiple Repositories, ViewModels, etc.).

Managing scopes manually is error-prone.

✅ Dagger Hilt

Hilt is a DI framework built on top of Dagger, designed specifically for Android.
It automatically provides, scopes, and injects dependencies based on lifecycle.

🧩 Benefits

Handles dependency creation automatically

Manages dependency lifecycles properly

Easier testing

Integrates with Jetpack components (@HiltViewModel, @AndroidEntryPoint)

Encourages clean architecture and modularization

🧩 Hilt Annotations Overview

Annotation	                                  Purpose
@AndroidEntryPoint	->    Marks Activity/Fragment/View that can receive injected dependencies
@HiltAndroidApp	    ->    Placed on Application class to trigger Hilt’s code generation
@Module	            ->    Marks a class that provides dependencies
@InstallIn	        ->    Specifies which component (scope) the module belongs to
@Provides	          ->    Tells Hilt how  to create a dependency (normal classes)
@Binds	            ->    Used when you have an interface + its implementation
@Named	            ->    Distinguish between multiple instances of the same type
@Assisted	          ->    Inject runtime parameters into ViewModels or classes


🧱 Components and Scopes

Components define how long a dependency lives and where it can be used.

Component	                            Scope	Lifecycle / Scope	                                                  Example Use
SingletonComponent	                      @Singleton	                                              Lives as long as the app	Network clients, databases
ActivityRetainedComponent	              @ActivityRetainedScoped	                                    Survives config changes	ViewModels
ActivityComponent	                      @ActivityScoped	                                            Lives as long as the Activity	Activity-specific UI deps
FragmentComponent	                      @FragmentScoped                                            	Lives as long as the Fragment	Fragment deps
ViewModelComponent	                    @ViewModelScoped                                          	Lives as long as the ViewModel	Repositories in VMs
ViewComponent	                          @ViewScoped                                                	Lives as long as the View	Custom Views
ServiceComponent                      	@ServiceScoped	                                            Lives as long as the Service	Background tasks



🔗 Parent–Child Relationship
SingletonComponent
   ↳ ActivityRetainedComponent
       ↳ ActivityComponent
           ↳ FragmentComponent
               ↳ ViewComponent


A child can access parent dependencies.

A parent cannot access child dependencies.
(e.g., Fragment can use Activity deps, but not vice versa.)

🧰 Types of Injection
Type	                                                                  Description	Example
Constructor Injection	(Recommended)                     Hilt creates object automatically	@Inject constructor(val repo: Repository)
Field Injection	                                        Used when you can’t use constructor injection (Activity/Fragment)	@Inject lateinit var repo: Repository
Method Injection                                      	Rarely used; injects via method	@Inject fun setLogger(logger: Logger)
Assisted Injection	                                    For runtime parameters	@Assisted in ViewModels


🧩 Typical Hilt Setup
Step 1 – Application Class
@HiltAndroidApp
class JarvisApp : Application()

Step 2 – Provide Dependencies
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit =
        Retrofit.Builder()
            .baseUrl("https://api.example.com")
            .build()
}

Step 3 – Inject into Repository
class AIRepository @Inject constructor(private val retrofit: Retrofit)

Step 4 – Inject into ViewModel
@HiltViewModel
class AIViewModel @Inject constructor(
    private val aiRepository: AIRepository
) : ViewModel()

Step 5 – Use in Compose / Activity
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val viewModel: AIViewModel = hiltViewModel()
        }
    }
}

🧩 @Binds vs @Provides
When to Use	Example
@Provides → when you can create dependency inside a function	@Provides fun provideOkHttp() = OkHttpClient()
@Binds → when binding interface to implementation	@Binds fun bindRepo(repoImpl: AIRepositoryImpl): AIRepository

@Binds must be inside an abstract class,
@Provides can be inside an object/class.

🧩 @Named Qualifier

Used when you have multiple dependencies of the same type:

@Provides
@Named("LLM2")
fun provideLLM2Retrofit(): Retrofit =
    Retrofit.Builder().baseUrl("https://llm2.api/").build()

@Provides
@Named("LLM3")
fun provideLLM3Retrofit(): Retrofit =
    Retrofit.Builder().baseUrl("https://llm3.api/").build()

class AIRepository @Inject constructor(
    @Named("LLM2") private val llm2Retrofit: Retrofit
)

🧩 @Assisted Injection

Used when constructor parameters are known at runtime, not by Hilt.

@HiltViewModel
class ChatViewModel @AssistedInject constructor(
    private val repo: ChatRepository,
    @Assisted private val sessionId: String
) : ViewModel()

⚠️ Common Pitfalls

Forgetting @AndroidEntryPoint on Activity/Fragment

Mixing manual and Hilt DI (duplicate deps)

Injecting into classes not managed by Hilt

Forgetting @HiltAndroidApp on Application class

💬 Summary
Concept                    	      Key Point
DI	              ->      Provides dependencies instead of creating them
Hilt	            ->      Manages dependencies automatically across lifecycles
Scopes	          ->      Control how long dependencies live
Modules	          ->      Define how to create dependencies
Entry Points      ->  	  Classes where dependencies are injected
Advantages	      ->      Cleaner code, less boilerplate, easier testing, lifecycle-aware
