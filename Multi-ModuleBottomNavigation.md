# 📱 Multi-Module Bottom Navigation Architecture (Jetpack Compose)

This project follows a **modular navigation architecture** using **Jetpack Compose Navigation (type-safe routes)**.

#

---

# 📁 RootNavigation.kt

```kotlin
@Composable
fun RootNavigation() {
    val rootNavController = rememberNavController()

    NavHost(
        navController = rootNavController,
        startDestination = AuthRouts.AuthGraph
    ) {

        authNavigation(
            navController = rootNavController,
            onNavigateHome = {
                rootNavController.navigate(Dashboard) {
                    popUpTo(AuthRouts.AuthGraph) {
                        inclusive = true
                    }
                }
            }
        )

        composable<Dashboard> {
            DashboardScreen(
                onLogout = {
                    rootNavController.navigate(AuthRouts.SignInRoute) {
                        popUpTo(0) { inclusive = true }
                    }
                }
            )
        }
    }
}
```

---

# 📁 DashboardScreen.kt

```kotlin
@Composable
fun DashboardScreen(
    onLogout: () -> Unit
) {
    val bottomNavController = rememberNavController()

    Scaffold(
        bottomBar = {
            DashboardBottomBar(navController = bottomNavController)
        }
    ) {
        DashboardNavHost(
            navController = bottomNavController,
            onLogout = onLogout
        )
    }
}
```

---

# 📁 DashboardNavHost.kt (Previously BottomNavHost)

```kotlin
@Composable
fun DashboardNavHost(
    navController: NavHostController,
    onLogout: () -> Unit
) {

    NavHost(
        navController = navController,
        startDestination = HomeRoutes.HomeGraph
    ) {

        homeNavigation(
            navController = navController,
            onProfileClick = { employeeId, userType ->
                navController.navigate(
                    MyTeamRoutes.ProfileRoute(employeeId, userType)
                )
            },
            onItemClick = { quickAction ->
                when (quickAction) {
                    GlobalUtils.QuickActionType.APPLY_LEAVE -> {
                        navController.navigate(
                            LeaveRoutes.RequestLeaveRoute("")
                        )
                    }
                    else -> Unit
                }
            }
        )

 

        settingNavigation(
            navController = navController,
            logOut = onLogout,
            isFromHome = true,
            onProfileClick = { employeeId, userType ->
                navController.navigate(
                    MyTeamRoutes.ProfileRoute(employeeId, userType)
                )
            },
            onBackClick = {
                navController.navigate(HomeRoutes.HomeRoute) {
                    popUpTo(navController.graph.findStartDestination().id) {
                        saveState = true
                    }
                    restoreState = true
                }
            }
        )
    }
}
```

---

# 📁 BottomNavItem.kt

```kotlin
sealed class BottomNavItem(
    val route: String?,
    val title: String
) {

    data object Home : BottomNavItem(
        route = HomeRoutes.HomeGraph::class.qualifiedName,
        title = "Home"
    )

   

    data object Settings : BottomNavItem(
        route = SettingsRoutes.SettingGraph::class.qualifiedName,
        title = "Settings"
    )

 
}
```

---

# 📁 DashboardBottomBar.kt (Previously BottomBar)

```kotlin
@Composable
fun DashboardBottomBar(navController: NavHostController) {

    val bottomNavItems = listOf(
        BottomNavItem.Home,
        BottomNavItem.Settings
    )

    NavigationBar {
        bottomNavItems.forEach { navItem ->

            NavigationBarItem(
                selected = false,
                onClick = {
                    navController.navigate(navItem.route!!) {

                        popUpTo(navController.graph.findStartDestination().id) {
                            saveState = true
                        }

                        launchSingleTop = true
                        restoreState = true
                    }
                },
                icon = { Icon(Icons.Default.Home, contentDescription = null) },
                label = { Text(navItem.title) }
            )
        }
    }
}
```

---

# 📁   SettingRoutes.kt

```kotlin
@Serializable
object  SettingRoutes {

    @Serializable
    object  SettingGraph

    @Serializable
    object  SettingRoute


}
```

---

# 📁   SettingNavigation.kt

```kotlin
fun NavGraphBuilder.leaveNavigation(
    navController: NavController,
    navigateToProfile: (String, Int) -> Unit
) {

    navigation<LeaveRoutes.  SettingGraph>(
        startDestination =  SettingRoutes.SettingRoute
    ) {

        composable<  SettingRoutes. SettingRoute> {
            LeaveScreen()
        }
    }
}
```

---

|   |
| - |

---
