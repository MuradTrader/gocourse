## Объяснение переменных окружения в Go

**Текст автора**: "Environment variables are key value pairs that are part of the environment in which a process runs."

**Объяснение**: Переменные окружения - это пары ключ-значение, которые являются частью среды выполнения процесса.

**Текст автора**: "In go environment, variables are accessed through the OS package."

**Объяснение**: В Go переменные окружения доступны через пакет `os`.

### Шаг за шагом код автора:

```go
package main

import (
    "fmt"
    "os"
    "strings"
)

func main() {
    // ШАГ 1: Получение значения переменной окружения с помощью os.Getenv()
    user := os.Getenv("USER")
    home := os.Getenv("HOME")

    fmt.Println("User environment variable is:", user)
    fmt.Println("Home environment variable is:", home)
    // Вывод (пример):
    // User environment variable is: username
    // Home environment variable is: /home/username

    // ШАГ 2: Установка переменной окружения с помощью os.Setenv()
    err := os.Setenv("FRUIT", "apple")
    if err != nil {
        fmt.Println("Error setting environment variable:", err)
    } else {
        fmt.Println("Environment variable FRUIT set successfully")
    }
    // Вывод: Environment variable FRUIT set successfully

    // ШАГ 3: Получение установленной переменной
    fruit := os.Getenv("FRUIT")
    fmt.Println("Fruit environment variable:", fruit)
    // Вывод: Fruit environment variable: apple

    // ШАГ 4: Получение всех переменных окружения с помощью os.Environ()
    // os.Environ() возвращает срез строк в формате "KEY=VALUE"
    allEnv := os.Environ()

    fmt.Println("\nAll environment variables (keys only):")
    for _, e := range allEnv {
        // Разделяем строку "KEY=VALUE" на две части
        // Используем strings.SplitN с n=2, чтобы разделить только по первому "="
        kvPair := strings.SplitN(e, "=", 2)
        if len(kvPair) == 2 {
            key := kvPair[0]
            // value := kvPair[1]  // Значение не используется в этом примере
            fmt.Println("  ", key)
        }
    }
    // Вывод (пример):
    // All environment variables (keys only):
    //   USER
    //   HOME
    //   PATH
    //   FRUIT
    //   ... и другие переменные окружения

    // ШАГ 5: Удаление переменной окружения с помощью os.Unsetenv()
    err = os.Unsetenv("FRUIT")
    if err != nil {
        fmt.Println("Error unsetting environment variable:", err)
    } else {
        fmt.Println("\nEnvironment variable FRUIT unset successfully")
    }
    // Вывод: Environment variable FRUIT unset successfully

    // ШАГ 6: Проверка, что переменная удалена
    fruitAfterUnset := os.Getenv("FRUIT")
    if fruitAfterUnset == "" {
        fmt.Println("FRUIT variable is now empty (unset)")
    } else {
        fmt.Println("FRUIT variable still exists:", fruitAfterUnset)
    }
    // Вывод: FRUIT variable is now empty (unset)

    // ШАГ 7: Демонстрация разницы между Split и SplitN
    fmt.Println("\n--- Demonstration of strings.SplitN ---")

    str := "a=b=c=d=e"
    fmt.Println("Original string:", str)

    // Разделение с n = -1 (все подстроки)
    resultMinusOne := strings.SplitN(str, "=", -1)
    fmt.Println("SplitN with n=-1:", resultMinusOne)
    // Вывод: SplitN with n=-1: [a b c d e]

    // Разделение с n = 0 (пустой результат)
    resultZero := strings.SplitN(str, "=", 0)
    fmt.Println("SplitN with n=0:", resultZero)
    // Вывод: SplitN with n=0: []

    // Разделение с n = 1 (возвращает всю строку)
    resultOne := strings.SplitN(str, "=", 1)
    fmt.Println("SplitN with n=1:", resultOne)
    // Вывод: SplitN with n=1: [a=b=c=d=e]

    // Разделение с n = 2 (разделяет на 2 части)
    resultTwo := strings.SplitN(str, "=", 2)
    fmt.Println("SplitN with n=2:", resultTwo)
    // Вывод: SplitN with n=2: [a b=c=d=e]

    // Разделение с n = 3 (разделяет на 3 части)
    resultThree := strings.SplitN(str, "=", 3)
    fmt.Println("SplitN with n=3:", resultThree)
    // Вывод: SplitN with n=3: [a b c=d=e]

    // Разделение с n = 4 (разделяет на 4 части)
    resultFour := strings.SplitN(str, "=", 4)
    fmt.Println("SplitN with n=4:", resultFour)
    // Вывод: SplitN with n=4: [a b c d=e]
}
```

### Подробные объяснения автора:

**Текст автора**: "So to save those environment variables I'll introduce a few variables. So first user variable is going to store the user environment variable."

**Объяснение**: Получаем значение переменной окружения с помощью `os.Getenv("KEY")`

**Текст автора**: "So now let's set an environment variable as well. So let's set using the set env."

**Объяснение**: Устанавливаем переменную окружения с помощью `os.Setenv("KEY", "VALUE")`

**Текст автора**: "But now let's print all the environment variables on my system. So how do we do that. We will use Os.environ."

**Объяснение**: Получаем все переменные окружения с помощью `os.Environ()`. Она возвращает срез строк в формате `"KEY=VALUE"`.

**Текст автора**: "So this returns us a list of strings. And because those strings are separated by equal to we are going to split those strings with the delimiter as the equal to sign."

**Объяснение**: Так как `os.Environ()` возвращает строки в формате `"KEY=VALUE"`, мы разделяем их по знаку `=`.

### Разница между `strings.Split` и `strings.SplitN`:

**Текст автора**: "Now the difference between split and split n is that split n gives us a little more functionality than split."

**Объяснение**:

- `strings.Split(s, sep)` - разделяет строку `s` по разделителю `sep` на все возможные части
- `strings.SplitN(s, sep, n)` - разделяет строку `s` по разделителю `sep`, но не более чем на `n` частей

**Пример от автора**:
Строка: `"a=b=c=d=e"`

- `SplitN(s, "=", -1)` → `["a", "b", "c", "d", "e"]` (все части)
- `SplitN(s, "=", 0)` → `[]` (пустой срез)
- `SplitN(s, "=", 1)` → `["a=b=c=d=e"]` (вся строка)
- `SplitN(s, "=", 2)` → `["a", "b=c=d=e"]` (2 части)
- `SplitN(s, "=", 3)` → `["a", "b", "c=d=e"]` (3 части)
- `SplitN(s, "=", 4)` → `["a", "b", "c", "d=e"]` (4 части)

**Текст автора**: "So we just need to separate our result on the basis of the first encounter of the equal to sign. All right. So we need to split the resulting string into only two parts. So we use two."

**Объяснение**: Для переменных окружения формата `"KEY=VALUE"` нам нужно разделить только по первому знаку `=`, поэтому используем `SplitN(e, "=", 2)`.

### Практическое применение переменных окружения:

**Текст автора**: "These environment variables are commonly used for application Configuration, such as database connection strings, API keys, or service endpoints."

**Объяснение**: Переменные окружения используются для:

1. **Конфигурации приложения** - строки подключения к БД, ключи API, URL сервисов
2. **Различных сред** - development, staging, production

**Текст автора**: "They provide a way to configure applications across different environments like development, staging, production, etc. without modifying the source code."

**Объяснение**: Можно настраивать приложение для разных сред (разработка, тестирование, продакшн) без изменения исходного кода.

### Лучшие практики от автора:

**Текст автора**: "Always use secure methods like encrypted configuration files or secrets management systems."

**Объяснение**: Не храните чувствительную информацию в переменных окружения без необходимости. Используйте безопасные методы.

**Текст автора**: "Mostly we use uppercase keys for environment variables, so use all caps for configuring the key for any environment variable."

**Объяснение**: Используйте ВЕРХНИЙ_РЕГИСТР для имен переменных окружения (например, `DATABASE_URL`, `API_KEY`).

**Текст автора**: "And finally, document the required environment variables and their purpose in your project's Readme or documentation."

**Объяснение**: Документируйте необходимые переменные окружения и их назначение в README или документации проекта.

### Важные замечания:

**Текст автора**: "First one being cross-platform compatibility. That means we need to ensure that environment variable usage is compatible across different operating systems."

**Объяснение**: Обеспечивайте кросс-платформенную совместимость при работе с переменными окружения.

**Текст автора**: "And also provide default values in your application code for environment variables to handle cases where they are not set."

**Объяснение**: Предоставляйте значения по умолчанию в коде приложения для переменных окружения.

### Полный пример с выводом:

```go
// Дополнительный пример: практическое использование
package main

import (
    "fmt"
    "os"
)

func main() {
    // Пример: получение переменных окружения с значениями по умолчанию
    getEnvWithDefault := func(key, defaultValue string) string {
        if value := os.Getenv(key); value != "" {
            return value
        }
        return defaultValue
    }

    // Получаем конфигурацию из переменных окружения или используем значения по умолчанию
    dbHost := getEnvWithDefault("DB_HOST", "localhost")
    dbPort := getEnvWithDefault("DB_PORT", "5432")
    dbName := getEnvWithDefault("DB_NAME", "mydb")
    debugMode := os.Getenv("DEBUG") == "true"

    fmt.Println("Database configuration:")
    fmt.Printf("  Host: %s\n", dbHost)
    fmt.Printf("  Port: %s\n", dbPort)
    fmt.Printf("  Name: %s\n", dbName)
    fmt.Printf("  Debug mode: %v\n", debugMode)

    // Пример вывода (без установленных переменных окружения):
    // Database configuration:
    //   Host: localhost
    //   Port: 5432
    //   Name: mydb
    //   Debug mode: false

    // Пример вывода (если установить переменные):
    // export DB_HOST=db.example.com
    // export DB_PORT=3306
    // export DEBUG=true
    // go run env_example.go
    //
    // Вывод:
    // Database configuration:
    //   Host: db.example.com
    //   Port: 3306
    //   Name: mydb
    //   Debug mode: true
}
```

### Итог от автора:

**Текст автора**: "Overall environment variables in go provide a flexible and secure way to configure applications without hardcoding sensitive information."

**Объяснение**: Переменные окружения в Go предоставляют гибкий и безопасный способ настройки приложений без жесткого кодирования чувствительной информации.

**Текст автора**: "By leveraging the OS package, developers can access, set and unset environment variables programmatically to control application behavior across different environments."

**Объяснение**: Используя пакет `os`, разработчики могут получать, устанавливать и удалять переменные окружения программно для управления поведением приложения в разных средах.

### Основные функции пакета `os` для работы с переменными окружения:

1. `os.Getenv("KEY")` - получить значение переменной
2. `os.Setenv("KEY", "VALUE")` - установить переменную
3. `os.Unsetenv("KEY")` - удалить переменную
4. `os.Environ()` - получить все переменные окружения
5. `os.ExpandEnv("string")` - развернуть переменные окружения в строке (например, "Path is: $PATH")
