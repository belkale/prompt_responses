# How do I implement scheduled notifications in Jetpack Compose Android App? I need to schedule recurring reminders with actions to snooze and complete reminder from within notifications.

Implementing scheduled notifications with interactive actions (Snooze/Complete) involves coordinating three main Android components:
1.  **AlarmManager:** To handle the precise scheduling.
2.  **BroadcastReceiver:** To wake up the app when the alarm fires and handle button clicks.
3.  **NotificationManager:** To display the UI and attach the actions.
4.  **Jetpack Compose:** To provide the user interface for setting the reminder.

Here is a complete, step-by-step implementation.

### Step 1: Add Permissions and Receiver to Manifest

In `AndroidManifest.xml`, you need permissions for notifications (Android 13+) and exact alarms (Android 12+). You also need to register the receiver.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" ...>

    <!-- Required for Android 13+ -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    <!-- Required for precise timing -->
    <uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
    <uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT" />

    <application ...>
        <!-- The receiver we will create later -->
        <receiver 
            android:name=".ReminderReceiver"
            android:enabled="true"
            android:exported="false" />
            
        <!-- ... activity setup ... -->
    </application>
</manifest>
```

### Step 2: Create the Data Model

Create a simple data class to hold reminder details.

```kotlin
data class ReminderItem(
    val id: Int,
    val title: String,
    val message: String,
    val timeInMillis: Long
)
```

### Step 3: Implement the BroadcastReceiver

This is the brain of the operation. It handles three scenarios:
1.  The Alarm fires (Show Notification).
2.  User clicks "Snooze".
3.  User clicks "Complete".

```kotlin
import android.app.NotificationManager
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.widget.Toast
import androidx.core.app.NotificationCompat

class ReminderReceiver : BroadcastReceiver() {

    companion object {
        const val ACTION_SHOW = "ACTION_SHOW_NOTIFICATION"
        const val ACTION_SNOOZE = "ACTION_SNOOZE"
        const val ACTION_COMPLETE = "ACTION_COMPLETE"
        
        const val EXTRA_ID = "EXTRA_ID"
        const val EXTRA_TITLE = "EXTRA_TITLE"
        const val EXTRA_MESSAGE = "EXTRA_MESSAGE"
    }

    override fun onReceive(context: Context, intent: Intent) {
        val notificationService = ReminderNotificationService(context)
        val alarmScheduler = AndroidAlarmScheduler(context) // We will create this next
        
        val id = intent.getIntExtra(EXTRA_ID, -1)
        val title = intent.getStringExtra(EXTRA_TITLE) ?: "Reminder"
        val message = intent.getStringExtra(EXTRA_MESSAGE) ?: ""

        when (intent.action) {
            ACTION_SHOW -> {
                // 1. Show the notification
                notificationService.showNotification(
                    ReminderItem(id, title, message, System.currentTimeMillis())
                )
            }
            ACTION_SNOOZE -> {
                // 2. Dismiss current notification
                val manager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
                manager.cancel(id)

                // 3. Schedule new alarm for 15 minutes later (Snooze)
                val snoozeTime = System.currentTimeMillis() + 15 * 60 * 1000 // 15 mins
                val snoozedItem = ReminderItem(id, title, message, snoozeTime)
                alarmScheduler.schedule(snoozedItem)
                
                Toast.makeText(context, "Snoozed for 15 mins", Toast.LENGTH_SHORT).show()
            }
            ACTION_COMPLETE -> {
                // 4. Mark as complete (Update your DB/Repo here)
                val manager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
                manager.cancel(id)
                
                Toast.makeText(context, "Reminder Completed!", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

### Step 4: Create the Notification Service

This helper class builds the notification with the **Action Buttons**.

```kotlin
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.content.Context
import android.content.Intent
import android.os.Build
import androidx.core.app.NotificationCompat
// Use R from your package
// import com.example.yourapp.R 

class ReminderNotificationService(private val context: Context) {

    private val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

    companion object {
        const val CHANNEL_ID = "reminder_channel"
    }

    fun showNotification(item: ReminderItem) {
        // 1. Create Channel (Required for Android O+)
        createNotificationChannel()

        // 2. Create Intents for Actions
        val snoozeIntent = Intent(context, ReminderReceiver::class.java).apply {
            action = ReminderReceiver.ACTION_SNOOZE
            putExtra(ReminderReceiver.EXTRA_ID, item.id)
            putExtra(ReminderReceiver.EXTRA_TITLE, item.title)
            putExtra(ReminderReceiver.EXTRA_MESSAGE, item.message)
        }
        val snoozePendingIntent = PendingIntent.getBroadcast(
            context, 
            item.id + 1, // Unique Request Code
            snoozeIntent, 
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )

        val completeIntent = Intent(context, ReminderReceiver::class.java).apply {
            action = ReminderReceiver.ACTION_COMPLETE
            putExtra(ReminderReceiver.EXTRA_ID, item.id)
        }
        val completePendingIntent = PendingIntent.getBroadcast(
            context, 
            item.id + 2, 
            completeIntent, 
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )

        // 3. Build Notification
        val notification = NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(android.R.drawable.ic_dialog_info) // Replace with your icon
            .setContentTitle(item.title)
            .setContentText(item.message)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            // Add Action Buttons
            .addAction(android.R.drawable.ic_lock_idle_alarm, "Snooze", snoozePendingIntent)
            .addAction(android.R.drawable.checkbox_on_background, "Complete", completePendingIntent)
            .setAutoCancel(true)
            .build()

        notificationManager.notify(item.id, notification)
    }

    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                "Reminders",
                NotificationManager.IMPORTANCE_HIGH
            ).apply {
                description = "Channels for reminders"
            }
            notificationManager.createNotificationChannel(channel)
        }
    }
}
```

### Step 5: Alarm Manager Scheduler

This handles talking to the Android System to wake the app up at a specific time.

```kotlin
import android.app.AlarmManager
import android.app.PendingIntent
import android.content.Context
import android.content.Intent

class AndroidAlarmScheduler(private val context: Context) {

    private val alarmManager = context.getSystemService(AlarmManager::class.java)

    fun schedule(item: ReminderItem) {
        // Intent to trigger the BroadcastReceiver
        val intent = Intent(context, ReminderReceiver::class.java).apply {
            action = ReminderReceiver.ACTION_SHOW
            putExtra(ReminderReceiver.EXTRA_ID, item.id)
            putExtra(ReminderReceiver.EXTRA_TITLE, item.title)
            putExtra(ReminderReceiver.EXTRA_MESSAGE, item.message)
        }

        val pendingIntent = PendingIntent.getBroadcast(
            context,
            item.id,
            intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )

        // Use setExactAndAllowWhileIdle for precise delivery even in Doze mode
        alarmManager.setExactAndAllowWhileIdle(
            AlarmManager.RTC_WAKEUP,
            item.timeInMillis,
            pendingIntent
        )
    }
    
    fun cancel(item: ReminderItem) {
        val intent = Intent(context, ReminderReceiver::class.java).apply {
            action = ReminderReceiver.ACTION_SHOW
        }
        val pendingIntent = PendingIntent.getBroadcast(
            context,
            item.id,
            intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        alarmManager.cancel(pendingIntent)
    }
}
```

### Step 6: Jetpack Compose UI

Finally, wire it all together in your UI. Note that you need to handle runtime permissions for Android 13+.

```kotlin
import android.Manifest
import android.content.pm.PackageManager
import android.os.Build
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp
import androidx.core.content.ContextCompat

@Composable
fun ReminderScreen() {
    val context = LocalContext.current
    val scheduler = remember { AndroidAlarmScheduler(context) }
    var hasNotificationPermission by remember {
        mutableStateOf(
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                ContextCompat.checkSelfPermission(
                    context,
                    Manifest.permission.POST_NOTIFICATIONS
                ) == PackageManager.PERMISSION_GRANTED
            } else true
        )
    }

    val permissionLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.RequestPermission(),
        onResult = { isGranted -> hasNotificationPermission = isGranted }
    )

    Column(
        modifier = Modifier.fillMaxSize().padding(16.dp),
        verticalArrangement = Arrangement.Center
    ) {
        if (!hasNotificationPermission && Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            Button(onClick = { permissionLauncher.launch(Manifest.permission.POST_NOTIFICATIONS) }) {
                Text("Request Notification Permission")
            }
        } else {
            Button(
                onClick = {
                    // Schedule for 10 seconds from now
                    val alarmTime = System.currentTimeMillis() + 10_000 
                    val item = ReminderItem(
                        id = 123,
                        title = "Take Medicine",
                        message = "It's time to take your daily pill.",
                        timeInMillis = alarmTime
                    )
                    scheduler.schedule(item)
                    Toast.makeText(context, "Scheduled for 10 seconds!", Toast.LENGTH_SHORT).show()
                }
            ) {
                Text("Schedule Reminder (10s)")
            }
        }
    }
}
```

### Key Concepts Explained

1.  **PendingIntent Flags:** We use `FLAG_IMMUTABLE` for security (mandatory in newer Android versions), but we combine it with `FLAG_UPDATE_CURRENT`. This ensures that if you reschedule an alarm with the same ID, the new data (time/message) overwrites the old one.
2.  **Unique Request Codes:** In `ReminderNotificationService`, when creating PendingIntents for Snooze/Complete, we use `item.id + 1` or `+ 2`. If you use the same request code for all buttons, they might overwrite each other.
3.  **Recurring Logic:**
    *   In the example above, I used `setExactAndAllowWhileIdle`. This fires **once**.
    *   To make it **Recurring** (e.g., daily): When the `ACTION_SHOW` or `ACTION_COMPLETE` block runs in the Receiver, you should calculate the *next* occurrence time (e.g., `currentTime + 24 hours`) and call `scheduler.schedule(nextItem)` again immediately.
    *   *Why not `setRepeating`?* `setRepeating` in Android is inexact to save battery. For a reminder app where "8:00 AM" means "8:00 AM", you must use `setExact` and manually schedule the next one when the current one fires.
4.  **Doze Mode:** `setExactAndAllowWhileIdle` allows the alarm to fire even if the phone is sitting on a table unused (Doze mode). Standard alarms might be delayed until the user picks up the phone.
