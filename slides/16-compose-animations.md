#### Animations

```kotlin [1-6|3|8-14|11|16-30|21-26]
Children(
    routerState = bloc.routerState, 
    animation = childAnimation(slide())
) {
    // root child
}

// combine animations
Children(
    routerState = bloc.routerState, 
    animation = childAnimation(fade() + scale())
) {
    // bottom nav
}

// separate animations
@Composable
fun RootContent(bloc: RootBloc) {
    Children(
        routerState = bloc.routerState,
        animation = childAnimation { child, direction ->
            when (child.instance) {
                is RootBloc.Child.Characters -> fade() + scale()
                is RootBloc.Child.CharacterDetail -> fade() + slide()
            }
        }
    ) {
        // Omitted code
    }
}
```