# build a jetpack compose android app for todo list. It should use sqlite database and use flow concepts.

Let's build a Todo List app using Jetpack Compose, SQLite (via Room Persistence Library), and Kotlin Flows!

Here's a breakdown of the app structure and code, followed by explanations:

**1. Project Setup (build.gradle files)**

First, you'll need to add the necessary dependencies to your `build.gradle (Module :app)` file.

```gradle
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt' // For Room annotation processor
}

android {
    namespace 'com.example.todolistapp'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.todolistapp"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary true
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
    buildFeatures {
        compose true
    }
    composeOptions {
        kotlinCompilerExtensionVersion '1.5.1'
    }
    packaging {
        resources {
            excludes += '/META-INF/{AL2.0,LGPL2.1}'
        }
    }
}

dependencies {

    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.6.2'
    implementation 'androidx.activity:activity-compose:1.8.2'
    implementation platform('androidx.compose:compose-bom:2023.08.00')
    implementation 'androidx.compose.ui:ui'
    implementation 'androidx.compose.ui:ui-graphics'
    implementation 'androidx.compose.ui:ui-tooling-preview'
    implementation 'androidx.compose.material3:material3'

    // Room components
    implementation "androidx.room:room-runtime:2.6.1"
    kapt "androidx.room:room-compiler:2.6.1"
    // Kotlin Extensions and Coroutines support for Room
    implementation "androidx.room:room-ktx:2.6.1"

    // Lifecycle components
    implementation "androidx.lifecycle:lifecycle-viewmodel-compose:2.6.2"
    implementation "androidx.lifecycle:lifecycle-runtime-compose:2.6.2"

    // Navigation Compose
    implementation "androidx.navigation:navigation-compose:2.7.6"


    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
    androidTestImplementation platform('androidx.compose:compose-bom:2023.08.00')
    androidTestImplementation 'androidx.compose.ui:ui-test-junit4'
    debugImplementation 'androidx.compose.ui:ui-tooling'
    debugImplementation 'androidx.compose.ui:ui-test-manifest'
}
```

**2. Data Layer (Room Database)**

*   **`Task.kt` (Entity)**

    ```kotlin
    package com.example.todolistapp.data

    import androidx.room.Entity
    import androidx.room.PrimaryKey

    @Entity(tableName = "tasks")
    data class Task(
        @PrimaryKey(autoGenerate = true)
        val id: Int = 0,
        val title: String,
        val description: String? = null,
        val isCompleted: Boolean = false,
        val createdAt: Long = System.currentTimeMillis()
    )
    ```

*   **`TaskDao.kt` (Data Access Object)**

    ```kotlin
    package com.example.todolistapp.data

    import androidx.room.*
    import kotlinx.coroutines.flow.Flow

    @Dao
    interface TaskDao {
        @Query("SELECT * FROM tasks ORDER BY createdAt DESC")
        fun getAllTasks(): Flow<List<Task>>

        @Query("SELECT * FROM tasks WHERE id = :taskId")
        suspend fun getTaskById(taskId: Int): Task?

        @Insert(onConflict = OnConflictStrategy.REPLACE)
        suspend fun insertTask(task: Task)

        @Update
        suspend fun updateTask(task: Task)

        @Delete
        suspend fun deleteTask(task: Task)
    }
    ```

*   **`TaskDatabase.kt` (Database Class)**

    ```kotlin
    package com.example.todolistapp.data

    import androidx.room.Database
    import androidx.room.RoomDatabase

    @Database(entities = [Task::class], version = 1, exportSchema = false)
    abstract class TaskDatabase : RoomDatabase() {
        abstract fun taskDao(): TaskDao
    }
    ```

**3. Repository Layer (`TaskRepository.kt`)**

```kotlin
package com.example.todolistapp.data

import kotlinx.coroutines.flow.Flow
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class TaskRepository @Inject constructor(private val taskDao: TaskDao) {

    fun getAllTasks(): Flow<List<Task>> = taskDao.getAllTasks()

    suspend fun getTaskById(taskId: Int): Task? = taskDao.getTaskById(taskId)

    suspend fun insertTask(task: Task) = taskDao.insertTask(task)

    suspend fun updateTask(task: Task) = taskDao.updateTask(task)

    suspend fun deleteTask(task: Task) = taskDao.deleteTask(task)
}
```

**4. ViewModel Layer (`TaskViewModel.kt`)**

```kotlin
package com.example.todolistapp.ui.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.todolistapp.data.Task
import com.example.todolistapp.data.TaskRepository
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

class TaskViewModel(private val repository: TaskRepository) : ViewModel() {

    private val _tasks = MutableStateFlow<List<Task>>(emptyList())
    val tasks: StateFlow<List<Task>> = _tasks.asStateFlow()

    private val _currentTask = MutableStateFlow<Task?>(null)
    val currentTask: StateFlow<Task?> = _currentTask.asStateFlow()

    init {
        viewModelScope.launch {
            repository.getAllTasks().collect {
                _tasks.value = it
            }
        }
    }

    fun insertTask(task: Task) {
        viewModelScope.launch {
            repository.insertTask(task)
        }
    }

    fun updateTask(task: Task) {
        viewModelScope.launch {
            repository.updateTask(task)
        }
    }

    fun deleteTask(task: Task) {
        viewModelScope.launch {
            repository.deleteTask(task)
        }
    }

    fun getTaskDetails(taskId: Int) {
        viewModelScope.launch {
            _currentTask.value = repository.getTaskById(taskId)
        }
    }

    fun toggleTaskCompletion(task: Task) {
        viewModelScope.launch {
            repository.updateTask(task.copy(isCompleted = !task.isCompleted))
        }
    }
}
```

**5. UI Layer (Jetpack Compose)**

*   **`MainActivity.kt`**

    ```kotlin
    package com.example.todolistapp

    import android.os.Bundle
    import androidx.activity.ComponentActivity
    import androidx.activity.compose.setContent
    import androidx.activity.viewModels
    import androidx.compose.foundation.layout.fillMaxSize
    import androidx.compose.material3.MaterialTheme
    import androidx.compose.material3.Surface
    import androidx.compose.runtime.Composable
    import androidx.compose.ui.Modifier
    import androidx.compose.ui.tooling.preview.Preview
    import androidx.lifecycle.viewmodel.compose.viewModel
    import androidx.navigation.compose.NavHost
    import androidx.navigation.compose.composable
    import androidx.navigation.compose.rememberNavController
    import androidx.room.Room
    import com.example.todolistapp.data.TaskDatabase
    import com.example.todolistapp.data.TaskRepository
    import com.example.todolistapp.ui.screens.TaskDetailScreen
    import com.example.todolistapp.ui.screens.TaskListScreen
    import com.example.todolistapp.ui.theme.TodoListAppTheme
    import com.example.todolistapp.ui.viewmodel.TaskViewModel
    import com.example.todolistapp.ui.viewmodel.TaskViewModelFactory

    class MainActivity : ComponentActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)

            val db = Room.databaseBuilder(
                applicationContext,
                TaskDatabase::class.java, "task-database"
            ).build()
            val repository = TaskRepository(db.taskDao())
            val viewModelFactory = TaskViewModelFactory(repository)

            setContent {
                TodoListAppTheme {
                    Surface(
                        modifier = Modifier.fillMaxSize(),
                        color = MaterialTheme.colorScheme.background
                    ) {
                        TodoListAppNavHost(viewModelFactory)
                    }
                }
            }
        }
    }

    @Composable
    fun TodoListAppNavHost(viewModelFactory: TaskViewModelFactory) {
        val navController = rememberNavController()
        val taskViewModel: TaskViewModel = viewModel(factory = viewModelFactory)

        NavHost(navController = navController, startDestination = "tasklist") {
            composable("tasklist") {
                TaskListScreen(
                    viewModel = taskViewModel,
                    onNavigateToDetail = { taskId ->
                        navController.navigate("taskdetail/$taskId")
                    },
                    onNavigateToAddEdit = {
                        navController.navigate("addedit")
                    }
                )
            }
            composable("taskdetail/{taskId}") { backStackEntry ->
                val taskId = backStackEntry.arguments?.getString("taskId")?.toIntOrNull()
                taskId?.let {
                    TaskDetailScreen(
                        viewModel = taskViewModel,
                        taskId = it,
                        onNavigateBack = { navController.popBackStack() },
                        onNavigateToEdit = { editTaskId ->
                            navController.navigate("addedit?taskId=$editTaskId")
                        }
                    )
                }
            }
            composable("addedit?taskId={taskId}") { backStackEntry ->
                val taskId = backStackEntry.arguments?.getString("taskId")?.toIntOrNull()
                // You'd create an AddEditTaskScreen here
                // For simplicity, we'll just navigate back for now
                // In a real app, this screen would handle adding new tasks or editing existing ones.
                TaskListScreen(
                    viewModel = taskViewModel, // Reusing TaskListScreen, but this would be a dedicated AddEditTaskScreen
                    onNavigateToDetail = { navController.popBackStack() },
                    onNavigateToAddEdit = { navController.popBackStack() }
                )
            }
            composable("addedit") {
                // This is for adding a new task
                TaskListScreen(
                    viewModel = taskViewModel, // Reusing TaskListScreen, but this would be a dedicated AddEditTaskScreen
                    onNavigateToDetail = { navController.popBackStack() },
                    onNavigateToAddEdit = { navController.popBackStack() }
                )
            }
        }
    }
    ```

*   **`TaskViewModelFactory.kt`**

    ```kotlin
    package com.example.todolistapp.ui.viewmodel

    import androidx.lifecycle.ViewModel
    import androidx.lifecycle.ViewModelProvider
    import com.example.todolistapp.data.TaskRepository

    class TaskViewModelFactory(private val repository: TaskRepository) : ViewModelProvider.Factory {
        override fun <T : ViewModel> create(modelClass: Class<T>): T {
            if (modelClass.isAssignableFrom(TaskViewModel::class.java)) {
                @Suppress("UNCHECKED_CAST")
                return TaskViewModel(repository) as T
            }
            throw IllegalArgumentException("Unknown ViewModel class")
        }
    }
    ```

*   **`TaskListScreen.kt`**

    ```kotlin
    package com.example.todolistapp.ui.screens

    import androidx.compose.foundation.clickable
    import androidx.compose.foundation.layout.*
    import androidx.compose.foundation.lazy.LazyColumn
    import androidx.compose.foundation.lazy.items
    import androidx.compose.material.icons.Icons
    import androidx.compose.material.icons.filled.Add
    import androidx.compose.material3.*
    import androidx.compose.runtime.Composable
    import androidx.compose.runtime.collectAsState
    import androidx.compose.runtime.getValue
    import androidx.compose.ui.Alignment
    import androidx.compose.ui.Modifier
    import androidx.compose.ui.text.style.TextDecoration
    import androidx.compose.ui.unit.dp
    import com.example.todolistapp.data.Task
    import com.example.todolistapp.ui.viewmodel.TaskViewModel

    @OptIn(ExperimentalMaterial3Api::class)
    @Composable
    fun TaskListScreen(
        viewModel: TaskViewModel,
        onNavigateToDetail: (Int) -> Unit,
        onNavigateToAddEdit: () -> Unit
    ) {
        val tasks by viewModel.tasks.collectAsState()

        Scaffold(
            topBar = {
                TopAppBar(title = { Text("Todo List") })
            },
            floatingActionButton = {
                FloatingActionButton(onClick = onNavigateToAddEdit) {
                    Icon(Icons.Filled.Add, "Add new task")
                }
            }
        ) { paddingValues ->
            if (tasks.isEmpty()) {
                Box(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(paddingValues),
                    contentAlignment = Alignment.Center
                ) {
                    Text("No tasks yet! Click + to add one.")
                }
            } else {
                LazyColumn(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(paddingValues)
                ) {
                    items(tasks) { task ->
                        TaskItem(task = task,
                            onTaskClick = { onNavigateToDetail(task.id) },
                            onToggleComplete = { viewModel.toggleTaskCompletion(task) },
                            onDeleteTask = { viewModel.deleteTask(task) }
                        )
                    }
                }
            }
        }
    }

    @Composable
    fun TaskItem(
        task: Task,
        onTaskClick: (Task) -> Unit,
        onToggleComplete: (Task) -> Unit,
        onDeleteTask: (Task) -> Unit
    ) {
        Card(
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 8.dp, vertical = 4.dp)
                .clickable { onTaskClick(task) }
        ) {
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp),
                verticalAlignment = Alignment.CenterVertically,
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Column(modifier = Modifier.weight(1f)) {
                    Text(
                        text = task.title,
                        style = MaterialTheme.typography.headlineSmall.copy(
                            textDecoration = if (task.isCompleted) TextDecoration.LineThrough else null
                        )
                    )
                    task.description?.let {
                        Text(
                            text = it,
                            style = MaterialTheme.typography.bodyMedium.copy(
                                textDecoration = if (task.isCompleted) TextDecoration.LineThrough else null
                            )
                        )
                    }
                }
                Spacer(modifier = Modifier.width(8.dp))
                Checkbox(
                    checked = task.isCompleted,
                    onCheckedChange = { onToggleComplete(task) }
                )
                // You might add a delete button here or handle deletion in detail screen
                // IconButton(onClick = { onDeleteTask(task) }) {
                //     Icon(Icons.Default.Delete, contentDescription = "Delete task")
                // }
            }
        }
    }
    ```

*   **`TaskDetailScreen.kt`**

    ```kotlin
    package com.example.todolistapp.ui.screens

    import androidx.compose.foundation.layout.*
    import androidx.compose.material.icons.Icons
    import androidx.compose.material.icons.filled.ArrowBack
    import androidx.compose.material.icons.filled.Delete
    import androidx.compose.material.icons.filled.Edit
    import androidx.compose.material3.*
    import androidx.compose.runtime.Composable
    import androidx.compose.runtime.LaunchedEffect
    import androidx.compose.runtime.collectAsState
    import androidx.compose.runtime.getValue
    import androidx.compose.ui.Modifier
    import androidx.compose.ui.unit.dp
    import com.example.todolistapp.ui.viewmodel.TaskViewModel
    import java.text.SimpleDateFormat
    import java.util.*

    @OptIn(ExperimentalMaterial3Api::class)
    @Composable
    fun TaskDetailScreen(
        viewModel: TaskViewModel,
        taskId: Int,
        onNavigateBack: () -> Unit,
        onNavigateToEdit: (Int) -> Unit
    ) {
        LaunchedEffect(taskId) {
            viewModel.getTaskDetails(taskId)
        }
        val task by viewModel.currentTask.collectAsState()

        Scaffold(
            topBar = {
                TopAppBar(
                    title = { Text("Task Details") },
                    navigationIcon = {
                        IconButton(onClick = onNavigateBack) {
                            Icon(Icons.Filled.ArrowBack, contentDescription = "Back")
                        }
                    },
                    actions = {
                        IconButton(onClick = { task?.let { onNavigateToEdit(it.id) } }) {
                            Icon(Icons.Filled.Edit, contentDescription = "Edit Task")
                        }
                        IconButton(onClick = {
                            task?.let { viewModel.deleteTask(it) }
                            onNavigateBack()
                        }) {
                            Icon(Icons.Filled.Delete, contentDescription = "Delete Task")
                        }
                    }
                )
            }
        ) { paddingValues ->
            task?.let { currentTask ->
                Column(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(paddingValues)
                        .padding(16.dp)
                ) {
                    Text(
                        text = currentTask.title,
                        style = MaterialTheme.typography.headlineLarge
                    )
                    Spacer(modifier = Modifier.height(8.dp))
                    currentTask.description?.let {
                        Text(
                            text = it,
                            style = MaterialTheme.typography.bodyLarge
                        )
                        Spacer(modifier = Modifier.height(16.dp))
                    }

                    Row(
                        modifier = Modifier.fillMaxWidth(),
                        horizontalArrangement = Arrangement.SpaceBetween
                    ) {
                        Text("Completed: ${if (currentTask.isCompleted) "Yes" else "No"}")
                        Text("Created: ${formatTimestamp(currentTask.createdAt)}")
                    }
                }
            } ?: run {
                Box(
                    modifier = Modifier.fillMaxSize().padding(paddingValues),
                    contentAlignment = androidx.compose.ui.Alignment.Center
                ) {
                    CircularProgressIndicator()
                }
            }
        }
    }

    private fun formatTimestamp(timestamp: Long): String {
        val sdf = SimpleDateFormat("MMM dd, yyyy HH:mm", Locale.getDefault())
        return sdf.format(Date(timestamp))
    }
    ```

**Explanation:**

1.  **Dependencies:**
    *   `androidx.room:room-runtime`, `androidx.room:room-compiler`, `androidx.room:room-ktx`: For SQLite database integration with Room Persistence Library, including Kotlin Coroutines/Flow support.
    *   `androidx.lifecycle:lifecycle-viewmodel-compose`, `androidx.lifecycle:lifecycle-runtime-compose`: For ViewModel integration with Compose.
    *   `androidx.navigation:navigation-compose`: For navigating between different screens in Compose.

2.  **Room Database (`data` package):**
    *   **`Task.kt` (Entity):** Defines the structure of your data (a task). `id` is the primary key and auto-generated.
    *   **`TaskDao.kt` (Data Access Object):** An interface that defines methods for interacting with the database.
        *   `getAllTasks()`: Returns a `Flow<List<Task>>`. This is crucial for real-time updates in Compose. When the database changes, the Flow emits a new list, and your UI automatically recomposes.
        *   `getTaskById()`, `insertTask()`, `updateTask()`, `deleteTask()`: Standard CRUD operations. All are `suspend` functions because Room performs database operations on a background thread.
    *   **`TaskDatabase.kt` (Database):** The main database class. It's an abstract class that extends `RoomDatabase` and provides access to your DAOs. `version = 1` indicates the database version.

3.  **Repository (`data/TaskRepository.kt`):**
    *   Acts as a clean API for the ViewModel to interact with data. It abstracts away the data source (Room in this case).
    *   It directly uses the `TaskDao` to perform database operations.
    *   The `getAllTasks()` method directly exposes the `Flow` from the DAO.

4.  **ViewModel (`ui/viewmodel/TaskViewModel.kt`):**
    *   **`_tasks` and `tasks`:** A `MutableStateFlow` (`_tasks`) is used to hold the current list of tasks. `asStateFlow()` exposes an immutable `StateFlow` to the UI, preventing external modification.
    *   **`_currentTask` and `currentTask`:** Similarly for holding the details of a single task when navigating to the detail screen.
    *   **`init` block:** When the ViewModel is created, it launches a coroutine to `collect` the `Flow` of all tasks from the `repository`. Every time the database changes and the `Flow` emits a new list, the `_tasks.value` is updated, which in turn triggers a recomposition of any Composable observing `tasks`.
    *   **CRUD functions:** `insertTask`, `updateTask`, `deleteTask`, `getTaskDetails`, `toggleTaskCompletion` are all `suspend` functions and are called within `viewModelScope.launch { ... }`. This ensures they run on a background thread and don't block the UI.

5.  **UI Layer (`ui/screens` and `MainActivity.kt`):**
    *   **`MainActivity.kt`:**
        *   Sets up the Room database instance and `TaskRepository`.
        *   Creates a `TaskViewModelFactory` to correctly instantiate `TaskViewModel` with the `TaskRepository`.
        *   Uses `NavHost` for navigation between screens.
    *   **`TaskListScreen.kt`:**
        *   `tasks by viewModel.tasks.collectAsState()`: Observes the `tasks` `StateFlow` from the ViewModel. When the `StateFlow` emits a new list, the UI recomposes automatically.
        *   `LazyColumn`: Efficiently displays a scrollable list of tasks.
        *   `TaskItem` Composable: Renders an individual task, handling click events and completion toggles.
        *   `FloatingActionButton`: For adding new tasks.
    *   **`TaskDetailScreen.kt`:**
        *   `LaunchedEffect(taskId)`: When the `taskId` changes (meaning a new task is being viewed), it calls `viewModel.getTaskDetails(taskId)` to fetch the specific task.
        *   `task by viewModel.currentTask.collectAsState()`: Observes the `currentTask` `StateFlow` for real-time updates of the detailed task.
        *   Displays task title, description, completion status, and creation date.
        *   Provides options to edit and delete the task (which will eventually navigate to an edit screen or delete the task and go back).

**How Flow is Used:**

*   **Real-time Updates:** The most significant benefit here is how `Flow` handles real-time updates from the database.
    *   `TaskDao.getAllTasks()` returns a `Flow<List<Task>>`.
    *   The `TaskViewModel` `collects` this `Flow`.
    *   Whenever an `insert`, `update`, or `delete` operation occurs in the database, Room automatically triggers the `Flow` to emit the new list of tasks.
    *   The `ViewModel` updates its `_tasks` `MutableStateFlow`.
    *   The `TaskListScreen` observes this `StateFlow` using `collectAsState()`, which then triggers a recomposition with the updated list, ensuring the UI is always in sync with the database.

**To Run This App:**

1.  Create a new Android Studio project with an "Empty Activity" template for Jetpack Compose.
2.  Update your `build.gradle (Module :app)` file with all the dependencies listed above.
3.  Create the packages `com.example.todolistapp.data`, `com.example.todolistapp.ui.screens`, and `com.example.todolistapp.ui.viewmodel`.
4.  Copy the respective Kotlin files into their correct packages.
5.  Run the app on an emulator or a physical device.

**Next Steps (Enhancements):**

*   **Add/Edit Task Screen:** Implement a dedicated screen (`AddEditTaskScreen`) to allow users to add new tasks and edit existing ones. This screen would take parameters like `taskId` (for editing) or no `taskId` (for adding).
*   **Error Handling:** Add proper error handling for database operations.
*   **Swipe to Delete:** Implement swipe-to-delete functionality for tasks in `TaskListScreen`.
*   **Search/Filter:** Add search and filtering capabilities to the task list.
*   **Dependency Injection:** For larger apps, consider using a DI framework like Hilt to manage dependencies (Room, Repository, ViewModel).
*   **Theming:** Customize the app's theme with more specific colors, typography, and shapes.

This comprehensive setup provides a solid foundation for a reactive and robust Todo List app using modern Android development practices!
