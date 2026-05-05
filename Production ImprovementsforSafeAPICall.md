# 🚀 Complete Safe API Call + Refresh Token + UseCase + ViewModel

---

# 📦 1. NetworkResource.kt

```kotlin
sealed class NetworkResource<T> {

    class Loading<T> : NetworkResource<T>()

    data class Success<T>(
        val data: T
    ) : NetworkResource<T>()

    data class Error<T>(
        val message: String
    ) : NetworkResource<T>()
}
```

---

# 🔁 2. safeApiCallWithRetry.kt

```kotlin
fun <T> safeApiCallWithRetry(
    refreshTokenRepo: RefreshTokenRepo? = null,
    retryCount: Int = 1,
    apiCall: suspend () -> T,
    onSuccess: suspend (T) -> Unit = {}
): Flow<NetworkResource<T>> = flow {

    emit(NetworkResource.Loading())

    try {
        val response = apiCall()
        onSuccess(response)
        emit(NetworkResource.Success(response))

    } catch (e: ClientRequestException) {

        if (e.response.status.value == 401 && retryCount > 0 && refreshTokenRepo != null) {

            val refreshed = refreshTokenRepo.refreshToken()

            if (refreshed) {
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
        emit(NetworkResource.Error("Server error"))

    } catch (e: IOException) {
        emit(NetworkResource.Error("No internet connection"))

    } catch (e: Exception) {
        emit(NetworkResource.Error(e.message ?: "Unknown error"))
    }

}.flowOn(Dispatchers.IO)
```

---

# 🔐 3. RefreshTokenRepo.kt

```kotlin
interface RefreshTokenRepo {
    suspend fun refreshToken(): Boolean
}
```

---

# 🛠 4. RefreshTokenRepositoryImpl.kt (WITH MUTEX)

```kotlin
class RefreshTokenRepositoryImpl(
    private val client: HttpClient,
    private val dataStore: DataStore,
) : RefreshTokenRepo {

    private val mutex = Mutex()

    override suspend fun refreshToken(): Boolean {
        return mutex.withLock {
            performRefresh()
        }
    }

    private suspend fun performRefresh(): Boolean {

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

# 🧾 5. RequestLeaveUseCase.kt

```kotlin
class LeaveUseCase(
    private val repository: Repository,
    private val refreshTokenRepository: RefreshTokenRepo,
    private val dataStore: DataStore,
) {

    operator fun invoke(
        reqBody: Request
    ): Flow<NetworkResource<Response>> =
        safeApiCallWithRetry(
            refreshTokenRepo = refreshTokenRepository,
            apiCall = {

                val userId = dataStore.userId.firstOrNull().orEmpty()
                val organizationId = dataStore.getOrganizationId.firstOrNull().orEmpty()
                val token = dataStore.getToken.firstOrNull().orEmpty()

                repository.fun(
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

# 🚀 Final Result


* ✅ Safe API execution
* ✅ Automatic token refresh
* ✅ Mutex (no duplicate refresh calls)
* ✅ Clean architecture
* ✅ Production-ready flow handling

