#### Bottom Nav Store Provider

```kotlin [1-4|5-7|9-16|18-19|21-26|28-33|36-41]
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
        CoroutineExecutor<Intent, Nothing, State, Result, Nothing>(dispatchers.main) {

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