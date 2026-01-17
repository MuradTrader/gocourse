## Часть 1: Основы текстовых шаблонов

### Что такое текстовые шаблоны?

**Текстовые шаблоны** (text templates) в Go - это мощный механизм для генерации текстового вывода на основе шаблонов и данных. Представь, что у тебя есть форма (шаблон) с пустыми полями, и ты заполняешь эти поля данными из программы.

**Аналогия**: Как в Word есть шаблоны документов с заполнителями вроде `{Имя}`, `{Дата}`, которые потом заменяются реальными значениями.

### Зачем они нужны?

1. **Генерация HTML** - создание веб-страниц
2. **Формирование JSON/XML** - сериализация данных
3. **SQL-запросы** - безопасное построение запросов
4. **Отчёты и документы** - создание структурированного текста

## Часть 2: Основные понятия

### Шаблон (Template)

Это строка или файл, содержащий:

- Обычный текст (неизменяемая часть)
- **Действия (actions)** - динамические части, которые заменяются значениями

Пример шаблона:

```
"Добро пожаловать, {{.Name}}! Ваш баланс: {{.Balance}} руб."
```

Здесь:

- `"Добро пожаловать, "` и `"! Ваш баланс: "` - обычный текст
- `{{.Name}}` и `{{.Balance}}` - действия (будут заменены значениями)

### Действия (Actions)

Действия заключаются в **двойные фигурные скобки `{{}}`**.

**Типы действий:**

1. **Вставка переменных**: `{{.ИмяПоля}}`
2. **Конвейеры (pipelines)**: `{{функция .Поля}}`
3. **Управляющие структуры**:
   - Условные операторы: `{{if .Условие}}...{{else}}...{{end}}`
   - Циклы: `{{range .Список}}...{{end}}`

**Важное замечание**: Точка (`.`) в шаблонах - это указатель на текущие данные.

- `.Name` означает "взять поле Name из текущих данных"
- В цикле `{{range .Users}}` точка внутри блока будет указывать на текущий элемент массива

## Часть 3: Пакеты для работы с шаблонами

В Go есть два пакета:

1. **`text/template`** - базовые возможности
2. **`html/template`** - то же самое + безопасность для HTML (автоматическое экранирование)

**Важный момент**: Даже если импортировать большой пакет (как `html/template`), Go компилятор использует **tree shaking** - удаляет неиспользуемые части, поэтому итоговый бинарник не становится больше.

## Часть 4: Простой пример - разбор пошагово

### Шаг 1: Импорт пакетов

```go
import (
    "html/template"  // или "text/template"
    "os"
)
```

Автор импортирует `html/template`, но в коде переменная почему-то называется `tmpl`. Это опечатка в тексте.

### Шаг 2: Создание шаблона

```go
tmpl := template.New("example")
```

- `template.New()` создаёт новый шаблон с именем "example"
- Имя нужно для отладки и для работы с несколькими шаблонами

### Шаг 3: Парсинг шаблона

```go
tmpl, err := tmpl.Parse("Welcome {{.Name}}! How are you doing?")
```

**Что происходит:**

1. Метод `Parse()` принимает строку с шаблоном
2. Анализирует её, находит все действия (`{{.Name}}`)
3. Создаёт внутреннюю структуру для быстрого выполнения

### Шаг 4: Подготовка данных

```go
data := map[string]interface{}{
    "Name": "John",
}
```

**Что такое `map[string]interface{}`?**
Это карта (словарь), где:

- Ключи - строки (`string`)
- Значения - любые типы (`interface{}` - пустой интерфейс, может содержать что угодно)

**Альтернатива** - можно использовать структуры:

```go
type User struct {
    Name string
}
data := User{Name: "John"}
```

### Шаг 5: Выполнение шаблона

```go
err = tmpl.Execute(os.Stdout, data)
```

**Что делает `Execute()`:**

1. Применяет шаблон к данным
2. Заменяет `{{.Name}}` на значение из `data["Name"]`
3. Выводит результат в `os.Stdout` (стандартный вывод - консоль)

## Часть 5: Упрощение с помощью `template.Must`

Автор показывает альтернативный способ:

```go
tmpl := template.Must(template.New("example").Parse("Welcome {{.Name}}! How are you doing?"))
```

**Что такое `template.Must`?**
Это вспомогательная функция, которая:

- Если парсинг успешен - возвращает шаблон
- Если есть ошибка - вызывает `panic`

**Почему это удобно?**
Вместо:

```go
tmpl, err := template.New("example").Parse(...)
if err != nil { panic(err) }
```

Можно написать одну строку:

```go
tmpl := template.Must(template.New("example").Parse(...))
```

**Важно**: Используй `Must` только если ты уверен, что шаблон корректен (например, при старте программы). Для динамически создаваемых шаблонов лучше обрабатывать ошибки.

## Часть 6: Консольное приложение с меню

Этот пример демонстрирует более сложное использование шаблонов.

### Шаг 1: Чтение ввода пользователя

```go
reader := bufio.NewReader(os.Stdin)
name, _ := reader.ReadString('\n')
name = strings.TrimSpace(name)
```

**Что здесь происходит:**

1. `bufio.NewReader(os.Stdin)` создаёт буферизованный ридер для чтения из консоли
2. `ReadString('\n')` читает строку до символа новой строки
3. `TrimSpace()` удаляет пробелы и символ новой строки в начале и конце

### Шаг 2: Определение нескольких шаблонов

```go
templates := map[string]string{
    "welcome":      "Welcome {{.NM}}! We are glad you joined.\n",
    "notification": "{{.NM}}, you have a new notification: {{.NTF}}\n",
    "error":        "Oops! An error occurred: {{.EM}}\n",
}
```

**Почему используется map?**

- Удобно хранить несколько шаблонов
- Можно легко добавлять/удалять шаблоны
- Доступ по имени: `templates["welcome"]`

### Шаг 3: Динамическое создание шаблонов

```go
parsedTemplates := make(map[string]*template.Template)
for name, tmpl := range templates {
    parsedTemplates[name] = template.Must(template.New(name).Parse(tmpl))
}
```

**Разберём эту магию:**

1. `make(map[string]*template.Template)` создаёт пустую карту для хранения уже распарсенных шаблонов
2. Цикл `for name, tmpl := range templates` проходит по всем шаблонам
3. Для каждого создаётся и парсится шаблон
4. Результат сохраняется в `parsedTemplates`

**Почему это эффективно?**
Если бы у нас было 100 шаблонов, код остался бы почти таким же!

### Шаг 4: Работа с меню

**Бесконечный цикл:**

```go
for {
    // показываем меню
}
```

Это аналог `while(true)` в других языках.

**Switch-case для обработки выбора:**

```go
switch choice {
case "1":
    // обработка выбора 1
case "2":
    // обработка выбора 2
default:
    // обработка неправильного выбора
}
```

### Шаг 5: Динамические данные для каждого случая

Для каждого пункта меню создаются свои данные:

**Для welcome:**

```go
data := map[string]interface{}{
    "NM": name,  // имя пользователя
}
tmpl := parsedTemplates["welcome"]
tmpl.Execute(os.Stdout, data)
```

**Для notification:**

```go
data := map[string]interface{}{
    "NM":  name,          // имя пользователя
    "NTF": notification,  // текст уведомления
}
```

## Часть 7: Важные нюансы

### 1. Соответствие ключей

**Очень важно**: Ключи в данных должны совпадать с именами в шаблоне!

Работает:

```go
// Шаблон
"{{.Name}}"
// Данные
map[string]interface{}{"Name": "John"}
```

Не работает:

```go
// Шаблон
"{{.Name}}"
// Данные
map[string]interface{}{"UserName": "John"}  // Ошибка!
```

### 2. Безопасность HTML

Автор упоминает, что `html/template` безопаснее для генерации HTML. Почему?

**Пример:**

```go
// С text/template (небезопасно)
data := map[string]interface{}{"Content": "<script>alert('xss')</script>"}
tmpl, _ := template.New("test").Parse("<div>{{.Content}}</div>")
// Результат: <div><script>alert('xss')</script></div> - XSS уязвимость!

// С html/template (безопасно)
tmpl, _ := template.New("test").Parse("<div>{{.Content}}</div>")
// Результат: <div>&lt;script&gt;alert('xss')&lt;/script&gt;</div> - безопасно!
```

### 3. Tree Shaking в Go

Автор упоминает, что не нужно бояться импортировать большие пакеты. Go компилятор удаляет неиспользуемый код. Это называется **dead code elimination**.

## Часть 8: Практические применения

1. **Веб-приложения**: Генерация HTML страниц
2. **Email-рассылки**: Шаблоны писем
3. **Конфигурационные файлы**: Генерация config'ов для разных окружений
4. **Отчёты**: PDF, CSV, текстовые отчёты
5. **Кодогенерация**: Создание кода на основе шаблонов

## Часть 9: Рекомендации по использованию

### Лучшие практики:

1. **Разделение ответственности**: Шаблоны - только для представления, бизнес-логика - в коде
2. **Предварительная компиляция**: Парси шаблоны один раз при старте программы
3. **Экранирование данных**: Всегда используй `html/template` для HTML
4. **Тестирование**: Тестируй шаблоны с разными данными

### Распространённые ошибки:

```go
// ОШИБКА: забыли точку
tmpl.Parse("{{Name}}")  // неправильно!
tmpl.Parse("{{.Name}}") // правильно!

// ОШИБКА: несоответствие типов
data := User{Name: "John"}
tmpl.Parse("{{.Name}} {{.Age}}")  // Age не существует в User!

// ОШИБКА: неправильный ключ в map
data := map[string]interface{}{"name": "John"}
tmpl.Parse("{{.Name}}")  // должно быть "name" с маленькой буквы!
```

## Часть 10: Расширенные возможности

Автор упоминает, но не раскрывает:

1. **Вложенные шаблоны**: Один шаблон может включать другой

```go
{{template "header" .}}
{{.Content}}
{{template "footer" .}}
```

2. **Пользовательские функции**: Можно добавлять свои функции в шаблоны

```go
funcMap := template.FuncMap{
    "uppercase": strings.ToUpper,
}
tmpl := template.New("test").Funcs(funcMap)
tmpl.Parse("{{.Name | uppercase}}")
```

3. **Изменение разделителей**: Можно использовать другие символы вместо `{{}}`

```go
tmpl := template.New("test").Delims("[[", "]]")
tmpl.Parse("[[.Name]]")
```

## Заключение

Текстовые шаблоны в Go - это мощный инструмент, который:

- **Прост в использовании**: базовый синтаксис легко понять
- **Гибкий**: поддерживает условия, циклы, функции
- **Безопасный**: особенно `html/template`
- **Эффективный**: быстрая работа даже с большими шаблонами

**Ключевые моменты для запоминания:**

1. Шаблоны = текст + действия в `{{}}`
2. Данные передаются в `Execute()`
3. Используй `html/template` для веба
4. Ключи в данных и шаблоне должны совпадать
5. `template.Must` упрощает обработку ошибок при парсинге

# Мой Вопрос

```
Давай заново, но также показывай что выводит тот или иной код
```

## Часть 1: Самый простой пример

### Код:

```go
package main

import (
    "html/template"
    "os"
)

func main() {
    // Создаём шаблон с именем "example"
    tmpl := template.New("example")

    // Парсим строку с шаблоном
    tmpl, err := tmpl.Parse("Welcome {{.Name}}! How are you doing?\n")
    if err != nil {
        panic(err)
    }

    // Подготавливаем данные
    data := map[string]interface{}{
        "Name": "John",
    }

    // Выполняем шаблон с данными
    err = tmpl.Execute(os.Stdout, data)
    if err != nil {
        panic(err)
    }
}
```

### Выполнение и вывод:

```
$ go run main.go
Welcome John! How are you doing?
```

**Что произошло:**

1. `{{.Name}}` в шаблоне был заменён на значение `"John"` из данных
2. `\n` в конце строки добавил перенос строки

## Часть 2: Упрощение с template.Must

### Код:

```go
package main

import (
    "html/template"
    "os"
)

func main() {
    // Создаём и парсим шаблон одной строкой
    tmpl := template.Must(template.New("example").Parse("Welcome {{.Name}}! How are you doing?\n"))

    data := map[string]interface{}{
        "Name": "Alice",
    }

    tmpl.Execute(os.Stdout, data)
}
```

### Вывод:

```
$ go run main.go
Welcome Alice! How are you doing?
```

**Что изменилось:**

- Одна строка вместо четырёх
- `template.Must` автоматически вызовет панику при ошибке парсинга

## Часть 3: Ошибка при несовпадении ключей

### Код:

```go
package main

import (
    "html/template"
    "os"
)

func main() {
    tmpl := template.Must(template.New("example").Parse("Welcome {{.Name}}! How are you doing?\n"))

    // ОШИБКА: ключ "UserName" не совпадает с "Name" в шаблоне
    data := map[string]interface{}{
        "UserName": "Bob",  // Должно быть "Name", а не "UserName"
    }

    tmpl.Execute(os.Stdout, data)
}
```

### Вывод:

```
$ go run main.go
Welcome ! How are you doing?
```

**Что произошло:**

- Шаблон ищет `{{.Name}}`, но в данных есть только `"UserName"`
- Go не выдаёт ошибку, просто оставляет пустое место

## Часть 4: Использование структуры вместо map

### Код:

```go
package main

import (
    "html/template"
    "os"
)

type Person struct {
    Name string
    Age  int
}

func main() {
    tmpl := template.Must(template.New("example").Parse("{{.Name}} is {{.Age}} years old.\n"))

    data := Person{
        Name: "Charlie",
        Age:  30,
    }

    tmpl.Execute(os.Stdout, data)
}
```

### Вывод:

```
$ go run main.go
Charlie is 30 years old.
```

**Что изменилось:**

- Вместо `map` используем структуру `Person`
- Обращение к полям структуры: `{{.Name}}` и `{{.Age}}`

## Часть 5: Условный оператор в шаблоне

### Код:

```go
package main

import (
    "html/template"
    "os"
)

func main() {
    tmplStr := `{{.Name}} is {{.Age}} years old.
{{if .IsStudent}}They are a student.{{else}}They are not a student.{{end}}
`

    tmpl := template.Must(template.New("example").Parse(tmplStr))

    data := map[string]interface{}{
        "Name":      "Diana",
        "Age":       22,
        "IsStudent": true,
    }

    tmpl.Execute(os.Stdout, data)

    fmt.Println("\n--- Другой человек ---")

    data2 := map[string]interface{}{
        "Name":      "Eve",
        "Age":       35,
        "IsStudent": false,
    }

    tmpl.Execute(os.Stdout, data2)
}
```

### Вывод:

```
$ go run main.go
Diana is 22 years old.
They are a student.

--- Другой человек ---
Eve is 35 years old.
They are not a student.
```

**Что нового:**

- `{{if .IsStudent}}` проверяет булево значение
- `{{else}}` позволяет задать альтернативный текст
- `{{end}}` закрывает блок условия

## Часть 6: Цикл range для списка

### Код:

```go
package main

import (
    "html/template"
    "os"
)

func main() {
    tmplStr := `Students in class:
{{range .}}
- {{.Name}} (Age: {{.Age}})
{{end}}`

    tmpl := template.Must(template.New("example").Parse(tmplStr))

    students := []struct {
        Name string
        Age  int
    }{
        {"Alice", 20},
        {"Bob", 21},
        {"Charlie", 22},
    }

    tmpl.Execute(os.Stdout, students)
}
```

### Вывод:

```
$ go run main.go
Students in class:
- Alice (Age: 20)
- Bob (Age: 21)
- Charlie (Age: 22)
```

**Что нового:**

- `{{range .}}` проходит по всем элементам среза
- Внутри цикла `.` указывает на текущий элемент
- `{{end}}` закрывает блок цикла

## Часть 7: Полное консольное приложение из видео

### Код:

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
    "text/template"
)

func main() {
    // Создаём reader для чтения ввода пользователя
    reader := bufio.NewReader(os.Stdin)

    fmt.Print("Enter your name: ")
    name, _ := reader.ReadString('\n')
    name = strings.TrimSpace(name)

    // Определяем шаблоны
    templates := map[string]string{
        "welcome":      "Welcome {{.NM}}! We are glad you joined.\n",
        "notification": "{{.NM}}, you have a new notification: {{.NTF}}\n",
        "error":        "Oops! An error occurred: {{.EM}}\n",
    }

    // Парсим и сохраняем шаблоны
    parsedTemplates := make(map[string]*template.Template)
    for name, tmplStr := range templates {
        parsedTemplates[name] = template.Must(template.New(name).Parse(tmplStr))
    }

    // Главный цикл приложения
    for {
        fmt.Println("\nMenu:")
        fmt.Println("1. Join")
        fmt.Println("2. Get Notification")
        fmt.Println("3. Get Error")
        fmt.Println("4. Exit")

        fmt.Print("Choose an option: ")
        choice, _ := reader.ReadString('\n')
        choice = strings.TrimSpace(choice)

        switch choice {
        case "1":
            // Welcome шаблон
            data := map[string]interface{}{"NM": name}
            parsedTemplates["welcome"].Execute(os.Stdout, data)

        case "2":
            // Notification шаблон
            fmt.Print("Enter notification message: ")
            notification, _ := reader.ReadString('\n')
            notification = strings.TrimSpace(notification)

            data := map[string]interface{}{
                "NM":  name,
                "NTF": notification,
            }
            parsedTemplates["notification"].Execute(os.Stdout, data)

        case "3":
            // Error шаблон
            fmt.Print("Enter error message: ")
            errorMsg, _ := reader.ReadString('\n')
            errorMsg = strings.TrimSpace(errorMsg)

            data := map[string]interface{}{
                "NM": name,
                "EM": errorMsg,
            }
            parsedTemplates["error"].Execute(os.Stdout, data)

        case "4":
            fmt.Println("Goodbye!")
            return

        default:
            fmt.Println("Invalid choice. Please try again.")
        }
    }
}
```

### Пример работы программы:

```
$ go run main.go
Enter your name: John

Menu:
1. Join
2. Get Notification
3. Get Error
4. Exit
Choose an option: 1
Welcome John! We are glad you joined.

Menu:
1. Join
2. Get Notification
3. Get Error
4. Exit
Choose an option: 2
Enter notification message: Your order has shipped
John, you have a new notification: Your order has shipped

Menu:
1. Join
2. Get Notification
3. Get Error
4. Exit
Choose an option: 3
Enter error message: Database connection failed
Oops! An error occurred: Database connection failed

Menu:
1. Join
2. Get Notification
3. Get Error
4. Exit
Choose an option: 5
Invalid choice. Please try again.

Menu:
1. Join
2. Get Notification
3. Get Error
4. Exit
Choose an option: 4
Goodbye!
```

## Часть 8: Проблема с несовпадающими ключами

Давай покажем, что происходит когда ключи не совпадают:

### Код (фрагмент с ошибкой):

```go
case "3":
    fmt.Print("Enter error message: ")
    errorMsg, _ := reader.ReadString('\n')
    errorMsg = strings.TrimSpace(errorMsg)

    // ОШИБКА: используем "ErrorMsg" вместо "EM"
    data := map[string]interface{}{
        "NM":        name,
        "ErrorMsg": errorMsg,  // Должно быть "EM"
    }
    parsedTemplates["error"].Execute(os.Stdout, data)
```

### Вывод при выборе опции 3:

```
Oops! An error occurred:
```

**Что произошло:**

- Шаблон ожидает `{{.EM}}`, но мы передали `ErrorMsg`
- Значение не подставляется, но ошибки нет
- Выводится пустая строка вместо сообщения об ошибке

## Часть 9: Использование html/template для безопасности

### Код:

```go
package main

import (
    "html/template"
    "os"
)

func main() {
    // Опасный пользовательский ввод
    userInput := "<script>alert('XSS attack!')</script>"

    // 1. text/template - НЕБЕЗОПАСНО
    tmpl1 := template.Must(template.New("unsafe").Parse("User input: {{.}}\n"))
    fmt.Print("text/template: ")
    tmpl1.Execute(os.Stdout, userInput)

    // 2. html/template - БЕЗОПАСНО
    tmpl2 := template.Must(template.New("safe").Parse("User input: {{.}}\n"))
    fmt.Print("html/template: ")
    tmpl2.Execute(os.Stdout, userInput)
}
```

### Вывод:

```
$ go run main.go
text/template: User input: <script>alert('XSS attack!')</script>
html/template: User input: &lt;script&gt;alert(&#39;XSS attack!&#39;)&lt;/script&gt;
```

**Что произошло:**

- `text/template` вывел HTML как есть (опасно для веб-страниц!)
- `html/template` автоматически экранировал специальные символы:
  - `<` → `&lt;`
  - `>` → `&gt;`
  - `'` → `&#39;`
  - Это защищает от XSS-атак

## Часть 10: Пользовательские функции в шаблонах

### Код:

```go
package main

import (
    "html/template"
    "os"
    "strings"
)

func main() {
    // Определяем функции для шаблона
    funcMap := template.FuncMap{
        "uppercase": strings.ToUpper,
        "add": func(a, b int) int {
            return a + b
        },
    }

    // Создаём шаблон с функциями
    tmplStr := `Name: {{.Name | uppercase}}
Age next year: {{add .Age 1}}`

    tmpl := template.Must(template.New("example").Funcs(funcMap).Parse(tmplStr))

    data := map[string]interface{}{
        "Name": "john smith",
        "Age":  25,
    }

    tmpl.Execute(os.Stdout, data)
}
```

### Вывод:

```
$ go run main.go
Name: JOHN SMITH
Age next year: 26
```

**Что нового:**

- `| uppercase` - конвейер, применяет функцию к значению
- `{{add .Age 1}}` - вызов функции с двумя аргументами
- `Funcs(funcMap)` - регистрация функций в шаблоне

## Итоговая таблица основных действий:

| Действие       | Пример                                        | Описание                         |
| -------------- | --------------------------------------------- | -------------------------------- | ------------------ |
| Переменная     | `{{.Name}}`                                   | Вставка значения поля            |
| Условие        | `{{if .Active}}Active{{else}}Inactive{{end}}` | Условный вывод                   |
| Цикл           | `{{range .Items}}{{.}}{{end}}`                | Итерация по коллекции            |
| Конвейер       | `{{.Name                                      | uppercase}}`                     | Применение функции |
| С определением | `{{define "header"}}...{{end}}`               | Определение именованного шаблона |
| Вызов шаблона  | `{{template "header" .}}`                     | Вставка именованного шаблона     |

**Ключевые выводы:**

1. Шаблоны позволяют отделить данные от представления
2. Всегда проверяй соответствие ключей в данных и шаблоне
3. Для веб-приложений используй `html/template` для безопасности
4. Шаблоны можно создавать динамически и переиспользовать
5. Ошибки парсинга лучше обрабатывать через `template.Must`
