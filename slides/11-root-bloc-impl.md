#### Root Bloc Impl

```kotlin [1|3-4|5-6|7|9-17|19-24|55-61|26-27|29-32|33-40|43-47|49-54]
typealias Consumer <T> = (T) -> Unit

class RootBlocImpl(
    componentContext: ComponentContext,
    private val bottomNav: (ComponentContext, Consumer<BottomNavBloc.Output>) -> BottomNavBloc,
    private val character: (ComponentContext, Int, Consumer<CharacterDetailBloc.Output>) -> CharacterDetailBloc,
) : RootBloc, ComponentContext by componentContext {

    constructor(componentContext: ComponentContext, di: DI) : this(
        componentContext = componentContext,
        bottomNav = { context, output ->
            BottomNavBlocImpl(/*..*/)
        },
        character = { context, id, output ->
            CharacterDetailBlocImpl(/*..*/)
        }
    )

    private val router = router<Configuration, RootBloc.Child>(
        initialConfiguration = Configuration.BottomNav,
        handleBackButton = true,
        childFactory = ::createChild,
        key = "RootRouter"
    )

    override val routerState: Value<RouterState<*, RootBloc.Child>> = 
        router.state

    private fun createChild(
        configuration: Configuration,
        context: ComponentContext
    ): RootBloc.Child {
        return when (configuration) {
            Configuration.BottomNav -> RootBloc.Child.BottomNav(
                bottomNav(context, this::onCharactersOutput)
            )
            is Configuration.Character -> RootBloc.Child.Character(
                character(context, configuration.id, this::onCharacterOutput)
            )
        }
    }

    private fun onCharacterOutput(output: CharacterDetailBloc.Output) {
        when (output) {
            CharacterDetailBloc.Output.Finished -> router.pop()
        }
    }

    private fun onCharactersOutput(output: BottomNavBloc.Output) {
        when (output) {
            is BottomNavBloc.Output.ShowCharacter -> router.push(Configuration.Character(output.id))
        }
    }

    private sealed class Configuration : Parcelable {
        @Parcelize
        object BottomNav : Configuration()

        @Parcelize
        data class Character(val id: Int) : Configuration()
    }
}
```