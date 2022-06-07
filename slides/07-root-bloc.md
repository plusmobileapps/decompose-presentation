#### Root Bloc

```kotlin [|4-7|2]
interface RootBloc {
    val routerState: Value<RouterState<*, Child>>

    sealed class Child {
        data class BottomNav(val bloc: BottomNavBloc) : Child()
        data class Character(val bloc: CharacterDetailBloc) : Child()
    }
}
```