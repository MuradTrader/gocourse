Отлично! Давайте разберем пакет `fmt` максимально подробно с примерами кода и вывода. Я объясню каждую функцию и покажу, что происходит под капотом.

## **ЧАСТЬ 1: ФУНКЦИИ ВЫВОДА (Printing Functions)**

### Код 1: `Print`, `Println`, `Printf` - в чем разница?

```go
package main

import "fmt"

func main() {
    fmt.Println("=== РАЗНИЦА МЕЖДУ Print, Println и Printf ===")

    // 1. Print - выводит без перевода строки
    fmt.Print("Hello ")
    fmt.Print("World")
    fmt.Print(" 123456")
    // Вывод пока: Hello World 123456

    // 2. Println - выводит с переводом строки И пробелами между аргументами
    fmt.Println()  // Переходим на новую строку для наглядности
    fmt.Println("Hello", "World", 123456)
    // Вывод: Hello World 123456

    // 3. Printf - форматированный вывод
    name := "John"
    age := 25
    fmt.Printf("Имя: %s, Возраст: %d\n", name, age)
    // Вывод: Имя: John, Возраст: 25

    // Дополнительные примеры с Printf
    fmt.Printf("Бинарное представление %d: %b\n", age, age)
    // Вывод: Бинарное представление 25: 11001

    fmt.Printf("Шестнадцатеричное представление %d: %X\n", age, age)
    // Вывод: Шестнадцатеричное представление 25: 19

    // Смешанный вывод
    fmt.Print("\nСмешанный вывод: ")
    fmt.Println("Имя:", name, "Возраст:", age)
    fmt.Printf("Форматированный: Имя:%s Возраст:%d\n", name, age)
}
```

**Вывод программы:**

```
=== РАЗНИЦА МЕЖДУ Print, Println и Printf ===
Hello World 123456
Hello World 123456
Имя: John, Возраст: 25
Бинарное представление 25: 11001
Шестнадцатеричное представление 25: 19

Смешанный вывод: Имя: John Возраст: 25
Форматированный: Имя:John Возраст:25
```

**Что происходит под капотом:**

1. `Print`:

   - Выводит аргументы как есть
   - **НЕ** добавляет пробелы между аргументами
   - **НЕ** добавляет перевод строки
   - Внутри вызывает `Fprint(os.Stdout, args...)`

2. `Println`:

   - Добавляет пробел между аргументами
   - Добавляет перевод строки (`\n`) в конце
   - Внутри вызывает `Fprintln(os.Stdout, args...)`

3. `Printf`:
   - Форматирует строку согласно шаблону
   - **НЕ** добавляет перевод строки автоматически
   - Внутри вызывает `Fprintf(os.Stdout, format, args...)`

## **ЧАСТЬ 2: ФУНКЦИИ ФОРМАТИРОВАНИЯ (Formatting Functions)**

### Код 2: `Sprint`, `Sprintln`, `Sprintf` - возвращают строки

```go
package main

import "fmt"

func main() {
    fmt.Println("=== Sprint, Sprintln и Sprintf ===")

    // 1. Sprint - возвращает строку без пробелов и перевода строки
    s1 := fmt.Sprint("Hello", "World", 123456)
    fmt.Print("Sprint: '", s1, "'")  // Вывод: Sprint: 'HelloWorld123456'
    fmt.Println(" ← нет пробелов между аргументами")

    // 2. Sprintln - возвращает строку с пробелами и переводом строки
    s2 := fmt.Sprintln("Hello", "World", 123456)
    fmt.Print("Sprintln: '", s2, "'")  // Вывод будет включать \n
    fmt.Println("← здесь был символ новой строки")

    // Проверим длину строки с \n
    fmt.Printf("Длина строки из Sprintln: %d символов\n", len(s2))
    // Длина на 1 больше из-за \n

    // 3. Sprintf - возвращает форматированную строку
    name := "Alice"
    age := 30
    s3 := fmt.Sprintf("Имя: %s, Возраст: %d", name, age)
    fmt.Println("Sprintf:", s3)  // Вывод: Имя: Alice, Возраст: 30

    // Практическое использование - сборка сложных строк
    userInfo := fmt.Sprintf(
        "Пользователь: %s\nВозраст: %d\nАктивен: %v",
        name, age, true,
    )
    fmt.Println("\nСобранная строка:")
    fmt.Println(userInfo)

    // Сравнение с конкатенацией
    concatStr := "Имя: " + name + ", Возраст: " + fmt.Sprint(age)
    fmt.Printf("\nКонкатенация: %s\n", concatStr)
    fmt.Printf("Sprintf: %s\n", s3)
}
```

**Вывод программы:**

```
=== Sprint, Sprintln и Sprintf ===
Sprint: 'HelloWorld123456' ← нет пробелов между аргументами
Sprintln: 'Hello World 123456
'← здесь был символ новой строки
Длина строки из Sprintln: 19 символов
Sprintf: Имя: Alice, Возраст: 30

Собранная строка:
Пользователь: Alice
Возраст: 30
Активен: true

Конкатенация: Имя: Alice, Возраст: 30
Sprintf: Имя: Alice, Возраст: 30
```

**Что происходит под капотом:**

1. `Sprint`:

   - Конвертирует аргументы в строки
   - Складывает их вместе без пробелов
   - Возвращает `string`

2. `Sprintln`:

   - Добавляет пробелы между аргументами
   - Добавляет `\n` в конце
   - Возвращает `string`

3. `Sprintf`:
   - Анализирует строку формата
   - Заменяет `%`-спецификаторы значениями
   - Возвращает отформатированную `string`

## **ЧАСТЬ 3: ФУНКЦИИ ВВОДА (Scanning Functions)**

### Код 3: `Scan`, `Scanln`, `Scanf` - чтение ввода

```go
package main

import "fmt"

func main() {
    fmt.Println("=== Scan, Scanln и Scanf ===")

    // 1. Scan - читает до пробела/перевода строки
    fmt.Println("\n1. Использование Scan (введите два значения):")
    var name string
    var age int

    fmt.Print("Введите имя и возраст через пробел: ")
    // Scan читает пока не получит все аргументы
    count, err := fmt.Scan(&name, &age)
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    fmt.Printf("Прочитано %d значений: Имя=%s, Возраст=%d\n", count, name, age)

    // 2. Scanln - читает одну строку
    fmt.Println("\n2. Использование Scanln (введите в одну строку):")
    var city string
    var zip int

    fmt.Print("Введите город и индекс: ")
    // Scanln читает до перевода строки
    count2, err2 := fmt.Scanln(&city, &zip)
    if err2 != nil {
        fmt.Println("Ошибка:", err2)
        // Если ввести только одно значение, будет ошибка "unexpected newline"
        return
    }
    fmt.Printf("Прочитано %d значений: Город=%s, Индекс=%d\n", count2, city, zip)

    // 3. Scanf - читает с форматированием
    fmt.Println("\n3. Использование Scanf (формат: Имя:Возраст):")
    var name2 string
    var age2 int

    fmt.Print("Введите в формате 'Имя:Возраст': ")
    // Точное соответствие формату
    count3, err3 := fmt.Scanf("%s:%d", &name2, &age2)
    if err3 != nil {
        fmt.Println("Ошибка:", err3)
        return
    }
    fmt.Printf("Прочитано %d значений: Имя=%s, Возраст=%d\n", count3, name2, age2)

    // 4. Чтение нескольких строк
    fmt.Println("\n4. Чтение нескольких строк с Scan:")
    fmt.Println("Введите 3 числа (можно в разных строках):")
    var a, b, c int
    fmt.Scan(&a, &b, &c)
    fmt.Printf("Вы ввели: %d, %d, %d\n", a, b, c)
}
```

**Пример вывода программы:**

```
=== Scan, Scanln и Scanf ===

1. Использование Scan (введите два значения):
Введите имя и возраст через пробел: John 25
Прочитано 2 значений: Имя=John, Возраст=25

2. Использование Scanln (введите в одну строку):
Введите город и индекс: Moscow 123456
Прочитано 2 значений: Город=Moscow, Индекс=123456

3. Использование Scanf (формат: Имя:Возраст):
Введите в формате 'Имя:Возраст': Alice:30
Прочитано 2 значений: Имя=Alice, Возраст=30

4. Чтение нескольких строк с Scan:
Введите 3 числа (можно в разных строках):
10
20
30
Вы ввели: 10, 20, 30
```

**Что происходит под капотом:**

1. `Scan`:

   - Читает из `os.Stdin` (стандартный ввод)
   - Разделяет ввод по пробелам
   - Ждет, пока не получит все необходимые значения
   - Может читать из нескольких строк

2. `Scanln`:

   - Читает только одну строку
   - Останавливается при `\n`
   - Если значений не хватает - ошибка

3. `Scanf`:
   - Проверяет точное соответствие формату
   - Чувствителен к пробелам и разделителям
   - Возвращает количество успешно прочитанных значений

**ВАЖНО про указатели (`&`):**

```go
var x int
fmt.Scan(&x)  // & - взять адрес переменной

// Без & не сработает, потому что:
// 1. Функция получает КОПИЮ значения
// 2. Не может изменить оригинальную переменную
// 3. С & передается АДРЕС, по которому можно записать значение
```

## **ЧАСТЬ 4: ФОРМАТИРОВАНИЕ ОШИБОК (Errorf)**

### Код 4: Создание ошибок с `Errorf`

```go
package main

import (
    "fmt"
)

// Функция проверки возраста
func checkAge(age int) error {
    if age < 18 {
        // Errorf создает ошибку с форматированным сообщением
        return fmt.Errorf("возраст %d слишком мал для вождения", age)
    }
    if age > 100 {
        return fmt.Errorf("возраст %d слишком велик для вождения", age)
    }
    return nil  // nil означает "нет ошибки"
}

// Функция деления с проверкой
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("деление на ноль: %.2f / %.2f", a, b)
    }
    return a / b, nil
}

// Функция с возвратом нескольких значений
func getUser(id int) (name string, age int, err error) {
    if id < 0 {
        err = fmt.Errorf("неверный ID пользователя: %d", id)
        return
    }
    // Имитация получения данных
    name = "John"
    age = 25
    return
}

func main() {
    fmt.Println("=== Использование Errorf ===")

    // 1. Простая проверка
    age := 15
    if err := checkAge(age); err != nil {
        fmt.Printf("Ошибка: %v\n", err)
        // Вывод: Ошибка: возраст 15 слишком мал для вождения
    } else {
        fmt.Println("Возраст подходит")
    }

    // 2. Проверка деления
    result, err := divide(10, 0)
    if err != nil {
        fmt.Printf("Ошибка деления: %v\n", err)
        // Вывод: Ошибка деления: деление на ноль: 10.00 / 0.00
    } else {
        fmt.Printf("Результат: %.2f\n", result)
    }

    // 3. Получение пользователя
    name, age2, err := getUser(-1)
    if err != nil {
        fmt.Printf("Ошибка получения пользователя: %v\n", err)
        // Вывод: Ошибка получения пользователя: неверный ID пользователя: -1
    } else {
        fmt.Printf("Пользователь: %s, %d лет\n", name, age2)
    }

    // 4. Цепочка ошибок (error wrapping)
    if err := checkAge(200); err != nil {
        // Добавляем контекст к ошибке
        wrappedErr := fmt.Errorf("проверка возраста не пройдена: %w", err)
        fmt.Printf("Обернутая ошибка: %v\n", wrappedErr)
        // Вывод: Обернутая ошибка: проверка возраста не пройдена: возраст 200 слишком велик для вождения

        // Можно получить оригинальную ошибку
        fmt.Printf("Оригинальная ошибка: %v\n", err)
    }
}
```

**Вывод программы:**

```
=== Использование Errorf ===
Ошибка: возраст 15 слишком мал для вождения
Ошибка деления: деление на ноль: 10.00 / 0.00
Ошибка получения пользователя: неверный ID пользователя: -1
Обернутая ошибка: проверка возраста не пройдена: возраст 200 слишком велик для вождения
Оригинальная ошибка: возраст 200 слишком велик для вождения
```

**Что происходит под капотом:**

1. `Errorf` создает структуру, реализующую интерфейс `error`
2. Внутри используется `Sprintf` для форматирования строки
3. Возвращается значение типа `error`, которое можно:
   - Проверить на `nil`
   - Вывести через `%v` или `%s`
   - Обернуть с `%w` (в новых версиях Go)

## **ЧАСТЬ 5: КАК ЭТО РАБОТАЕТ ПОД КАПОТОМ**

### Код 5: Что происходит внутри fmt

```go
package main

import (
    "fmt"
    "io"
    "os"
    "strings"
)

// Упрощенная реализация собственной "Print" функции
func myPrint(args ...interface{}) {
    for _, arg := range args {
        // Преобразуем каждый аргумент в строку
        switch v := arg.(type) {
        case string:
            os.Stdout.WriteString(v)
        case int:
            os.Stdout.WriteString(fmt.Sprint(v))
        default:
            // Для других типов используем fmt.Sprint
            os.Stdout.WriteString(fmt.Sprint(v))
        }
    }
}

// Упрощенная реализация собственного "Scan"
func myScan(a *int) {
    var input string
    fmt.Scanln(&input)
    // Упрощенное преобразование строки в int
    // В реальности тут много проверок ошибок
    fmt.Sscan(input, a)
}

func main() {
    fmt.Println("=== КАК РАБОТАЕТ FMT ПОД КАПОТОМ ===")

    // 1. Что такое os.Stdout и os.Stdin?
    fmt.Println("\n1. Стандартные потоки:")
    fmt.Printf("os.Stdout тип: %T\n", os.Stdout)  // *os.File
    fmt.Printf("os.Stdin тип: %T\n", os.Stdin)    // *os.File

    // 2. Fprint - общая функция для любого io.Writer
    fmt.Println("\n2. Использование Fprint:")
    fmt.Fprint(os.Stdout, "Hello via Fprint\n")

    // Можно писать в строку
    var builder strings.Builder
    fmt.Fprint(&builder, "Записали в builder")
    fmt.Printf("Builder содержит: %s\n", builder.String())

    // 3. Собственная реализация (упрощенная)
    fmt.Println("\n3. Собственная функция myPrint:")
    myPrint("Hello ", 42, " world\n")

    // 4. Как работает сканирование
    fmt.Println("\n4. Сканирование числа:")
    fmt.Print("Введите число: ")
    var num int
    myScan(&num)
    fmt.Printf("Вы ввели: %d\n", num)

    // 5. Форматирование в память
    fmt.Println("\n5. Форматирование в разные цели:")

    // В файл
    file, _ := os.Create("test.txt")
    fmt.Fprintf(file, "Запись в файл: %d\n", 123)
    file.Close()

    // В строку
    var result strings.Builder
    fmt.Fprintf(&result, "Имя: %s, Возраст: %d", "John", 25)
    fmt.Printf("Результат в строке: %s\n", result.String())
}
```

**Вывод программы:**

```
=== КАК РАБОТАЕТ FMT ПОД КАПОТОМ ===

1. Стандартные потоки:
os.Stdout тип: *os.File
os.Stdin тип: *os.File

2. Использование Fprint:
Hello via Fprint
Builder содержит: Записали в builder

3. Собственная функция myPrint:
Hello 42 world

4. Сканирование числа:
Введите число: 42
Вы ввели: 42

5. Форматирование в разные цели:
Результат в строке: Имя: John, Возраст: 25
```

**Внутреннее устройство:**

1. **Все функции `Print` используют `Fprint`**:

   - `Print` = `Fprint(os.Stdout, ...)`
   - `Println` = `Fprintln(os.Stdout, ...)`
   - `Printf` = `Fprintf(os.Stdout, ...)`

2. **`Fprint` работает с интерфейсом `io.Writer`**:

   ```go
   type Writer interface {
       Write([]byte) (int, error)
   }
   ```

   - `os.Stdout` реализует `Writer` (пишет в консоль)
   - `os.File` реализует `Writer` (пишет в файл)
   - `strings.Builder` реализует `Writer` (пишет в строку)

3. **Сканирование использует `io.Reader`**:
   - `Scan` = `Fscan(os.Stdin, ...)`
   - Аналогично для `Scanln` и `Scanf`

## **ЧАСТЬ 6: ПРАКТИЧЕСКИЕ ПРИМЕРЫ**

### Код 6: Реальные кейсы использования

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Println("=== ПРАКТИЧЕСКИЕ ПРИМЕРЫ ===")

    // 1. Логирование с форматированием
    fmt.Println("\n1. Логирование:")
    user := "Alice"
    action := "login"
    timestamp := "2024-01-15 10:30:00"

    logMessage := fmt.Sprintf("[%s] Пользователь %s: %s",
        timestamp, user, action)
    fmt.Println(logMessage)

    // 2. Генерация SQL запроса (осторожно с SQL-инъекциями!)
    fmt.Println("\n2. Генерация SQL (упрощенно):")
    table := "users"
    id := 42
    query := fmt.Sprintf("SELECT * FROM %s WHERE id = %d", table, id)
    fmt.Println("SQL:", query)

    // 3. Форматирование денежных значений
    fmt.Println("\n3. Денежные значения:")
    amounts := []float64{1234.56, 789.01, 45678.90}
    for _, amount := range amounts {
        fmt.Printf("$%10.2f\n", amount)  // 10 символов, 2 после точки
    }

    // 4. Вывод таблицы
    fmt.Println("\n4. Таблица пользователей:")
    fmt.Println(strings.Repeat("-", 40))
    fmt.Printf("%-20s %-10s %-10s\n", "Имя", "Возраст", "Баланс")
    fmt.Println(strings.Repeat("-", 40))

    users := []struct{
        name string
        age int
        balance float64
    }{
        {"Алиса", 25, 1234.56},
        {"Боб", 30, 5678.90},
        {"Чарли", 35, 9876.54},
    }

    for _, u := range users {
        fmt.Printf("%-20s %-10d $%-9.2f\n",
            u.name, u.age, u.balance)
    }
    fmt.Println(strings.Repeat("-", 40))

    // 5. Чтение конфигурации
    fmt.Println("\n5. Чтение конфигурации:")
    fmt.Print("Введите путь к файлу: ")
    var path string
    fmt.Scanln(&path)

    fmt.Print("Введите количество попыток: ")
    var attempts int
    fmt.Scanln(&attempts)

    config := fmt.Sprintf(
        "Конфигурация:\n  Путь: %s\n  Попытки: %d\n",
        path, attempts,
    )
    fmt.Println(config)

    // 6. Обработка ошибок в реальном сценарии
    fmt.Println("\n6. Обработка ошибок:")

    processData := func(data string) error {
        if len(data) < 3 {
            return fmt.Errorf("данные слишком короткие: '%s' (нужно минимум 3 символа)", data)
        }
        // Обработка данных...
        return nil
    }

    testData := []string{"ok", "good", "no"}
    for _, data := range testData {
        if err := processData(data); err != nil {
            fmt.Printf("Ошибка обработки: %v\n", err)
        } else {
            fmt.Printf("Данные '%s' обработаны успешно\n", data)
        }
    }
}
```

**Вывод программы:**

```
=== ПРАКТИЧЕСКИЕ ПРИМЕРЫ ===

1. Логирование:
[2024-01-15 10:30:00] Пользователь Alice: login

2. Генерация SQL (упрощенно):
SQL: SELECT * FROM users WHERE id = 42

3. Денежные значения:
$   1234.56
$    789.01
$  45678.90

4. Таблица пользователей:
----------------------------------------
Имя                  Возраст    Баланс
----------------------------------------
Алиса                25         $1234.56
Боб                  30         $5678.90
Чарли                35         $9876.54
----------------------------------------

5. Чтение конфигурации:
Введите путь к файлу: /home/user/data.txt
Введите количество попыток: 3
Конфигурация:
  Путь: /home/user/data.txt
  Попытки: 3

6. Обработка ошибок:
Ошибка обработки: данные слишком короткие: 'ok' (нужно минимум 3 символа)
Данные 'good' обработаны успешно
Ошибка обработки: данные слишком короткие: 'no' (нужно минимум 3 символа)
```

## **КРАТКАЯ ШПАРГАЛКА:**

```
ФУНКЦИИ ВЫВОДА:
  Print()    - без пробелов, без \n
  Println()  - с пробелами, с \n
  Printf()   - форматирование, без \n

ФУНКЦИИ ВОЗВРАТА СТРОК:
  Sprint()    - как Print, но возвращает string
  Sprintln()  - как Println, но возвращает string
  Sprintf()   - как Printf, но возвращает string

ФУНКЦИИ ВВОДА:
  Scan()    - читает до пробела, может из нескольких строк
  Scanln()  - читает одну строку, требует все значения
  Scanf()   - форматированный ввод, точное соответствие

ФОРМАТИРОВАНИЕ ОШИБОК:
  Errorf() - создает ошибку с форматированным сообщением

ОБЩИЕ ФУНКЦИИ (для любого io.Writer/Reader):
  Fprint()    / Fscan()
  Fprintln()  / Fscanln()
  Fprintf()   / Fscanf()
```

**Ключевые моменты:**

1. Все функции `Print` в итоге вызывают `Fprint` с `os.Stdout`
2. Все функции `Scan` в итоге вызывают `Fscan` с `os.Stdin`
3. `Errorf` использует `Sprintf` внутри
4. Для сканирования **всегда** передавайте указатели (`&variable`)
5. Используйте `Printf` для сложного форматирования, `Println` для простого вывода

Пакет `fmt` - это основа ввода/вывода в Go. Понимание его работы критически важно для любого Go-разработчика.
