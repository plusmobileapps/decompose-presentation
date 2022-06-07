#### Bottom Nav Bloc Impl

```kotlin [1|2-7|8|10-13|14-25|27-29|31-33|35-40|90-99|42-43|45-46|47-56|58-65|67-75|77-79|80-88]
class BottomNavBlocImpl(
    componentContext: ComponentContext,
    storeFactory: StoreFactory,
    dispatchers: Dispatchers,
    private val charactersBloc: (ComponentContext, Consumer<CharactersBloc.Output>) -> CharactersBloc,
    private val episodesBloc: (ComponentContext) -> EpisodesBloc,
    private val bottomNavOutput: Consumer<BottomNavBloc.Output>
) : BottomNavBloc, ComponentContext by componentContext {

    constructor(
        componentContext: ComponentContext,
        di: DI,
        output: Consumer<BottomNavBloc.Output>
    ) : this(
        componentContext = componentContext,
        storeFactory = di.storeFactory,
        dispatchers = di.dispatchers,
        charactersBloc = { context, characterOutput ->
            CharactersBlocImpl(/*..*/)
        },
        episodesBloc = { context, episodesOutput ->
            EpisodesBlocImpl(/*..*/)
        },
        bottomNavOutput = output
    )

    private val store: BottomNavigationStore = instanceKeeper.getStore {
        BottomNavStoreProvider(storeFactory, dispatchers).create()
    }

    override val models: Value<BottomNavBloc.Model> = store.asValue().map {
        BottomNavBloc.Model(it.navItems)
    }

    private val router = router<Configuration, BottomNavBloc.Child>(
        initialConfiguration = Configuration.Characters,
        handleBackButton = true,
        childFactory = ::createChild,
        key = "BottomNavRouter"
    )

    override val routerState: Value<RouterState<*, BottomNavBloc.Child>> = 
        router.state

    private val routerSubscriber: 
        (RouterState<Configuration, BottomNavBloc.Child>) -> Unit = {
            store.accept(
                BottomNavigationStore.Intent.SelectNavItem(
                    when (it.activeChild.instance) {
                        is Child.Characters -> NavItem.Type.CHARACTERS
                        is Child.Episodes -> NavItem.Type.EPISODES
                        is Child.About -> NavItem.Type.ABOUT
                    }
                )
            )
        }

    init {
        lifecycle.doOnResume {
            router.state.subscribe(routerSubscriber)
        }
        lifecycle.doOnPause {
            router.state.unsubscribe(routerSubscriber)
        }
    }

    override fun onNavItemClicked(item: NavItem) {
        router.bringToFront(
            when (item.type) {
                NavItem.Type.CHARACTERS -> Configuration.Characters
                NavItem.Type.EPISODES -> Configuration.Episodes
                NavItem.Type.ABOUT -> Configuration.About
            }
        )
    }

    private fun createChild(
        configuration: Configuration,
        context: ComponentContext
    ): BottomNavBloc.Child = when (configuration) {
        Configuration.Characters -> BottomNavBloc.Child.Characters(
            charactersBloc(context, this::onCharactersBlocOutput)
            //omitted output function that just delegates to bottom nav output
        )
        Configuration.Episodes -> 
            BottomNavBloc.Child.Episodes(episodesBloc(context))
        Configuration.About -> BottomNavBloc.Child.About
    }

    sealed class Configuration : Parcelable {
        @Parcelize
        object Characters : Configuration()

        @Parcelize
        object Episodes : Configuration()

        @Parcelize
        object About : Configuration()
    }
}
```