# 🔄 Background Sync with WorkManager (Android)

This document explains how to implement **background data synchronization** using Android WorkManager with a `CoroutineWorker`.

---

# 📌 Overview

This setup:

* Runs background sync tasks
* Automatically retries on failure
* Executes only when network is available
* Ensures only one worker runs at a time

---

# 🧠 Architecture Flow

```
UI / App Start → startSyncWorker()
                ↓
          WorkManager
                ↓
           SyncWorker
                ↓
     SaveDataRemoteUseCase
                ↓
             API Call
```

---

# 📦 1. SyncWorker

### ✅ Description

A background worker that:

* Calls a use case
* Handles success/failure
* Retries automatically

---

### 📄 File: `SyncWorker.kt`

```kotlin
class SyncWorker(
    appContext: Context,
    workerParams: WorkerParameters
) : CoroutineWorker(appContext, workerParams), KoinComponent {

    private val useCase: SaveDataRemoteUseCase by inject()

    override suspend fun doWork(): Result {
        return try {

            val success = useCase.invoke()

            if (success) {
                Result.success()
            } else {
                Result.retry() // retry later
            }

        } catch (e: Exception) {
            Result.retry()
        }
    }
}
```

---

# ⚙️ 2. Start Worker Function

### ✅ Description

Schedules the worker with:

* Network constraint
* Retry strategy
* Unique work policy

---

### 📄 File: `WorkerHelper.kt`

```kotlin
fun startSyncWorker(context: Context) {

    val request = OneTimeWorkRequestBuilder<SyncWorker>()
        .setConstraints(
            Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
                .build()
        )
        .setBackoffCriteria(
            BackoffPolicy.EXPONENTIAL,
            10,
            TimeUnit.SECONDS
        )
        .build()

    WorkManager.getInstance(context)
        .enqueueUniqueWork(
            "sync_worker",
            ExistingWorkPolicy.KEEP,
            request
        )
}
```

---

# ▶️ 3. Trigger Worker

Call this when:

* App starts
* User logs in
* Data needs syncing

```kotlin
startSyncWorker(context)
```

---

# 🔁 Retry Behavior

| Condition   | Result                       |
| ----------- | ---------------------------- |
| Success     | Work completes               |
| Failure     | Retry triggered              |
| Exception   | Retry triggered              |
| No Internet | Wait until network available |

---

# ⚠️ Important Configurations

### 🌐 Network Constraint

```kotlin
setRequiredNetworkType(NetworkType.CONNECTED)
```

✔ Ensures worker runs only when internet is available

---

### 🔄 Backoff Policy

```kotlin
setBackoffCriteria(
    BackoffPolicy.EXPONENTIAL,
    10,
    TimeUnit.SECONDS
)
```

✔ Retry delay increases exponentially

---

### 🔒 Unique Work

```kotlin
enqueueUniqueWork(
    "sync_worker",
    ExistingWorkPolicy.KEEP,
    request
)
```

✔ Prevents duplicate workers
✔ Keeps existing running worker

---

# 🧠 Best Practices

### ✅ Do

* Use `CoroutineWorker` for suspend functions
* Keep logic inside UseCase
* Handle exceptions properly
* Use constraints wisely

---

### ❌ Avoid

* Running heavy work on main thread
* Creating multiple workers for same task
* Ignoring retry logic
* Doing network calls without constraints

---

# 🚀 Production Improvements

### 🔹 Add Logging

Use:
Timber

```kotlin
Timber.d("Sync started")
```

---

### 🔹 Add Input/Output Data (Optional)

```kotlin
inputData.getString("key")
```

---

### 🔹 Add Periodic Sync (Optional)

```kotlin
PeriodicWorkRequestBuilder<SyncWorker>(
    15, TimeUnit.MINUTES
)
```

---

### 🔹 Handle Foreground Work (Large Tasks)

```kotlin
setForegroundAsync(...)
```

---

# 🎯 Summary

This WorkManager setup provides:

* ✅ Reliable background execution
* 🔁 Automatic retry mechanism
* 🌐 Network-aware execution
* 🔒 Single worker enforcement
* 🧠 Clean architecture (UseCase driven)

---
