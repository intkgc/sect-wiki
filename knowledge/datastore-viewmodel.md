---
title: Управление настройками с помощью Jetpack DataStore — Простой подход к сохранению данных разных типов
---

## **Введение**

При разработке Android-приложений часто возникает необходимость сохранять пользовательские настройки, такие как выбор темы, язык или другие данные пользователя. Jetpack DataStore — это современное решение для хранения таких данных, которое заменяет устаревший `SharedPreferences`. В этой статье мы рассмотрим, как использовать `DataStore` для хранения различных типов данных (Boolean, Integer, Float и String) и как интегрировать его с архитектурой MVVM, используя `StateFlow` для обновления UI.

---

### **Что такое DataStore?**

Jetpack DataStore — это библиотека для хранения данных, поддерживающая асинхронное и безопасное чтение и запись данных. Она предоставляет два типа хранилищ:

1. **Preferences DataStore**: Используется для хранения пар "ключ-значение" (похож на `SharedPreferences`).
2. **Proto DataStore**: Используется для хранения структурированных данных на основе протокольных буферов.

В этой статье мы сосредоточимся на **Preferences DataStore**, которое идеально подходит для хранения настроек приложения.

---

### **Структура проекта**

1. **DataStoreManager** — Класс, который абстрагирует логику работы с `DataStore` и предоставляет методы для сохранения и получения данных различных типов.
2. **AppViewModel** — `ViewModel`, который обрабатывает логику взаимодействия с `DataStoreManager`. Мы будем использовать `StateFlow` для асинхронного обновления состояния.
3. **Использование в UI** — Примеры того, как использовать `ViewModel` в Activity или Fragment для взаимодействия с данными и обновления UI.

---

### **Необходимые библиотеки**

Чтобы использовать Jetpack DataStore в вашем проекте, необходимо добавить следующие зависимости в файл `build.gradle`:

```gradle
dependencies {
    // Jetpack DataStore
    implementation "androidx.datastore:datastore-preferences:1.0.0"
    
    // Поддержка Kotlin Coroutines
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.0"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.0"

    // Компоненты Lifecycle для ViewModel и LiveData
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.0"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.4.0"
}
```

Эти зависимости обеспечивают поддержку для:

- **DataStore Preferences** для хранения пар "ключ-значение".
- **Kotlin Coroutines** для асинхронных операций.
- **Компонентов Lifecycle** для управления ViewModel и LiveData.

---

### **1. DataStoreManager**

Класс `DataStoreManager` абстрагирует взаимодействие с `DataStore`. Мы создаем универсальные методы для сохранения и чтения данных различных типов: `Boolean`, `Integer`, `Float` и `String`.

#### Код для `DataStoreManager`

```kotlin
package com.example.app

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.*
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

// Создаем DataStore с именем "settings"
private val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")

class DataStoreManager(private val context: Context) {

    // Универсальный метод для сохранения данных
    suspend fun <T> save(key: Preferences.Key<T>, value: T) {
        context.dataStore.edit { settings ->
            settings[key] = value
        }
    }

    // Универсальный метод для чтения данных
    fun <T> read(key: Preferences.Key<T>, defaultValue: T): Flow<T> = context.dataStore.data
        .map { settings ->
            settings[key] ?: defaultValue
        }

    // Пример для работы с Boolean
    private val BOOLEAN_KEY = booleanPreferencesKey("boolean_key")
    suspend fun setBoolean(value: Boolean) = save(BOOLEAN_KEY, value)
    fun getBoolean(): Flow<Boolean> = read(BOOLEAN_KEY, false)

    // Пример для работы с Integer
    private val INTEGER_KEY = intPreferencesKey("integer_key")
    suspend fun setInteger(value: Int) = save(INTEGER_KEY, value)
    fun getInteger(): Flow<Int> = read(INTEGER_KEY, 0)

    // Пример для работы с Float
    private val FLOAT_KEY = floatPreferencesKey("float_key")
    suspend fun setFloat(value: Float) = save(FLOAT_KEY, value)
    fun getFloat(): Flow<Float> = read(FLOAT_KEY, 0f)

    // Пример для работы с String
    private val STRING_KEY = stringPreferencesKey("string_key")
    suspend fun setString(value: String) = save(STRING_KEY, value)
    fun getString(): Flow<String> = read(STRING_KEY, "")
}
```

#### Описание методов:
- **save(key: Preferences.Key<T>, value: T)** — Сохраняет данные в `DataStore`. Метод принимает ключ и значение для сохранения.
- **read(key: Preferences.Key<T>, defaultValue: T)** — Читает данные из `DataStore`. Если данные отсутствуют, возвращает значение по умолчанию.
- Методы для каждого типа данных (`setBoolean`, `getBoolean`, `setInteger`, `getInteger` и т. д.) облегчают работу с `Boolean`, `Integer`, `Float`, `String`.

---

### **2. AppViewModel**

`AppViewModel` обрабатывает логику взаимодействия с `DataStoreManager` и предоставляет реактивное поведение с использованием `StateFlow`. Каждый тип данных (`Boolean`, `Integer`, `Float`, `String`) представляет поток данных, который можно наблюдать в UI.

#### Код для `AppViewModel`

```kotlin
package com.example.app

import android.app.Application
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.viewModelScope
import com.byteflipper.soulplayer.data.DataStoreManager
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class AppViewModel(application: Application) : AndroidViewModel(application) {

    private val dataStoreManager = DataStoreManager(application)

    // Обработчик для Boolean
    val booleanValue: StateFlow<Boolean> = dataStoreManager.getBoolean()
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), false)
    fun setBoolean(value: Boolean) {
        viewModelScope.launch(Dispatchers.IO) {
            dataStoreManager.setBoolean(value)
        }
    }

    // Обработчик для Integer
    val integerValue: StateFlow<Int> = dataStoreManager.getInteger()
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), 0)
    fun setInteger(value: Int) {
        viewModelScope.launch(Dispatchers.IO) {
            dataStoreManager.setInteger(value)
        }
    }

    // Обработчик для Float
    val floatValue: StateFlow<Float> = dataStoreManager.getFloat()
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), 0f)
    fun setFloat(value: Float) {
        viewModelScope.launch(Dispatchers.IO) {
            dataStoreManager.setFloat(value)
        }
    }

    // Обработчик для String
    val stringValue: StateFlow<String> = dataStoreManager.getString()
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), "")
    fun setString(value: String) {
        viewModelScope.launch(Dispatchers.IO) {
            dataStoreManager.setString(value)
        }
    }

    // ViewModelFactory для создания AppViewModel
    class AppViewModelFactory(private val application: Application) : ViewModelProvider.Factory {
        override fun <T : ViewModel> create(modelClass: Class<T>): T {
            if (modelClass.isAssignableFrom(AppViewModel::class.java)) {
                return AppViewModel(application) as T
            }
            throw IllegalArgumentException("Unknown ViewModel class")
        }
    }
}
```

#### Описание свойств и методов:
- `StateFlow` для каждого типа данных (например, `booleanValue`, `integerValue`, `floatValue`, `stringValue`) позволяет наблюдать изменения состояния.
- **setBoolean(value: Boolean)** — Метод для сохранения нового значения для `Boolean`.
- **setInteger(value: Int)** — Метод для сохранения нового значения для `Integer`.
- **setFloat(value: Float)** — Метод для сохранения нового значения для `Float`.
- **setString(value: String)** — Метод для сохранения нового значения для `String`.

---

### **3. Использование в приложении**

Чтобы использовать `AppViewModel` в `Activity` или `Fragment`, создайте экземпляр `ViewModel` и подпишитесь на изменения состояния.

#### Пример использования в `Activity` или `Fragment`

```kotlin
val viewModel: AppViewModel by viewModels {
    AppViewModel.AppViewModelFactory(application)
}

// Подписка на изменения данных
viewModel.booleanValue.observe(this) { isEnabled ->
    // Обновление UI на основе значения isEnabled
}

viewModel.integerValue.observe(this) { value ->
    // Обновление UI на основе значения
}

viewModel.floatValue.observe(this) { value ->
    // Обновление UI на основе значения
}

viewModel.stringValue.observe(this) { text ->
    // Обновление UI на основе текста
}

// Сохранение данных
viewModel.setBoolean(true)
viewModel.setInteger(42)
viewModel.setFloat(3.14f)
viewModel.setString("Hello, DataStore!")
```

---

### **Заключение**

В этой статье мы рассмотрели, как использовать Jetpack `DataStore` для управления настройками приложения, предоставляя простой способ работы с различными типами данных, такими как `Boolean`, `Integer`, `Float` и `String`. Мы продемонстрировали, как интегрировать `DataStore` с архитектурой MVVM, используя `StateFlow` для асинхронных обновлений UI. Этот подход помогает сделать хранение и доступ к данным более эффективным, безопасным и реактивным.

---

**Контакты автора статьи**  
- **Telegram:** [@ByteFlipper](https://t.me/byteflipper)  
- **Вебсайт:** [https://byteflipper.web.app/](https://byteflipper.web.app/)  
- **Канал Telegram:** [t.me/byteflipper](https://t.me/byteflipper)  
- **Электронная почта:** [byteflipper.business@gmail.com](mailto:byteflipper.business@gmail.com)  
- **Личный Telegram:** [@ibremminer837](https://t.me/ibremminer837)
