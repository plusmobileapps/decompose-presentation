#### Value

```kotlin []
abstract class Value<out T : Any> {

    abstract val value: T

    abstract fun subscribe(observer: ValueObserver<T>)

    abstract fun unsubscribe(observer: ValueObserver<T>)
}
```