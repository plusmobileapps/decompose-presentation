#### Bottom Nav Store

```kotlin [1-2|4-6|8-13]
interface BottomNavigationStore :
    Store<BottomNavigationStore.Intent, BottomNavigationStore.State, Nothing> {

    sealed class Intent {
        data class SelectNavItem(val type: NavItem.Type) : Intent()
    }

    data class State(
        val navItems: List<NavItem> = listOf(
            NavItem(true, CHARACTERS),
            NavItem(false, EPISODES),
            NavItem(false, ABOUT)
        )
    )
}
```