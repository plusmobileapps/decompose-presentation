#### Observing Values

```kotlin [3-5|6-14|19-22|23|24|25-38]
@Composable
fun BottomNavUI(bloc: BottomNavBloc) {
    Scaffold(bottomBar = {
        BottomNavigationBar(bloc.models, bloc::onNavItemClicked)
    }) { paddingValues ->
        Children(routerState = bloc.routerState) {
            when (val child = it.instance) {
                is BottomNavBloc.Child.Characters -> 
                    CharactersUI(bloc = child.bloc, paddingValues)
                is BottomNavBloc.Child.Episodes -> 
                    EpisodesUI(bloc = child.bloc, paddingValues)
                is BottomNavBloc.Child.About -> AboutUI()
            }
        }
    }
}

@Composable
fun BottomNavigationBar(
    models: Value<BottomNavBloc.Model>, 
    onClick: (BottomNavBloc.NavItem) -> Unit
) {
    val model = models.subscribeAsState()
    val items = model.value.navItems
    NavigationBar {
        items.forEach {
            NavigationBarItem(
                selected = it.selected,
                onClick = { onClick(it) },
                icon = {
                    Icon(
                        painter = rememberAsyncImagePainter(it.type.icon),
                        contentDescription = null
                    )
                },
                label = { Text(stringResource(id = it.type.label)) }
            )
        }
    }
}
```