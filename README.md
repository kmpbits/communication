# Communication Android

[![](https://jitpack.io/v/kmpbits/communication.svg)](https://jitpack.io/#kmpbits/communication)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A lightweight, flexible network library for Android written in Kotlin. The Communication library provides a clean and intuitive API for handling network requests with support for LiveData, Flow, object deserialization, and pagination.

## 🌟 Features

- **Multiple Response Types**
  - Support for LiveData responses
  - Support for Flow responses
  - Direct object deserialization
  
- **Android Architecture Components Integration**
  - Seamless integration with Android ViewModel and LiveData
  - Coroutines and Flow support for reactive programming

- **Pagination Support**
  - Built-in pagination handling with Jetpack Paging 3 library
  - Customizable page parameters

- **Flexibility**
  - Customizable headers and parameters
  - Support for all HTTP methods
  - Easy to configure base URL

- **Local Data Integration**
  - Support for caching and local data observation
  - Callbacks for handling successful network responses

## 📦 Installation

### Step 1: Add JitPack repository

In your root `settings.gradle` file, add the JitPack repository:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

### Step 2: Add the dependencies

In your app module's `build.gradle` file:

```kotlin
dependencies {
    // Core module (required)
    implementation("com.github.kmpbits.communication:communication-core:latest_version")
    
    // Android-specific extensions (optional)
    // Only if you need LiveData responses
    implementation("com.github.kmpbits.communication:communication-android:latest_version")

    // Pagination support (optional)
    // Only if you need pagination functionality
    implementation("com.github.kmpbits.communication:communication-paging:latest_version")
}
```

Replace `latest_version` with the current release version from JitPack.

## 🚀 Getting Started

### Initialize the Client

Create a client instance in your application class, a singleton, or your dependency injection setup:

```kotlin
val client = communicationClient {
    baseUrl = "https://api.example.com"
    
    // Optional: Add default headers
    header(Header(HttpHeader.CONTENT_TYPE, "application/json"))
    header(Header(HttpHeader.custom("custom-header"), "This is a custom header"))
}
```

### Making Simple Requests

#### Basic Request

```kotlin
val response: CommunicationResponse = client.call {
    path = "/users"
    method = HttpMethod.Get
}.response()
```

#### Deserialize to a Model

```kotlin
val user: User = client.call {
    path = "/users/1"
}.responseToModel<User>()
```

### Working with Flow

Retrieve data as a Flow with built-in state handling:

```kotlin
val userFlow: Flow<ResultState<User>> = client.call {
    path = "/users/1"
    parameter("include" to "details")
}.responseFlow()
```

You can also customize the flow response behavior:

```kotlin
val usersFlow = client.call {
    path = "/users"
    method = HttpMethod.Get
}.responseFlow<List<User>> {
    // Handle successful network response
    onNetworkSuccess { users ->
        // Save to local database
        userDao.insertAll(users)
    }
    
    // Observe local data source
    local {
        observe { userDao.getAllUsers() }
    }
}
```

### Observing Flow Responses

Using lifecycle-aware collection:

```kotlin
userFlow.observe(viewLifecycleOwner) { state ->
    when(state) {
        is ResultState.Loading -> showLoading()
        is ResultState.Success -> showUsers(state.data)
        is ResultState.Error -> showError(state.exception.message)
        is ResultState.Empty -> showEmptyState()
    }
}
```

Using coroutines:

```kotlin
lifecycleScope.launch {
    userFlow.collectLatest { state ->
        when(state) {
            is ResultState.Loading -> showLoading()
            is ResultState.Success -> showUsers(state.data)
            is ResultState.Error -> showError(state.exception.message)
            is ResultState.Empty -> showEmptyState()
        }
    }
}
```

### Pagination (Paging Extension)

Easy integration with Android's Paging 3 library:

```kotlin
val pagedUsers: Flow<PagingData<User>> = client.call {
    path = "/users"
    parameter("size" to 20)
}.responsePaginated {
    // Use API-only pagination or combine with local storage
    onlyApiCall = true
    
    // Customize the page parameter name (default is "page")
    pageQueryName = "page"
    
    // If using local storage (when onlyApiCall = false)
    pagingSource { userDao.getPagingSource() }
    deleteAll { userDao.deleteAll() }
    insertAll { users -> userDao.insertAll(users) }
    
    // Optional: Optimize initial loading state
    firstItemDatabase { userDao.getFirstUser() }
}
```

### Handling Pagination States

```kotlin
// Observe loading states
lifecycleScope.launch { 
    adapter.loadStateFlow.collectLatest { loadState ->
        // Handle refresh loading state
        binding.progressBar.isVisible = loadState.refresh is LoadState.Loading
        
        // Handle refresh error state
        if (loadState.refresh is LoadState.Error) {
            val error = (loadState.refresh as LoadState.Error).error
            showErrorMessage(error.message)
        }
    }
}

// Add loading footer
recyclerView.adapter = userAdapter.withLoadStateFooter(
    footer = LoadingStateAdapter(
        retry = { userAdapter.retry() }
    )
)
```

## 📋 Advanced Configuration

### Custom Headers

```kotlin
client.call {
    path = "/secure-endpoint"
    header(Header(HttpHeader.custom("X-Custom-Header"), "CustomValue"))
}.responseFlow<SecureData>()
```

### Request Parameters

```kotlin
client.call {
    path = "/users"
    parameter("role" to "admin")
    parameter("active" to true)
}.responseFlow<List<User>>()
```

### Error Handling

The library provides several ways to handle errors:

```kotlin
try {
    val response = client.call {
        path = "/might-fail"
    }.responseToModel<Data>()
} catch (e: stateTalkException) {
    // Handle stateTalk errors
    when (e) {
        is NetworkException -> // Handle network errors
        is SerializationException -> // Handle parsing errors
        is HttpException -> {
            // Access HTTP error details
            val statusCode = e.code
            val errorBody = e.errorBody
        }
    }
}
```

## 🤝 Contributing

Contributions are welcome! Feel free to open issues or submit pull requests.

1. Fork the project
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
