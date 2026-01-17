## **Работа с JSON в Go**

### **1. Введение в JSON и пакет encoding/json**

JSON (JavaScript Object Notation) — это легковесный формат обмена данными, который легко читать и писать людям, и легко анализировать и генерировать машинам. В Go работа с JSON осуществляется с помощью пакета `encoding/json`.

### **2. Кодирование структур Go в JSON (Marshal)**

**Код:**

```go
package main

import (
    "encoding/json"
    "fmt"
)

type person struct {
    FirstName string `json:"first_name"`
    Age       int    `json:"age"`
}

func main() {
    p := person{
        FirstName: "John",
        Age:       30,
    }

    jsonData, err := json.Marshal(p)
    if err != nil {
        fmt.Println("Error marshalling to JSON:", err)
        return
    }

    fmt.Println(string(jsonData))
}
```

**Объяснение:**

1. Создаем структуру `person` с двумя полями: `FirstName` и `Age`.
2. Используем теги структуры (в обратных кавычках `` `json:"field_name"` ``) для указания, как поля должны называться в JSON.
3. `json.Marshal()` преобразует структуру Go в JSON (байтовый срез).
4. Преобразуем байтовый срез в строку для вывода.

**Вывод в консоли:**

```json
{ "first_name": "John", "age": 30 }
```

### **3. Теги структур и их использование**

**Теги структур** — это метаданные, которые добавляются к полям структуры между обратными кавычками. Они особенно полезны для:

- Сопоставления имен полей Go с ключами JSON
- Указания ORM для работы с базами данных
- Добавления правил валидации

**Пример тегов для разных целей:**

```go
type User struct {
    UserID string `json:"user_id" db:"user_id"`
    Name   string `json:"name" db:"full_name"`
}
```

### **4. Использование omitempty для пропуска пустых полей**

**Код:**

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Person struct {
    FirstName string `json:"first_name"`
    Age       int    `json:"age,omitempty"`
    Email     string `json:"email,omitempty"`
}

func main() {
    p1 := Person{
        FirstName: "John",
        Age:       0,  // будет пропущено из-за omitempty
        Email:     "", // будет пропущено из-за omitempty
    }

    jsonData, err := json.Marshal(p1)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }

    fmt.Println("Without age and email:", string(jsonData))

    p2 := Person{
        FirstName: "Jane",
        Age:       25,
        Email:     "jane@example.com",
    }

    jsonData2, _ := json.Marshal(p2)
    fmt.Println("With age and email:", string(jsonData2))
}
```

**Объяснение:**

- `omitempty` указывает пропускать поле, если оно содержит нулевое значение (0 для чисел, "" для строк, nil для указателей).
- **Важно:** не должно быть пробелов между `json:"age"` и `omitempty`.

**Вывод в консоли:**

```json
Without age and email: {"first_name":"John"}
With age and email: {"first_name":"Jane","age":25,"email":"jane@example.com"}
```

### **5. Вложенные структуры в JSON**

**Код:**

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Address struct {
    City  string `json:"city"`
    State string `json:"state"`
}

type Person struct {
    FirstName string  `json:"first_name"`
    Age       int     `json:"age"`
    Email     string  `json:"email,omitempty"`
    Address   Address `json:"address"`
}

func main() {
    person1 := Person{
        FirstName: "Jane",
        Age:       30,
        Email:     "jane@fakegmail.com",
        Address: Address{
            City:  "New York",
            State: "NY",
        },
    }

    jsonData1, err := json.Marshal(person1)
    if err != nil {
        fmt.Println("Error marshalling to JSON:", err)
        return
    }

    fmt.Println(string(jsonData1))
}
```

**Вывод в консоли:**

```json
{
  "first_name": "Jane",
  "age": 30,
  "email": "jane@fakegmail.com",
  "address": {
    "city": "New York",
    "state": "NY"
  }
}
```

### **6. Декодирование JSON в структуры Go (Unmarshal)**

**Код:**

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Address struct {
    City  string `json:"city"`
    State string `json:"state"`
}

type Employee struct {
    FullName   string  `json:"full_name"`
    EmployeeID string  `json:"emp_id"`
    Age        int     `json:"age"`
    Address    Address `json:"address"`
}

func main() {
    jsonData := `{
        "full_name": "Jenny Doe",
        "emp_id": "0009",
        "age": 30,
        "address": {
            "city": "San Jose",
            "state": "California"
        }
    }`

    var employeeFromJSON Employee

    err := json.Unmarshal([]byte(jsonData), &employeeFromJSON)
    if err != nil {
        fmt.Println("Error unmarshalling JSON:", err)
        return
    }

    fmt.Printf("%+v\n", employeeFromJSON)

    // Доступ к полям структуры
    fmt.Println("\nJenny's age incremented by 5 years:", employeeFromJSON.Age+5)
    fmt.Println("Jenny's city:", employeeFromJSON.Address.City)
}
```

**Объяснение:**

1. Создаем строку JSON с помощью raw string literal (обратные кавычки).
2. `json.Unmarshal()` принимает байтовый срез JSON и указатель на структуру.
3. **Важно:** передавать указатель (`&employeeFromJSON`), чтобы изменения сохранились.
4. После декодирования можно обращаться к полям структуры.

**Вывод в консоли:**

```
{FullName:Jenny Doe EmployeeID:0009 Age:30 Address:{City:San Jose State:California}}

Jenny's age incremented by 5 years: 35
Jenny's city: San Jose
```

### **7. Работа с массивами/срезами JSON**

**Код:**

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Address struct {
    City  string `json:"city"`
    State string `json:"state"`
}

func main() {
    // Срез структур Address
    listOfCityState := []Address{
        {City: "New York", State: "New York"},
        {City: "San Jose", State: "California"},
    }

    fmt.Println("Original slice:", listOfCityState)

    // Кодирование среза в JSON
    jsonList, err := json.Marshal(listOfCityState)
    if err != nil {
        fmt.Println("Error marshalling to JSON:", err)
        return
    }

    fmt.Println("JSON array:", string(jsonList))
}
```

**Вывод в консоли:**

```
Original slice: [{New York New York} {San Jose California}]
JSON array: [{"city":"New York","state":"New York"},{"city":"San Jose","state":"California"}]
```

### **8. Обработка неизвестных структур JSON с помощью map[string]interface{}**

**Код:**

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
)

func main() {
    jsonData := `{
        "name": "John",
        "age": 30,
        "address": {
            "city": "New York",
            "state": "New York"
        }
    }`

    var data map[string]interface{}

    err := json.Unmarshal([]byte(jsonData), &data)
    if err != nil {
        log.Fatal("Error unmarshalling JSON:", err)
    }

    fmt.Printf("%+v\n", data)

    // Доступ к значениям
    name := data["name"]
    address := data["address"].(map[string]interface{})
    city := address["city"]

    fmt.Println("\nName:", name)
    fmt.Println("City:", city)
}
```

**Объяснение:**

- `map[string]interface{}` может хранить JSON любой структуры.
- Ключи всегда строки (`string`).
- Значения могут быть любого типа (`interface{}`).
- Для доступа к вложенным объектам требуется приведение типа.

**Вывод в консоли:**

```
map[address:map[city:New York state:New York] age:30 name:John]

Name: John
City: New York
```

### **9. Лучшие практики работы с JSON в Go**

1. **Используйте теги структур** для контроля кодирования/декодирования:

   - Сопоставление имен полей
   - Пропуск пустых полей (`omitempty`)
   - Игнорирование полей (`-`)

2. **Валидируйте JSON-данные** перед обработкой, чтобы избежать ошибок времени выполнения.

3. **Всегда обрабатывайте ошибки**, возвращаемые `json.Marshal` и `json.Unmarshal`.

4. **Используйте `omitempty`** для пропуска нулевых значений.

5. **Для неизвестных структур** используйте `map[string]interface{}`.

6. **Используйте raw string literals** (обратные кавычки) для JSON-строк в коде.

### **Пример с тегом `-` для игнорирования поля:**

**Код:**

```go
type User struct {
    Username string `json:"username"`
    Password string `json:"-"` // Никогда не будет включено в JSON
    Email    string `json:"email,omitempty"`
}
```

**Вывод в консоли (если Password = "secret"):**

```json
{ "username": "john_doe", "email": "john@example.com" }
// Поле Password отсутствует
```

---

Пакет `encoding/json` в Go предоставляет мощную и гибкую поддержку для работы с JSON. Понимание основных функций и лучших практик позволяет эффективно работать с JSON в ваших приложениях Go.
