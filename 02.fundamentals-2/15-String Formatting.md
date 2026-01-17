Отлично! Я буду вашим профессиональным Go-разработчиком и максимально подробно объясню каждую концепцию из этого текста, раскрывая внутреннее устройство.

## Подробное объяснение текста курса:

### Часть 1: Введение в форматирование строк

```go
// String formatting in go refers to the techniques used
// to create formatted output from variables or constants.
```

**Что такое форматирование строк:**
Это процесс преобразования данных (чисел, строк, структур) в определенный текстовый формат. Например:

- Число 42 → строка "42"
- Число 42 → строка "0042" (с ведущими нулями)
- Строка "hello" → " hello" (с выравниванием)

### Часть 2: Основные механизмы Go

```go
// Go provides several mechanisms for formatting strings,
// including the FMT package.
```

**Пакет `fmt` - это основной инструмент для:**

1. **Вывода** на экран (`Print`, `Println`, `Printf`)
2. **Ввода** с клавиатуры (`Scan`, `Scanf`, `Scanln`)
3. **Форматирования** в строки (`Sprintf`, `Sprint`, `Sprintln`)

**Внутренняя структура fmt:**

```go
// Примерный вид (упрощенно):
type pp struct {
    buf buffer  // буфер для накопления результата
    // ... другие поля для управления форматированием
}
```

### Часть 3: Форматирование чисел с ведущими нулями

```go
num := 42
fmt.Printf("%05d\n", num)  // Выведет: 00042
```

**Давайте разберем `%05d` по частям:**

```
%   - начало спецификатора формата
0   - флаг: заполнить нулями
5   - минимальная ширина поля
d   - тип: целое число (decimal)
```

**Что происходит внутри `Printf`:**

1. Go видит спецификатор `%05d`
2. Преобразует число 42 в строку "42" (2 символа)
3. Проверяет минимальную ширину 5 символов
4. Так как "42" короче 5 символов, добавляет 3 ведущих нуля
5. Получает "00042"

**Более подробно:**

```go
// Псевдокод того, что происходит:
func formatNumber(num int, format string) string {
    str := strconv.Itoa(num)  // "42"
    width := 5
    if len(str) < width {
        padding := width - len(str)  // 3
        return strings.Repeat("0", padding) + str  // "000" + "42"
    }
    return str
}
```

### Часть 4: Выравнивание строк

```go
message := "Hello"
fmt.Printf("|%10s|\n", message)   // |     Hello| (по правому краю)
fmt.Printf("|%-10s|\n", message)  // |Hello     | (по левому краю)
```

**Разбор `%10s` и `%-10s`:**

- `%10s` - строка шириной 10 символов, выравнивание по правому краю
- `%-10s` - строка шириной 10 символов, выравнивание по левому краю (минус указывает на левое выравнивание)

**Почему используются символы `|`:**

```go
// Без символов '|' трудно увидеть пробелы:
fmt.Printf("%10s\n", "Hello")   // Выведет: "     Hello"
// Где пробелы? Непонятно!

// С символами '|' видно четко:
fmt.Printf("|%10s|\n", "Hello") // Выведет: "|     Hello|"
// Теперь видно 5 пробелов слева
```

**Что происходит в памяти:**

```
message = "Hello" (5 байт)
Ширина 10 символов, значит нужно добавить 5 пробелов:

Правое выравнивание: "     Hello" (5 пробелов + "Hello")
Левое выравнивание:  "Hello     " ("Hello" + 5 пробелов)
```

### Часть 5: Raw String Literals (Сырые строковые литералы)

```go
// Двойные кавычки - интерпретируемые строки
message1 := "Hello\nWorld"
// Обратные кавычки - сырые строки
message2 := `Hello\nWorld`
```

**Разница в памяти:**

1. **Двойные кавычки:**

```go
message1 := "Hello\nWorld"
// В памяти: [72 101 108 108 111 10 87 111 114 108 100]
// 10 - это байт для символа новой строки (\n)
```

2. **Обратные кавычки (backticks):**

```go
message2 := `Hello\nWorld`
// В памяти: [72 101 108 108 111 92 110 87 111 114 108 100]
// 92 = '\' (обратный слеш)
// 110 = 'n' (буква n)
// Т.е. буквально "\n" как два символа
```

**Демонстрация:**

```go
package main

import "fmt"

func main() {
    // Пример 1: с двойными кавычками
    message1 := "Hello\nWorld"
    fmt.Println("С двойными кавычками:")
    fmt.Print(message1)
    fmt.Println("---")

    // Пример 2: с обратными кавычками
    message2 := `Hello\nWorld`
    fmt.Println("С обратными кавычками:")
    fmt.Print(message2)
    fmt.Println("---")

    // Пример 3: многострочная строка
    message3 := `Это
многострочная
строка`
    fmt.Println("Многострочная строка:")
    fmt.Print(message3)
}
```

**Вывод:**

```
С двойными кавычками:
Hello
World
---
С обратными кавычками:
Hello\nWorld---
Многострочная строка:
Это
многострочная
строка
```

### Часть 6: Практическое применение обратных кавычек

**1. Регулярные выражения:**

```go
// С двойными кавычками (неудобно):
re1 := regexp.MustCompile("\\d+")  // Два слеша!

// С обратными кавычками (удобно):
re2 := regexp.MustCompile(`\d+`)   // Один слеш
```

**Почему так?**

- В двойных кавычках `\` - escape-символ
- Чтобы получить один `\` в строке, нужно написать `\\`
- В обратных кавычках `\` - обычный символ

**2. SQL запросы:**

```go
// Плохо: нужно экранировать кавычки
query1 := "SELECT * FROM users WHERE name = \"John\""

// Хорошо: никакого экранирования
query2 := `SELECT * FROM users WHERE name = "John"`

// Еще лучше: многострочный запрос
query3 := `
SELECT
    id,
    name,
    email
FROM users
WHERE
    age > 30
    AND status = 'active'
`
```

**3. JSON/XML шаблоны:**

```go
jsonTemplate := `
{
    "name": "%s",
    "age": %d,
    "city": "%s"
}
`
formatted := fmt.Sprintf(jsonTemplate, "John", 30, "New York")
```

### Часть 7: Все спецификаторы формата на практике

```go
package main

import "fmt"

func main() {
    // 1. Целые числа
    fmt.Printf("Decimal: %d\n", 42)        // 42
    fmt.Printf("Binary: %b\n", 42)         // 101010
    fmt.Printf("Octal: %o\n", 42)          // 52
    fmt.Printf("Hex (lower): %x\n", 42)    // 2a
    fmt.Printf("Hex (upper): %X\n", 42)    // 2A

    // 2. Вещественные числа
    fmt.Printf("Float: %f\n", 3.14159)     // 3.141590
    fmt.Printf("Float (2 decimal): %.2f\n", 3.14159) // 3.14
    fmt.Printf("Scientific: %e\n", 3.14159) // 3.141590e+00

    // 3. Строки
    fmt.Printf("String: %s\n", "Hello")    // Hello
    fmt.Printf("Quoted: %q\n", "Hello")    // "Hello"

    // 4. Ширина и точность
    fmt.Printf("Width 10: %10d\n", 42)      // "        42"
    fmt.Printf("Width 10, left: %-10d|\n", 42) // "42        |"
    fmt.Printf("Width 10, zero: %010d\n", 42)  // "0000000042"

    // 5. Аргументы по индексу
    fmt.Printf("%[2]d %[1]d\n", 10, 20)    // 20 10

    // 6. Тип значения
    fmt.Printf("Type: %T\n", 42)           // int
    fmt.Printf("Type: %T\n", "Hello")      // string

    // 7. Значение в формате Go
    fmt.Printf("Go syntax: %#v\n", "Hello\tWorld") // "Hello\\tWorld"
}
```

### Часть 8: Sprintf - форматирование в строку (не вывод)

```go
package main

import "fmt"

func main() {
    name := "Alice"
    age := 30

    // Printf - выводит на экран
    fmt.Printf("Name: %s, Age: %d\n", name, age)

    // Sprintf - возвращает строку (не выводит)
    result := fmt.Sprintf("Name: %s, Age: %d", name, age)
    fmt.Println("Результат Sprintf:", result)

    // Пример использования
    filename := fmt.Sprintf("report_%s_%d.txt", name, age)
    fmt.Println("Имя файла:", filename)

    // Составное форматирование
    header := fmt.Sprintf("%-20s %10s %10s", "Name", "Age", "Salary")
    fmt.Println(header)
    fmt.Println(fmt.Sprintf("%-20s %10d %10.2f", "John Doe", 30, 50000.50))
}
```

### Часть 9: Расширенное форматирование с флагами

```go
package main

import "fmt"

func main() {
    // Флаг '+' - всегда показывать знак
    fmt.Printf("%+d\n", 42)   // +42
    fmt.Printf("%+d\n", -42)  // -42

    // Флаг ' ' (пробел) - добавлять пробел для положительных чисел
    fmt.Printf("% d\n", 42)   // " 42"
    fmt.Printf("% d\n", -42)  // "-42"

    // Флаг '#' - альтернативный формат
    fmt.Printf("%#x\n", 42)   // 0x2a
    fmt.Printf("%#o\n", 42)   // 052
    fmt.Printf("%#b\n", 42)   // 0b101010

    // Комбинация флагов
    fmt.Printf("%#010x\n", 42) // 0x0000002a
    fmt.Printf("%-10s %10.2f\n", "Item1", 123.456) // Item1      123.46
}
```

### Часть 10: Форматирование структур

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
    City string
}

func main() {
    p := Person{"John", 30, "New York"}

    // Простой вывод
    fmt.Printf("%v\n", p)        // {John 30 New York}

    // С именами полей
    fmt.Printf("%+v\n", p)       // {Name:John Age:30 City:New York}

    // В синтаксисе Go
    fmt.Printf("%#v\n", p)       // main.Person{Name:"John", Age:30, City:"New York"}

    // Тип структуры
    fmt.Printf("%T\n", p)        // main.Person
}
```

### Часть 11: Пользовательское форматирование (Stringer interface)

```go
package main

import "fmt"

type RGB struct {
    R, G, B int
}

// Реализуем интерфейс Stringer
func (c RGB) String() string {
    return fmt.Sprintf("RGB(%d, %d, %d)", c.R, c.G, c.B)
}

func main() {
    color := RGB{255, 0, 128}

    // Автоматически вызовется метод String()
    fmt.Println(color)           // RGB(255, 0, 128)
    fmt.Printf("%s\n", color)    // RGB(255, 0, 128)
    fmt.Printf("%v\n", color)    // RGB(255, 0, 128)
}
```

### Часть 12: Эффективное форматирование для производительности

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    // НЕЭФФЕКТИВНО: создает много промежуточных строк
    result := ""
    for i := 0; i < 10; i++ {
        result += fmt.Sprintf("Item %d, ", i)
    }

    // ЭФФЕКТИВНО: используем strings.Builder
    var builder strings.Builder
    for i := 0; i < 10; i++ {
        fmt.Fprintf(&builder, "Item %d, ", i)
    }
    result2 := builder.String()

    // Еще лучше: заранее выделить память
    builder3 := strings.Builder{}
    builder3.Grow(100) // Резервируем 100 байт заранее
    for i := 0; i < 10; i++ {
        fmt.Fprintf(&builder3, "Item %d, ", i)
    }
    result3 := builder3.String()

    fmt.Println("Результат 1:", result)
    fmt.Println("Результат 2:", result2)
    fmt.Println("Результат 3:", result3)
}
```

### Часть 13: Полный пример со всеми концепциями

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // 1. Базовое форматирование
    name := "Alice"
    age := 30
    salary := 50000.50

    fmt.Printf("1. Имя: %s, Возраст: %d, Зарплата: $%.2f\n", name, age, salary)

    // 2. Табличное форматирование
    fmt.Println("\n2. Таблица сотрудников:")
    fmt.Printf("%-20s %10s %15s\n", "Имя", "Возраст", "Зарплата")
    fmt.Println(strings.Repeat("-", 50))
    fmt.Printf("%-20s %10d %15.2f\n", "Alice Johnson", 30, 50000.50)
    fmt.Printf("%-20s %10d %15.2f\n", "Bob Smith", 25, 45000.00)
    fmt.Printf("%-20s %10d %15.2f\n", "Charlie Brown", 35, 60000.75)

    // 3. Форматирование с разными системами счисления
    fmt.Println("\n3. Число 42 в разных системах:")
    fmt.Printf("Десятичное: %d\n", 42)
    fmt.Printf("Двоичное: 0b%b\n", 42)
    fmt.Printf("Восьмеричное: 0%o\n", 42)
    fmt.Printf("Шестнадцатеричное: 0x%X\n", 42)

    // 4. Сырые строки для сложных данных
    fmt.Println("\n4. SQL запрос:")
    sqlQuery := `
SELECT
    u.id,
    u.name,
    u.email,
    COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id
ORDER BY order_count DESC
LIMIT 10;`
    fmt.Println(sqlQuery)

    // 5. Форматирование ошибок
    fmt.Println("\n5. Форматирование ошибок:")
    err := fmt.Errorf("ошибка доступа к файлу '%s': %v", "data.txt", os.ErrNotExist)
    fmt.Printf("Ошибка: %v\n", err)

    // 6. Пользовательские типы
    type Point struct{ X, Y int }
    p := Point{10, 20}
    fmt.Printf("\n6. Точка: %+v\n", p)

    // 7. Ширина и точность из аргументов
    fmt.Println("\n7. Динамическая ширина:")
    width := 15
    precision := 3
    value := 3.14159265
    fmt.Printf("%*.*f\n", width, precision, value)
}
```

## Ключевые выводы:

1. **`fmt` пакет** - основной инструмент форматирования
2. **Спецификаторы формата** (`%d`, `%s`, `%f`) определяют тип вывода
3. **Флаги** (`0`, `-`, `+`, `#`) меняют поведение форматирования
4. **Ширина и точность** контролируют размер вывода
5. **Обратные кавычки** для сырых строк (регулярки, SQL, JSON)
6. **Интерфейс `Stringer`** для пользовательского форматирования типов
7. **Производительность**: используйте `strings.Builder` для сложного форматирования в циклах
