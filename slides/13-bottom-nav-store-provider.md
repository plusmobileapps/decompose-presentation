#### Bottom Nav Store Provider

```kotlin [1-4|5-7|9|10|11-16|18-19|20-24|25|27-32|29-31|34-39|35-37|38|42|43|44-46]
class BottomNavStoreProvider(
    private val storeFactory: StoreFactory, 
    private val dispatchers: Dispatchers
) {
    private sealed class Result {
        data class NavListUpdated(val navItems: List<NavItem>) : Result()
    }

    fun create(): BottomNavigationStore =
        object : BottomNavigationStore, Store<Intent, State, Nothing> 
            by storeFactory.create(
                name = "NavigationComponentStore",
                initialState = State(),
                executorFactory = ::ExecutorImpl,
                reducer = ReducerImpl
            ) {}

    private inner class ExecutorImpl : 
        CoroutineExecutor<
            Intent,
            Nothing, // start up action 
            State, 
            Result, 
            Nothing // label
        >(dispatchers.main) {

        override fun executeIntent(intent: Intent, getState: () -> State) =
            when (intent) {
                is Intent.SelectNavItem -> {
                    selectNavItem(intent.type, getState())
                }
            }

        private fun selectNavItem(type: NavItem.Type, state: State) {
            val newList = state.navItems.map { 
                it.copy(selected = type == it.type) 
            }
            dispatch(Result.NavListUpdated(newList))
        }
    }

    private object ReducerImpl : Reducer<State, Result> {
        override fun State.reduce(msg: Result): State =
            when (msg) {
                is Result.NavListUpdated -> copy(navItems = msg.navItems)
            }
    }
}
```