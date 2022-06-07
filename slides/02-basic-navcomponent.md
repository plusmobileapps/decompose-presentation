#### Basic Nav Component

```kotlin [1-7|10|12]
val navController = rememberNavController()

NavHost(navController = navController, startDestination = "profile") {
    composable("profile") { Profile(/*...*/) }
    composable("friendslist") { FriendsList(/*...*/) }
    /*...*/
}

@Composable
fun Profile(navController: NavController) {
    /*...*/
    Button(onClick = { navController.navigate("friendslist") }) {
        Text(text = "Navigate next")
    }
    /*...*/
}
```