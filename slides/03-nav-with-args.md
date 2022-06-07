#### Navigating With Arguments

```kotlin [2|2-3|7-8]
NavHost(startDestination = "profile/{userId}") {
    composable("profile/{userId}") { backStackEntry ->
        Profile(navController, backStackEntry.arguments?.getString("userId"))
    }
}

// in another composable
navController.navigate("profile/user1234")
```