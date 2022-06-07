#### Characters Bloc

```kotlin [1|3-7|9-13|15-17]
interface CharactersBloc {

    data class Model(
        val characters: List<RickAndMortyCharacter> = emptyList(),
        val error: String? = null,
        val isLoading: Boolean = false
    )

    sealed class Output {
        data class OpenCharacter(
            val character: RickAndMortyCharacter
        ) : Output()
    }

    val models: Value<Model>

    fun onCharacterClicked(character: RickAndMortyCharacter)

}
```