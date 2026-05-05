# 📁 AppBottomNav.kt (Custom Animated Bottom Navigation)

```kotlin
@Composable
fun AppBottomNav(
    navController: NavHostController,
    items: List<BottomNavItem>,
) {
    val navBackStackEntry by navController.currentBackStackEntryAsState()
    val currentDestination = navBackStackEntry?.destination
    val route = currentDestination?.route
    val currentRoute = bottomNavController.currentBackStackEntryAsState().value?.destination?.route

    var isBottomBarVisible by remember { mutableStateOf(true) }

 LaunchedEffect(currentRoute) {

        isBottomBarVisible = when (currentRoute) {
            HomeRoutes.HomeRoute::class.qualifiedName -> {
                true
            }

    
            SettingsRoutes.SettingRoute::class.qualifiedName -> {
                true
            }

            else -> {
                false
            }
        }
    }

    
    AnimatedVisibility(
        visible = isBottomBarVisible,
        enter = fadeIn() + slideInVertically { it / 2 },
        exit = fadeOut() + slideOutVertically { it / 2 }
    ) {
        CustomBottomBar(
            items = items,
            currentDestination = currentDestination,
            onItemSelected = { selectedItem ->
                navController.navigate(selectedItem.route ?: "") {
                    popUpTo(navController.graph.findStartDestination().id) {
                        saveState = true
                    }
                    launchSingleTop = true
                    restoreState = true
                }
            }
        )
    }
}
```

---

# 📁 CustomBottomBar.kt (UI Layer)

```kotlin
@Composable
fun CustomBottomBar(
    items: List<BottomNavItem>,
    currentDestination: NavDestination?,
    onItemSelected: (BottomNavItem) -> Unit,
) {
    Box(
        modifier = Modifier
            .background(Color.White)
            .clickable(onClick = {}, interactionSource = null, indication = null)
    ) {
        // Top divider shadow
        Canvas(
            modifier = Modifier
                .fillMaxWidth()
                .height(1.dp)
                .align(Alignment.TopCenter)
        ) {
            drawRect(
                brush = Brush.verticalGradient(
                    colors = listOf(Color.Transparent, gray.copy(alpha = .3f)),
                    startY = 0f,
                    endY = size.height
                )
            )
        }

        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(top = 10.dp)
                .padding(horizontal = 8.dp),
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.SpaceEvenly
        ) {
            items.forEach { navItem ->

                val isSelected =
                    currentDestination?.hierarchy?.any { it.route == navItem.route } == true

                Column(
                    modifier = Modifier
                        .weight(1f)
                        .padding(vertical = 8.dp),
                    horizontalAlignment = Alignment.CenterHorizontally,
                    verticalArrangement = Arrangement.Center
                ) {

                    Icon(
                        painter = painterResource(navItem.selectedIcon),
                        contentDescription = navItem.title,
                        tint = if (isSelected) lightIndigo else gray,
                        modifier = Modifier
                            .size(24.dp)
                            .clickable(
                                interactionSource = remember { MutableInteractionSource() },
                                indication = null
                            ) {
                                onItemSelected(navItem)
                            }
                    )

                    Spacer(Modifier.height(4.dp))

                    Text(
                        text = navItem.title,
                        color = if (isSelected) lightIndigo else gray,
                        fontSize = 11.sp,
                        fontWeight = if (isSelected) FontWeight.SemiBold else FontWeight.Medium,
                        style = MaterialTheme.typography.bodyMedium,
                        maxLines = 1,
                        overflow = TextOverflow.Ellipsis
                    )

                    Spacer(Modifier.height(6.dp))

                    if (isSelected) {
                        Box(
                            modifier = Modifier
                                .width(40.dp)
                                .height(4.dp)
                                .clip(RoundedCornerShape(topStart = 8.dp, topEnd = 8.dp))
                                .background(lightIndigo)
                        )
                    }
                }
            }
        }
    }
}
```

---

# ✅ Behavior Summary

* Bottom bar visibility is controlled dynamically based on route
* Animated show/hide using `AnimatedVisibility`
* State-safe navigation using:

  * `popUpTo`
  * `launchSingleTop`
  * `restoreState`
* Selection handled using `NavDestination.hierarchy`

---

# 🚀 Notes

* Keep route naming consistent (Graph vs Screen)
* Avoid hardcoded string checks for large projects (use route constants if scaling)
* This implementation is production-ready for medium-scale apps
