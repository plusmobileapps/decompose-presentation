#### Character Detail Bloc

```kotlin [1|3-9|11-13|15-17]
interface CharacterDetailBloc {

    data class Model(
        val isLoading: Boolean = true,
        val name: String = "",
        val status: String = "",
        val species: String = "",
        val image: String = ""
    )

    sealed class Output {
        object Finished : Output()
    }

    val models: Value<Model>

    fun onBackClicked()

}
```