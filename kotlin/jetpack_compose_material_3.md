# Show me how to build all the common UI elements that an app needs using Jetpack Compose Material 3

## Mastering Material 3 in Jetpack Compose: A Guide to Building Common UI Elements

Creating a modern and visually appealing Android application is streamlined with Jetpack Compose and Material 3, Google's latest design system. This guide will walk you through the implementation of the most common UI elements your app will need, complete with code snippets and explanations to get you up and running quickly.

### Getting Started: Setting Up Your Project

Before you can start building your UI, you need to add the Jetpack Compose Material 3 dependency to your app's `build.gradle.kts` (or `build.gradle`) file:

```kotlin
dependencies {
    implementation("androidx.compose.material3:material3:1.2.1")
}
```

With the dependency in place, you can set up a Material 3 theme for your application. This is typically done in a `Theme.kt` file. A key feature of Material 3 is dynamic coloring, which adapts the app's color scheme to the user's wallpaper on Android 12 and above.

```kotlin
@Composable
fun YourAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) {
        darkColorScheme()
    } else {
        lightColorScheme()
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography, // Your defined typography
        content = content
    )
}
```

### The Foundation: Scaffold

The `Scaffold` composable provides a basic layout structure for your app, with slots for common UI elements like top app bars, bottom app bars, floating action buttons, and drawers.

```kotlin
Scaffold(
    topBar = { /* TopAppBar goes here */ },
    bottomBar = { /* BottomAppBar or NavigationBar goes here */ },
    floatingActionButton = { /* FloatingActionButton goes here */ }
) { innerPadding ->
    // Your screen content goes here, using innerPadding
    Column(modifier = Modifier.padding(innerPadding)) {
        // ...
    }
}
```

### Top-Level Navigation

**Top App Bar:** The `TopAppBar` displays information and actions related to the current screen. Material 3 offers several types of top app bars, including `CenterAlignedTopAppBar`, `MediumTopAppBar`, and `LargeTopAppBar`.

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MyTopAppBar() {
    CenterAlignedTopAppBar(
        title = { Text("My App") },
        navigationIcon = {
            IconButton(onClick = { /* Handle navigation icon press */ }) {
                Icon(Icons.Filled.Menu, contentDescription = "Menu")
            }
        },
        actions = {
            IconButton(onClick = { /* Handle action icon press */ }) {
                Icon(Icons.Filled.Search, contentDescription = "Search")
            }
        }
    )
}
```

**Bottom App Bar:** A `BottomAppBar` provides access to navigation and key actions at the bottom of the screen.

```kotlin
@Composable
fun MyBottomAppBar() {
    BottomAppBar(
        actions = {
            IconButton(onClick = { /* doSomething() */ }) {
                Icon(Icons.Filled.Share, contentDescription = "Share")
            }
            IconButton(onClick = { /* doSomething() */ }) {
                Icon(Icons.Filled.Favorite, contentDescription = "Favorite")
            }
        }
    )
}
```

**Navigation Bar:** For bottom navigation, Material 3 provides the `NavigationBar` and `NavigationBarItem` composables.

```kotlin
@Composable
fun MyNavigationBar() {
    var selectedItem by remember { mutableStateOf(0) }
    val items = listOf("Home", "Favorites", "Profile")
    val icons = listOf(Icons.Filled.Home, Icons.Filled.Favorite, Icons.Filled.Person)

    NavigationBar {
        items.forEachIndexed { index, item ->
            NavigationBarItem(
                icon = { Icon(icons[index], contentDescription = item) },
                label = { Text(item) },
                selected = selectedItem == index,
                onClick = { selectedItem = index }
            )
        }
    }
}
```

### Key Actions: Floating Action Button

The `FloatingActionButton` (FAB) is used for a primary, promoted action. Material 3 offers different sizes: `FloatingActionButton`, `SmallFloatingActionButton`, and `LargeFloatingActionButton`.

```kotlin
@Composable
fun MyFloatingActionButton() {
    FloatingActionButton(onClick = { /* Handle FAB click */ }) {
        Icon(Icons.Filled.Add, contentDescription = "Add")
    }
}

@Composable
fun MyExtendedFloatingActionButton() {
    ExtendedFloatingActionButton(
        onClick = { /* Handle FAB click */ },
        icon = { Icon(Icons.Filled.Edit, contentDescription = "Edit") },
        text = { Text("Edit") }
    )
}
```

### Input Controls

**Buttons:** Material 3 provides five distinct button types for different levels of emphasis: `Button` (filled), `FilledTonalButton`, `OutlinedButton`, `ElevatedButton`, and `TextButton`.

```kotlin
// Filled Button
Button(onClick = { /* ... */ }) {
    Text("Submit")
}

// Outlined Button
OutlinedButton(onClick = { /* ... */ }) {
    Text("Cancel")
}

// Text Button
TextButton(onClick = { /* ... */ }) {
    Text("Learn More")
}
```

**Text Fields:** For user text input, you can use `TextField` for a filled style or `OutlinedTextField` for a bordered style.

```kotlin
@Composable
fun MyTextField() {
    var text by remember { mutableStateOf("") }

    TextField(
        value = text,
        onValueChange = { text = it },
        label = { Text("Email") }
    )
}

@Composable
fun MyOutlinedTextField() {
    var text by remember { mutableStateOf("") }

    OutlinedTextField(
        value = text,
        onValueChange = { text = it },
        label = { Text("Password") }
    )
}
```

**Selection Controls:** These allow users to make selections.

*   **Checkbox:** For selecting one or more options from a list.

    ```kotlin
    @Composable
    fun MyCheckbox() {
        var checked by remember { mutableStateOf(true) }
        Row(verticalAlignment = Alignment.CenterVertically) {
            Checkbox(
                checked = checked,
                onCheckedChange = { checked = it }
            )
            Text("I agree to the terms")
        }
    }
    ```
*   **Radio Button:** For selecting a single option from a set.

    ```kotlin
    @Composable
    fun MyRadioButton() {
        val radioOptions = listOf("Option 1", "Option 2", "Option 3")
        val (selectedOption, onOptionSelected) = remember { mutableStateOf(radioOptions[0]) }

        Column {
            radioOptions.forEach { text ->
                Row(verticalAlignment = Alignment.CenterVertically) {
                    RadioButton(
                        selected = (text == selectedOption),
                        onClick = { onOptionSelected(text) }
                    )
                    Text(text = text)
                }
            }
        }
    }
    ```
*   **Switch:** To toggle a single setting on or off.

    ```kotlin
    @Composable
    fun MySwitch() {
        var checked by remember { mutableStateOf(true) }
        Switch(
            checked = checked,
            onCheckedChange = { checked = it }
        )
    }
    ```

**Slider:** Sliders allow users to make selections from a range of values.

```kotlin
@Composable
fun MySlider() {
    var sliderPosition by remember { mutableFloatStateOf(0f) }
    Slider(
        value = sliderPosition,
        onValueChange = { sliderPosition = it }
    )
}
```

### Containers

**Cards:** Cards are used to group related content and actions. Material 3 offers filled, elevated, and outlined card styles.

```kotlin
// Filled Card
Card(modifier = Modifier.padding(16.dp)) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Filled Card Title")
        Text("This is the content of the card.")
    }
}

// Outlined Card
OutlinedCard(modifier = Modifier.padding(16.dp)) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Outlined Card Title")
        Text("This is the content of the card.")
    }
}
```

**Dialogs:** Dialogs are small windows that prompt the user to make a decision or enter additional information.

```kotlin
@Composable
fun MyAlertDialog() {
    var showDialog by remember { mutableStateOf(false) }

    if (showDialog) {
        AlertDialog(
            onDismissRequest = { showDialog = false },
            title = { Text("Dialog Title") },
            text = { Text("This is the message of the dialog.") },
            confirmButton = {
                TextButton(onClick = { showDialog = false }) {
                    Text("Confirm")
                }
            },
            dismissButton = {
                TextButton(onClick = { showDialog = false }) {
                    Text("Dismiss")
                }
            }
        )
    }
    Button(onClick = { showDialog = true }) {
        Text("Show Dialog")
    }
}
```

**Modal Bottom Sheet:** A `ModalBottomSheet` is a surface anchored to the bottom of the screen that can be swiped up to reveal more content.

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MyModalBottomSheet() {
    var showSheet by remember { mutableStateOf(false) }

    if (showSheet) {
        ModalBottomSheet(onDismissRequest = { showSheet = false }) {
            // Sheet content
            Column(modifier = Modifier.padding(16.dp)) {
                Text("Sheet Content")
                Button(onClick = { showSheet = false }) {
                    Text("Close Sheet")
                }
            }
        }
    }

    Button(onClick = { showSheet = true }) {
        Text("Show Bottom Sheet")
    }
}
```

### Progress Indicators

Progress indicators inform users about the status of ongoing processes.

*   **Circular Progress Indicator:**

    ```kotlin
    @Composable
    fun MyCircularProgressIndicator() {
        CircularProgressIndicator()
    }
    ```
*   **Linear Progress Indicator:**

    ```kotlin
    @Composable
    fun MyLinearProgressIndicator() {
        var progress by remember { mutableFloatStateOf(0.1f) }
        LinearProgressIndicator(progress = { progress })
    }
    ```

By utilizing these fundamental Material 3 components in Jetpack Compose, you can build a robust and aesthetically pleasing user interface that adheres to modern Android design principles. Remember to explore the extensive customization options available for each component to perfectly match your app's branding and user experience goals.
