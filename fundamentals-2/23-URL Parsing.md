### **1. Что такое URL parsing в Go?**

Это извлечение различных компонентов из строки URL: схемы (протокола), хоста, пути, параметров запроса и т.д. Это важно для веб-приложений, API-эндпоинтов и обработки URL.

---

### **2. Структура URL (Uniform Resource Locator):**

```
scheme://userinfo@host:port/path?query#fragment
```

- **Scheme (протокол)**: HTTP, HTTPS, FTP.
- **Userinfo**: имя пользователя и пароль (необязательно).
- **Host**: доменное имя или IP-адрес.
- **Port**: номер порта (необязательно).
- **Path**: путь к ресурсу на сервере.
- **Query**: параметры запроса в формате `ключ=значение`.
- **Fragment**: идентификатор фрагмента (необязательно, указывает на часть ресурса).

---

### **3. Пакет `net/url`**

Стандартная библиотека Go предоставляет пакет `net/url` для разбора и манипуляции URL.

---

### **4. Пример 1: Разбор URL на компоненты**

**Код:**

```go
rawURL := "https://example.com:8080/path?query=value#fragment"
parsedURL, err := url.Parse(rawURL)
if err != nil {
    fmt.Println("Error parsing URL:", err)
    return
}
fmt.Println("Scheme:", parsedURL.Scheme)
fmt.Println("Host:", parsedURL.Host)
fmt.Println("Port:", parsedURL.Port())
fmt.Println("Path:", parsedURL.Path)
fmt.Println("Raw Query:", parsedURL.RawQuery)
fmt.Println("Fragment:", parsedURL.Fragment)
```

**Вывод в консоли:**

```
Scheme: https
Host: example.com:8080
Port: 8080
Path: /path
Raw Query: query=value
Fragment: fragment
```

**Что происходит:**

1. `url.Parse()` принимает строку URL и возвращает структуру `*url.URL` и ошибку.
2. Компоненты URL извлекаются через поля структуры: `Scheme`, `Host`, `Path`, `RawQuery`, `Fragment`.
3. `Port()` — метод, извлекающий порт из `Host`.

---

### **5. Пример 2: Извлечение параметров запроса (query parameters)**

**Код:**

```go
rawURL1 := "https://example.com/path?name=John&age=30"
parsedURL1, err := url.Parse(rawURL1)
if err != nil {
    fmt.Println("Error parsing URL:", err)
    return
}
queryParams := parsedURL1.Query()
fmt.Println("Query params:", queryParams)
fmt.Println("Name:", queryParams.Get("name"))
fmt.Println("Age:", queryParams.Get("age"))
```

**Вывод в консоли:**

```
Query params: map[age:[30] name:[John]]
Name: John
Age: 30
```

**Что происходит:**

1. `Query()` возвращает `url.Values` (тип `map[string][]string`), содержащий параметры запроса.
2. `Get(key)` возвращает первое значение для ключа `key`.

---

### **6. Пример 3: Сборка URL с параметрами запроса**

**Способ 1: Использование `url.URL` и `url.Values`**
**Код:**

```go
baseURL := &url.URL{
    Scheme: "https",
    Host:   "example.com",
    Path:   "/path",
}
query := baseURL.Query()
query.Set("name", "John")
query.Set("age", "30")
baseURL.RawQuery = query.Encode()
fmt.Println("Built URL:", baseURL.String())
```

**Вывод в консоли:**

```
Built URL: https://example.com/path?age=30&name=John
```

**Что происходит:**

1. Создаётся структура `url.URL` с полями `Scheme`, `Host`, `Path`.
2. `Query()` возвращает `url.Values`.
3. `Set(key, value)` устанавливает значение для ключа (заменяя существующие).
4. `Encode()` преобразует параметры в строку вида `key=value&key2=value2`.
5. `RawQuery` присваивает закодированную строку параметров.
6. `String()` возвращает собранный URL в виде строки.

---

**Способ 2: Использование `url.Values` напрямую**
**Код:**

```go
values := url.Values{}
values.Add("name", "Jane")
values.Add("age", "30")
values.Add("city", "London")
values.Add("country", "UK")
encodedQuery := values.Encode()
fmt.Println("Encoded query:", encodedQuery)
baseURL1 := "https://example.com/search"
fullURL := baseURL1 + "?" + encodedQuery
fmt.Println("Full URL:", fullURL)
```

**Вывод в консоли:**

```
Encoded query: age=30&city=London&country=UK&name=Jane
Full URL: https://example.com/search?age=30&city=London&country=UK&name=Jane
```

**Что происходит:**

1. `url.Values{}` создаёт пустой набор параметров.
2. `Add(key, value)` добавляет значение к ключу (можно несколько значений для одного ключа).
3. `Encode()` кодирует параметры в строку.
4. Собираем полный URL, добавляя `?` и закодированные параметры к базовому URL.

---

### **7. Что означает «parsing»?**

В программировании «parsing» означает обработку данных для извлечения информации или преобразования формата. В контексте чисел — преобразование строк в числа. В контексте URL — извлечение компонентов.

---

### **8. Обработка ошибок**

Всегда обрабатывайте ошибки при разборе URL, так как строка может быть некорректной. Используйте проверку `if err != nil`.

---

### **Итог:**

- Для разбора URL используйте пакет `net/url`.
- `url.Parse()` разбирает строку URL на компоненты.
- `Query()` возвращает параметры запроса как `url.Values` (map).
- `url.Values` методы: `Set()`, `Add()`, `Get()`, `Encode()`.
- `url.URL` структура позволяет собирать URL программно.
- Всегда обрабатывайте ошибки.
