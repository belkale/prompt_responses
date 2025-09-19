# Give me a crash course on kotlinx.datetime library

Of course! Here is a crash course on the `kotlinx.datetime` library.

This guide will take you from the basics to practical application, covering the most important concepts you need to know.

### Why `kotlinx.datetime`?

Before `kotlinx.datetime`, Kotlin developers had a few choices for handling dates and times:
*   **Java's `java.time` (JSR-310):** Excellent and the standard on the JVM, but not available on other platforms like Kotlin/JS or Kotlin/Native.
*   **Platform-specific libraries:** Using `NSDate` on iOS or Moment.js on JavaScript. This breaks the "write once, run anywhere" promise of Kotlin Multiplatform.
*   **Third-party multiplatform libraries:** Several existed, but none were official.

`kotlinx.datetime` solves this by providing a **standardized, multiplatform, and idiomatic Kotlin library** for working with dates and times.

### 1. Setup

First, add the dependency to your `build.gradle.kts` file. It's a multiplatform library, so you add it to the `commonMain` source set.

```kotlin
// In your build.gradle.kts

kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.6.0") // Check for the latest version
            }
        }
    }
}
```

### 2. Core Concepts: The Main Classes

The library is built around a few core, immutable classes. Understanding the difference between them is crucial.

#### `Instant`: The Machine Time
*   **What it is:** A single, specific point in time on the global timeline. It represents the number of nanoseconds since the UNIX epoch (`1970-01-01T00:00:00Z`) in UTC.
*   **Key characteristic:** It's **unambiguous** and has no concept of a time zone or calendar. It's the same `Instant` everywhere in the world.
*   **When to use it:**
    *   Storing timestamps in a database.
    *   Sending timestamps over an API.
    *   Logging events.
    *   Any time you need an absolute, universal point in time.

```kotlin
import kotlinx.datetime.*

// Get the current moment in time
val now: Instant = Clock.System.now()
println(now) // e.g., 2023-10-27T10:30:00.123456789Z (ISO 8601 format)

// Create from a string
val specificInstant = Instant.parse("2024-01-01T12:00:00Z")
```

#### `LocalDateTime`: The Human "Wall Clock" Time
*   **What it is:** A date and time without any time zone information. Think of it as what you see on a wall clock.
*   **Key characteristic:** It's **ambiguous**. `2024-11-05T09:00:00` could be morning in New York or evening in Tokyo. It doesn't represent a single point on the global timeline.
*   **When to use it:**
    *   Representing a user-selected date and time from a calendar UI.
    *   Describing a future event that happens at the same "local" time everywhere (e.g., "New Year's Day starts at midnight").

```kotlin
// Create a specific LocalDateTime
val newYearsDay = LocalDateTime(2025, 1, 1, 0, 0, 0)
println(newYearsDay) // 2025-01-01T00:00:00

// Parse from a string
val meetingTime = LocalDateTime.parse("2024-07-22T14:30")
```

#### `LocalDate` and `LocalTime`
These are just the date and time parts of a `LocalDateTime`.

```kotlin
val today: LocalDate = Clock.System.todayIn(TimeZone.currentSystemDefault())
println(today) // e.g., 2023-10-27

val lunchTime = LocalTime(12, 30)
println(lunchTime) // 12:30
```

#### `TimeZone`: The Bridge Between Machine and Human Time
*   **What it is:** A set of rules for a specific region of the world, including its offset from UTC and rules for Daylight Saving Time.
*   **Key characteristic:** It's the essential piece of information needed to convert between an `Instant` (universal time) and a `LocalDateTime` (local time).

```kotlin
// Get the system's default time zone
val systemZone: TimeZone = TimeZone.currentSystemDefault()
println(systemZone) // e.g., Europe/Berlin

// Get a specific time zone by its ID
val newYorkZone: TimeZone = TimeZone.of("America/New_York")

// UTC is a special constant
val utcZone: TimeZone = TimeZone.UTC
```

---

### 3. Key Operations

#### Converting Between `Instant` and `LocalDateTime`

This is the most common and important operation.

```kotlin
val currentTime: Instant = Clock.System.now()
val timeZone = TimeZone.of("Asia/Tokyo")

// Instant -> LocalDateTime (What time is it in Tokyo right now?)
val tokyoTime: LocalDateTime = currentTime.toLocalDateTime(timeZone)
println("Current time in Tokyo: $tokyoTime")

// LocalDateTime -> Instant (What universal moment does 9 AM on Christmas in Tokyo represent?)
val christmasMorningInTokyo = LocalDateTime(2024, 12, 25, 9, 0)
val christmasInstant: Instant = christmasMorningInTokyo.toInstant(timeZone)
println("Christmas 9 AM in Tokyo is $christmasInstant in UTC")

// Be careful: A LocalDateTime can be ambiguous or non-existent during DST changes.
// .toInstant() will throw an exception in those cases.
```

> **Golden Rule:**
> *   **Store and transmit `Instant`s.** They are absolute and unambiguous.
> *   **Use `LocalDateTime` and `TimeZone` for display and user input.** Convert to an `Instant` as soon as possible.

#### Date and Time Arithmetic

The library provides two classes for representing spans of time:

*   **`Duration`**: A fixed, exact amount of time measured in seconds and nanoseconds (e.g., "2 hours and 30 minutes"). Machine-scale.
*   **`DateTimePeriod`**: A calendar-based amount of time measured in years, months, and days (e.g., "1 month and 5 days"). Human-scale.

```kotlin
// --- Using Duration (exact time) ---
val now = Clock.System.now()
val inTenMinutes = now + 10.minutes // Cool extension functions!
val twoHoursAgo = now - 2.hours

// Calculate the difference between two Instants
val durationBetween: Duration = inTenMinutes - now
println(durationBetween.inWholeMinutes) // 10


// --- Using DateTimePeriod (calendar time) ---
val today = LocalDate(2024, 2, 20)
val period = DateTimePeriod(months = 1, days = 10)
val futureDate = today.plus(period) // plus() takes a period
println(futureDate) // 2024-03-30

// Adding 1 month to Jan 31 results in Feb 29 (in a leap year)
val trickyDate = LocalDate(2024, 1, 31).plus(DateTimePeriod(months = 1))
println(trickyDate) // 2024-02-29
```

#### Getting Components

You can easily extract parts of a date or time.

```kotlin
val dateTime = LocalDateTime(2024, 10, 31, 20, 30, 15)

println(dateTime.year)       // 2024
println(dateTime.month)      // OCTOBER
println(dateTime.monthNumber) // 10
println(dateTime.dayOfMonth)  // 31
println(dateTime.dayOfWeek)   // THURSDAY
println(dateTime.hour)        // 20
```

#### Formatting and Parsing

`kotlinx.datetime` sticks to the **ISO 8601 standard** for its default `toString()` and `parse()` methods.

```kotlin
// Parsing is built-in for ISO 8601
val instant = Instant.parse("2023-10-27T12:00:00Z")
val local = LocalDateTime.parse("2023-10-27T14:00:00")

// toString() produces ISO 8601
println(instant.toString()) // 2023-10-27T12:00:00Z
println(local.toString())   // 2023-10-27T14:00:00
```

> **What about custom formats (e.g., "dd/MM/yyyy")?**
> The core `kotlinx.datetime` library **does not include a custom formatter**. This is a deliberate design choice to keep the common library lean.
> For custom formatting, you must rely on **platform-specific APIs**:
> *   **JVM:** Use `java.time.format.DateTimeFormatter`. You can convert between `kotlinx.datetime` and `java.time` types.
>     ```kotlin
>     // On JVM only
    >     import java.time.format.DateTimeFormatter
    >     val kxDateTime = LocalDateTime(2024, 1, 1, 10, 0)
    >     val javaDateTime = java.time.LocalDateTime.parse(kxDateTime.toString()) // Convert
    >     val formatter = DateTimeFormatter.ofPattern("MM/dd/yyyy HH:mm")
    >     println(javaDateTime.format(formatter)) // 01/01/2024 10:00
>     ```
> *   **JS/Native:** Use platform-specific formatting functions available there.

### Practical Example: Is an appointment in the past?

Let's schedule a meeting for **December 25th, 2024, at 9:00 AM in New York** and check if it has already passed.

```kotlin
fun checkAppointment() {
    // 1. Define the appointment in its local context
    val appointmentLocal = LocalDateTime(2024, 12, 25, 9, 0)
    val appointmentZone = TimeZone.of("America/New_York")

    // 2. Convert the local time to a universal Instant
    val appointmentInstant = appointmentLocal.toInstant(appointmentZone)
    println("Appointment is at universal time: $appointmentInstant")

    // 3. Get the current universal time
    val nowInstant = Clock.System.now()
    println("Current universal time:       $nowInstant")

    // 4. Compare the two Instants
    if (nowInstant > appointmentInstant) {
        println("The appointment is in the past!")
        val timePassed = nowInstant - appointmentInstant
        println("It was ${timePassed.inWholeDays} days ago.")
    } else {
        println("The appointment is in the future.")
        val timeUntil = appointmentInstant - nowInstant
        println("It is in ${timeUntil.inWholeDays} days.")
    }
}

checkAppointment()
```

### Summary & Best Practices

1.  **`Instant` is King for data:** Use `Instant` for storing, logging, and sending time data. It's the unambiguous source of truth.
2.  **`LocalDateTime` is for Humans:** Use `LocalDateTime` when interacting with a user (e.g., calendar pickers).
3.  **`TimeZone` is the crucial link:** Always be aware of which `TimeZone` you're in when converting between `Instant` and `LocalDateTime`. Never assume a default.
4.  **Arithmetic:** Use `Duration` for exact time calculations and `DateTimePeriod` for calendar-based logic.
5.  **Formatting:** Stick to ISO 8601 for machine-to-machine communication. For user-facing custom formats, use platform-specific formatters.

This crash course covers over 90% of what you'll need to use `kotlinx.datetime` effectively. Happy coding
