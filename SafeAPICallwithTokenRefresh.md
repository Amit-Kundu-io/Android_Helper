# 🔐 Safe API Call with Token Refresh (Ktor + Flow)

A production-ready solution for:

* Safe API execution using Kotlin Flow
* Handling 401 Unauthorized errors
* Refreshing expired tokens
* Retrying failed requests automatically

---

## 📌 Architecture Overview

```
UseCase → SafeApiCall → Repository → API
                      ↘ RefreshTokenRepo
```

---

## 📦 1. Network Resource Wrapper

### ✅ Description

Represents API states: Loading, Success, Error

```kotlin
sealed class NetworkResource<T>(val data: T? = null, val message: String? = null) {
    class Success<T>(data: T) : NetworkResource<T>(data)
    class Error<T>(message: String?, data: T? = null) : NetworkResource<T>(data, message)
    class Loading<T>(data: T? = null) : NetworkResource<T>(data)
}
```

---

## 🔁 2. Safe API Call with Retry

### ✅ Description

* Executes API call safely
* Emits Flow states
* Refreshes token on 401
* Retries request automatically

```kotlin
fun <T> safeApiCallWithRetry(
    refreshTokenRepo: RefreshTokenRepo? = null,
    apiCall: suspend () -> T,
    onSuccess: suspend (T) -> Unit = {}
): Flow<NetworkResource<T>> = flow {

    emit(NetworkResource.Loading())

    try {
        val response = apiCall()
        onSuccess(response)
        emit(NetworkResource.Success(response))

    } catch (e: ClientRequestException) {

        if (e.response.status.value == 401 && refreshTokenRepo != null) {

            val isRefreshed = refreshTokenRepo.refreshToken()

            if (isRefreshed) {
                val retryResponse = apiCall()
                onSuccess(retryResponse)
                emit(NetworkResource.Success(retryResponse))
            } else {
                emit(NetworkResource.Error("Session expired. Please login again."))
            }

        } else {
            emit(NetworkResource.Error(e.message ?: "Client error"))
        }

    } catch (e: ServerResponseException) {
        emit(NetworkResource.Error("Server error, please try again"))

    } catch (e: RedirectResponseException) {
        emit(NetworkResource.Error("Unexpected redirect"))

    } catch (e: IOException) {
        emit(NetworkResource.Error("No internet connection"))

    } catch (e: Exception) {
        emit(NetworkResource.Error(e.message ?: "Unknown error"))
    }
}
```

---

## 🔐 3. Refresh Token Repository

### ✅ Interface

```kotlin
interface RefreshTokenRepo {
    suspend fun refreshToken(): Boolean
}
```

---

## 🛠 4. RefreshTokenRepositoryImpl

### ✅ Description

Handles token refresh API and updates local storage

### 📦 Endpoint

```
POST /auth/refresh-token
```

```kotlin
class RefreshTokenRepositoryImpl(
    private val client: HttpClient,
    private val dataStore: DataStore,
) : RefreshTokenRepo {

    override suspend fun refreshToken(): Boolean {

        val token = dataStore.getToken.firstOrNull().orEmpty()
        val refreshToken = dataStore.getRefreshToken.firstOrNull().orEmpty()

        return try {

            val response: RefreshTokenResponse =
                client.post("/refresh-token") {

                    contentType(ContentType.Application.Json)
                    setBody(
                        RefreshTokenRequest(
                            refreshToken = refreshToken,
                            token = token
                        )
                    )
                }.body()

            if (response.succeeded == true) {

                dataStore.setToken(response.data?.token.orEmpty())
                dataStore.setRefreshToken(response.data?.refreshToken.orEmpty())
                dataStore.setTokenExpiredAt(response.data?.tokenExpiredAt ?: 0L)
                dataStore.setRefreshTokenExpiredAt(
                    response.data?.refreshTokenExpiredAt ?: 0L
                )

                true
            } else {
                false
            }

        } catch (e: Exception) {
            false
        }
    }
}
```

---

## 🧾 5. RequestLeaveUseCase

### ✅ Description

Use case to request leave using safe API call

```kotlin
class RequestAddUseCase(
    private val repository: addRepository,
    private val refreshTokenRepository: RefreshTokenRepo,
    private val dataStore: DataStore,
) {

    operator fun invoke(
        reqBody: AddRequest
    ): Flow<NetworkResource<AddResponse>> =
        safeApiCallWithRetry(
            refreshTokenRepo = refreshTokenRepository,
            apiCall = {

                val userId = dataStore.userId.firstOrNull().orEmpty()
                val organizationId = dataStore.getOrganizationId.firstOrNull().orEmpty()
                val token = dataStore.getToken.firstOrNull().orEmpty()

                repository.addLeave(
                    authorization = "Bearer $token",
                    organisationId = organizationId,
                    userId = userId,
                    reqBody = reqBody
                )
            }
        )
}
```

---

## 🔄 Example Usage (ViewModel)

```kotlin
viewModelScope.launch {
    useCase(request).collect { state ->

        when (state) {
            is NetworkResource.Loading -> {
                // Show loader
            }

            is NetworkResource.Success -> {
                // Handle success
            }

            is NetworkResource.Error -> {
                // Show error
            }
        }
    }
}
```

---

## 🧠 Best Practices

### ✅ Do

* Use Flow for async handling
* Keep UseCase clean
* Handle errors centrally
* Use repository for API calls

### ❌ Avoid

* Mixing callbacks with suspend functions
* Emitting inside callbacks
* Hardcoding tokens
* Ignoring 401 errors

---

## 🎯 Summary

This setup provides:

* Clean architecture ✅
* Automatic token refresh 🔐
* Retry mechanism 🔁
* Scalable & production-ready design 🚀

---
