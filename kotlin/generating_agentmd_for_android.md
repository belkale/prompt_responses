# How do I generate AGENT.md for my Jetpack Compose Android app. It should use my build.gradle.kts file in my app directory to understand frameworks used. It should understand the existing code structure based on general coding conventions. It should also have details of coding style based on existing code. Also it should add setup and build commands based on gradle for Android app.

Of course! An `AGENT.md` file is an excellent way to provide context to an AI coding assistant, enabling it to understand your project's architecture, dependencies, and conventions. This leads to more accurate and helpful code generation and modifications.

Here is a comprehensive template for an `AGENT.md` file for a typical Jetpack Compose Android app. I've designed it to be a "fill-in-the-blanks" guide.

### How to Use This Guide

1.  **Create the File:** In the root directory of your Android project, create a new file named `AGENT.md`.
2.  **Analyze `build.gradle.kts`:** Open your `app/build.gradle.kts` file. You will use the `dependencies` block to fill out the "Technology Stack" section below.
3.  **Review Your Code:** Look at your existing code structure (package names, file names) and coding style to fill in the "Project Structure" and "Coding Style" sections.
4.  **Copy, Paste, and Customize:** Copy the template below into your `AGENT.md` file and customize the bracketed `[ ]` sections with your project's specific details.

---

### `AGENT.md` Template for a Jetpack Compose App

````markdown
# AGENT.md: AI Coding Assistant Guide for [Your Project Name]

This document provides the necessary context for an AI assistant to understand and contribute to this project effectively. It outlines the technology stack, project architecture, coding conventions, and key commands.

## 1. Project Overview

*   **Project Name:** [Your Project Name]
*   **Description:** [A brief, one-sentence description of what this app does. e.g., "A minimalist note-taking app that syncs with the cloud."]
*   **Core Functionality:**
    *   [List 2-3 core features, e.g., "User authentication via Firebase."]
    *   [e.g., "Creating, editing, and deleting notes."]
    *   [e.g., "Displaying a list of notes on the main screen."]

## 2. Technology Stack & Frameworks

This project is a native Android application written in Kotlin, using the Jetpack Compose toolkit. The primary dependencies are listed below. This information is derived from `app/build.gradle.kts`.

*   **Core:**
    *   **Language:** Kotlin
    *   **UI Toolkit:** Jetpack Compose
    *   **Build System:** Gradle with Kotlin DSL (`.kts`)

*   **Jetpack & AndroidX Libraries:**
    *   **Architecture Components:**
        *   `ViewModel`: For managing UI-related data in a lifecycle-conscious way.
        *   `Navigation Compose`: For handling in-app navigation between composable screens.
        *   `Lifecycle`: For observing Android lifecycles (`collectAsStateWithLifecycle`).
    *   **Asynchronous Programming:**
        *   `Kotlin Coroutines & Flow`: For managing background threads and handling asynchronous data streams.
    *   **Dependency Injection:**
        *   `Hilt`: For managing dependencies throughout the app. We use `@HiltViewModel` and `@AndroidEntryPoint`.
    *   **Data Persistence (Local):**
        *   `Room`: [If you use Room, describe it here. e.g., "For storing user notes locally in a SQLite database."]
        *   `DataStore`: [If you use DataStore, describe it here. e.g., "For storing user preferences and settings."]
    *   **UI & Theming:**
        *   `Compose Material 3`: The primary design system for UI components.
        *   `Coil` / `Glide`: [Choose one, e.g., "Coil for loading images from the network asynchronously."]

*   **Networking:**
    *   `Retrofit`: For making type-safe HTTP requests to our REST API.
    *   `OkHttp`: As the underlying HTTP client for Retrofit (often used for interceptors).
    *   `Moshi` / `Gson`: [Choose one, e.g., "Moshi for parsing JSON responses into Kotlin data classes."]

*   **Testing:**
    *   `JUnit 4/5`: For unit testing ViewModels and Repositories.
    *   `MockK` / `Mockito`: [Choose one, e.g., "MockK for creating mocks in unit tests."]
    *   `Turbine`: For testing Kotlin Flow emissions.
    *   `Compose Test Suite`: For UI and integration testing of composable functions.

## 3. Project Structure & Architecture

The project follows the **MVVM (Model-View-ViewModel)** architecture pattern, organized by feature.

```
com.example.yourapp/
├── core/                # Shared code across features
│   ├── data/            # Core data components (Database, Network Client, etc.)
│   ├── di/              # Core Hilt modules (e.g., AppModule)
│   └── domain/          # Core domain models or use cases
│   └── ui/              # Shared UI components (e.g., CustomButton, LoadingSpinner)
│
├── data/                  # Data layer implementation
│   ├── model/             # Data Transfer Objects (DTOs) and domain models
│   ├── local/             # Room DAO and database definitions
│   ├── remote/            # Retrofit API service interfaces
│   └── repository/        # Repository implementations
│
├── di/                    # Feature-specific or app-wide Hilt modules
│
├── ui/                    # Presentation layer (Jetpack Compose)
│   ├── navigation/        # Navigation graph and route definitions (e.g., AppNavigation.kt)
│   ├── theme/             # App theme, colors, typography (Theme.kt, Color.kt)
│   └── feature_name/      # A specific feature screen (e.g., home, profile)
│       ├── HomeScreen.kt  # The main Composable function for the screen
│       └── HomeViewModel.kt # The ViewModel for the screen
│
└── MainActivity.kt        # The main entry point of the app
```

### Key Architectural Principles:

*   **View (Composables):** Reside in the `ui/feature_name` packages. They are responsible for displaying state and forwarding user events to the ViewModel. They should be as "dumb" as possible and observe state from a `StateFlow`.
*   **ViewModel:** Resides alongside its screen composable. It contains the business logic for the screen, exposes UI state via a `StateFlow<UiState>`, and is injected with repositories or use cases. All asynchronous work is launched in `viewModelScope`.
*   **Repository:** The single source of truth for data. It fetches data from remote (network) or local (database) sources and abstracts the data source from the ViewModel.
*   **Dependency Injection:** Hilt is used to provide dependencies. ViewModels are injected using `@HiltViewModel`, and dependencies like repositories are provided in Hilt Modules (`@Module`, `@Provides`).

## 4. Coding Style & Conventions

*   **Language:** Follow the official [Kotlin Style Guide](https://kotlinlang.org/docs/coding-conventions.html).
*   **Compose Naming:**
    *   Composable functions are named in `PascalCase`, e.g., `UserProfileScreen`.
    *   Reusable, generic composables are in the `core/ui` package.
*   **ViewModel State Management:**
    *   Each ViewModel should expose a single `StateFlow` representing the UI state.
    *   A `data class` is used to define the state, e.g., `data class HomeUiState(...)`.
    *   The state is collected in the Composable using `viewModel.uiState.collectAsStateWithLifecycle()`.
*   **User Events:** User actions from the UI are sent to the ViewModel via public functions, e.g., `fun onSaveNoteClicked()`.
*   **Immutability:** Prefer `val` over `var` and use immutable data structures (e.g., `List` instead of `MutableList`) for UI state to ensure unidirectional data flow.
*   **Coroutines:** Use structured concurrency. Launch coroutines from `viewModelScope`. Use `Dispatchers.IO` for network/database operations and `Dispatchers.Default` for CPU-intensive work.

## 5. Setup & Build Instructions

This project uses the Gradle wrapper to ensure a consistent build environment.

**Prerequisites:**
*   Android Studio [e.g., Hedgehog or newer]
*   JDK 17 or higher

**Setup:**
1.  Clone the repository: `git clone [URL_to_your_repo]`
2.  Open the project in Android Studio. It will automatically sync Gradle.

**Common Gradle Commands (run from the project root directory):**

*   **Clean the project:**
    ```bash
    ./gradlew clean
    ```
*   **Build a debug APK:**
    ```bash
    ./gradlew assembleDebug
    ```
*   **Build a release APK:**
    ```bash
    ./gradlew assembleRelease
    ```
*   **Install the debug app on a connected device/emulator:**
    ```bash
    ./gradlew installDebug
    ```
*   **Run all unit tests:**
    ```bash
    ./gradlew testDebugUnitTest
    ```
*   **Run all instrumented (UI) tests:**
    ```bash
    ./gradlew connectedDebugAndroidTest
    ```
*   **On Windows**, use `gradlew.bat` instead of `./gradlew`.

## 6. How to Add a New Feature (Screen)

Follow these steps to add a new feature, for example, a "Settings" screen:

1.  **Create Package:** Create a new package `ui.settings`.
2.  **Create ViewModel:** Create `SettingsViewModel.kt` inside `ui.settings`. Annotate it with `@HiltViewModel` and define its `SettingsUiState`.
3.  **Create Screen:** Create `SettingsScreen.kt`. Define the composable `SettingsScreen()` that takes the `SettingsViewModel` as a parameter (using `hiltViewModel()`).
4.  **Update Navigation:** Add a new route for the settings screen in your navigation graph file (e.g., `ui/navigation/AppNavigation.kt`).
5.  **Data Layer (if needed):** If the feature requires new data, add new functions to the relevant `Repository` and create any necessary `DAO` or network API endpoints.
6.  **Dependency Injection (if needed):** If you create new repositories or data sources, provide them in a Hilt module.
````
