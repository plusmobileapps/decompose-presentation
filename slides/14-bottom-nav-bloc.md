#### Bottom Nav Bloc Impl

```kotlin [1-2|4|5-7|8-10|11|13-19|21-23|25-27|82-91|29-37|39|40|41|42-44|47|50-57|59-60|61-65|69-80|74-75]
typealias CharactersBlocBuilder = 
    (ComponentContext, Consumer<CharactersBloc.Output>) -> CharactersBloc

class BottomNavBlocImpl(
    componentContext: ComponentContext,
    storeFactory: StoreFactory,
    dispatchers: Dispatchers,
    private val charactersBloc: CharactersBlocBuilder,
    private val episodesBloc: (ComponentContext) -> EpisodesBloc,
    private val bottomNavOutput: Consumer<BottomNavBloc.Output>
) : BottomNavBloc, ComponentContext by componentContext {

    constructor(
        componentContext: ComponentContext,
        di: DI,
        output: Consumer<BottomNavBloc.Output>
    ) : this(
        /* inject dependencies */
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

    private val routerSubscriber: (RouterState<Configuration, Child>) -> Unit = {
        val intent = BottomNavigationStore.Intent.SelectNavItem(
            when (it.activeChild.instance) {
                is Child.Characters -> NavItem.Type.CHARACTERS
                is Child.Episodes -> NavItem.Type.EPISODES
                is Child.About -> NavItem.Type.ABOUT
            }
        )
        store.accept(intent)
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