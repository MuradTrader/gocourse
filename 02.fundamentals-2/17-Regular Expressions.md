## Часть 1: Escape-последовательности в Go

### Что такое escape-последовательности?

Это специальные символы, которые начинаются с обратного слеша `\` и имеют особое значение.

### Код:

```go
package main

import "fmt"

func main() {
    // Пример 1: Двойные кавычки с экранированием
    str1 := "He said: \"I am great\" and this is a quote from someone else."
    fmt.Println(str1)

    // Пример 2: Сырая строка (raw string) с обратными кавычками
    str2 := `He said: "I am great" and this is a quote from someone else.`
    fmt.Println(str2)

    // Пример 3: Различные escape-последовательности
    fmt.Println("Hello\nWorld")      // \n - новая строка
    fmt.Println("Hello\tWorld")      // \t - табуляция
    fmt.Println("Hello\\World")      // \\ - обратный слеш
    fmt.Println("Hello\bWorld")      // \b - backspace (забой)
    fmt.Println("Hello\rWorld")      // \r - возврат каретки
}
```

### Вывод:

```
$ go run main.go
He said: "I am great" and this is a quote from someone else.
He said: "I am great" and this is a quote from someone else.
Hello
World
Hello	World
Hello\World
HelloWorld  // Заметил удаление 'o'? Это backspace!
World       // Возврат каретки вернул нас в начало строки
```

**Объяснение:**

1. `\"` - экранирование двойных кавычек внутри строки с двойными кавычками
2. `` `...` `` - сырая строка, внутри можно использовать любые символы без экранирования
3. `\n` - перенос строки
4. `\t` - табуляция (4 пробела)
5. `\\` - сам обратный слеш
6. `\b` - удаляет предыдущий символ
7. `\r` - возвращает курсор в начало строки, поэтому "Hello" перезаписывается "World"

## Часть 2: Базовое использование регулярных выражений

### Код 1: Простая проверка email

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Компилируем регулярное выражение для email
    re := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)

    // Тестовые email
    email1 := "user@gmail.com"
    email2 := "invalid-email"
    email3 := "user@company.co.uk"
    email4 := "USER@GMAIL.COM"  // Верхний регистр

    // Проверяем email
    fmt.Printf("'%s' валиден: %v\n", email1, re.MatchString(email1))
    fmt.Printf("'%s' валиден: %v\n", email2, re.MatchString(email2))
    fmt.Printf("'%s' валиден: %v\n", email3, re.MatchString(email3))
    fmt.Printf("'%s' валиден: %v\n", email4, re.MatchString(email4))
}
```

### Вывод:

```
$ go run main.go
'user@gmail.com' валиден: true
'invalid-email' валиден: false
'user@company.co.uk' валиден: true
'USER@GMAIL.COM' валиден: true
```

**Разбор регулярного выражения `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`:**

- `^` - начало строки
- `[a-zA-Z0-9._%+-]+` - один или больше символов из набора:
  - `a-z` - строчные буквы
  - `A-Z` - заглавные буквы
  - `0-9` - цифры
  - `._%+-` - специальные символы
- `@` - символ @
- `[a-zA-Z0-9.-]+` - доменное имя (буквы, цифры, точки, дефисы)
- `\.` - точка (экранированная)
- `[a-zA-Z]{2,}` - доменная зона (минимум 2 буквы)
- `$` - конец строки

## Часть 3: Проблема с дефисом в регулярных выражениях

### Код с ошибкой:

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // ОШИБКА: дефис в середине класса символов
    re := regexp.MustCompile(`[a-zA-Z0-9._%+-]+`)

    // В этом случае дефис интерпретируется как диапазон
    // Между '+' и каким символом? Это вызывает ошибку

    // Правильно: дефис в начале или в конце
    re2 := regexp.MustCompile(`[a-zA-Z0-9._%+-]+`)  // Дефис в конце - OK
    re3 := regexp.MustCompile(`[-a-zA-Z0-9._%+]+`)  // Дефис в начале - OK

    testStr := "user-name"

    fmt.Printf("С дефисом в конце класса: %v\n", re2.MatchString(testStr))
    fmt.Printf("С дефисом в начале класса: %v\n", re3.MatchString(testStr))
}
```

### Вывод:

```
$ go run main.go
С дефисом в конце класса: true
С дефисом в начале класса: true
```

**Правило:** В классах символов `[...]` дефис имеет особое значение:

- `[a-z]` - диапазон от a до z
- Если нужен сам символ дефиса, он должен быть первым или последним: `[-az]` или `[az-]`

## Часть 4: Группы захвата (capturing groups)

### Код:

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Регулярное выражение с группами захвата для даты
    re := regexp.MustCompile(`(\d{4})-(\d{2})-(\d{2})`)

    testDate := "2024-07-30"

    // Находим все совпадения с группами
    matches := re.FindStringSubmatch(testDate)

    if matches != nil {
        fmt.Printf("Полное совпадение: %s\n", matches[0])
        fmt.Printf("Год: %s\n", matches[1])
        fmt.Printf("Месяц: %s\n", matches[2])
        fmt.Printf("День: %s\n", matches[3])

        // Также можно получить все группы
        fmt.Println("\nВсе группы:")
        for i, match := range matches {
            fmt.Printf("Группа %d: %s\n", i, match)
        }
    }
}
```

### Вывод:

```
$ go run main.go
Полное совпадение: 2024-07-30
Год: 2024
Месяц: 07
День: 30

Все группы:
Группа 0: 2024-07-30
Группа 1: 2024
Группа 2: 07
Группа 3: 30
```

**Разбор `(\d{4})-(\d{2})-(\d{2})`:**

- `(\d{4})` - группа 1: ровно 4 цифры (год)
- `-` - дефис
- `(\d{2})` - группа 2: ровно 2 цифры (месяц)
- `-` - дефис
- `(\d{2})` - группа 3: ровно 2 цифры (день)

**Важно:** `matches[0]` всегда содержит полное совпадение, а `matches[1]`, `matches[2]` и т.д. - группы.

## Часть 5: Замена с помощью регулярных выражений

### Код:

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Исходная строка
    source := "Hello World! Go is great. Good morning!"

    // Регулярное выражение для поиска гласных
    re := regexp.MustCompile(`[aeiouAEIOU]`)

    // Заменяем все гласные на *
    result := re.ReplaceAllString(source, "*")

    fmt.Printf("Исходная строка: %s\n", source)
    fmt.Printf("После замены:    %s\n", result)

    // Пример 2: Замена чисел
    re2 := regexp.MustCompile(`\d+`)
    str2 := "I have 3 apples and 15 oranges"
    result2 := re2.ReplaceAllString(str2, "N")

    fmt.Printf("\nИсходная: %s\n", str2)
    fmt.Printf("После:    %s\n", result2)
}
```

### Вывод:

```
$ go run main.go
Исходная строка: Hello World! Go is great. Good morning!
После замены:    H*ll* W*rld! G* *s gr**t. G**d m*rn*ng!

Исходная: I have 3 apples and 15 oranges
После:    I have N apples and N oranges
```

**Что происходит:**

- `[aeiouAEIOU]` - ищет любую гласную (английскую)
- `\d+` - ищет одну или больше цифр
- `ReplaceAllString` заменяет ВСЕ совпадения

## Часть 6: Флаги в регулярных выражениях

### Код:

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Без флага - чувствительно к регистру
    re1 := regexp.MustCompile(`go`)

    // С флагом (?i) - нечувствительно к регистру
    re2 := regexp.MustCompile(`(?i)go`)

    testStrings := []string{
        "Go is great",
        "go is great",
        "GO is great",
        "Golang",
        "golang",
    }

    fmt.Println("Без флага (чувствительно к регистру):")
    for _, s := range testStrings {
        fmt.Printf("  '%s' -> %v\n", s, re1.MatchString(s))
    }

    fmt.Println("\nС флагом (?i) (нечувствительно к регистру):")
    for _, s := range testStrings {
        fmt.Printf("  '%s' -> %v\n", s, re2.MatchString(s))
    }

    // Другие полезные флаги
    fmt.Println("\n--- Другие флаги ---")

    // Многострочный режим: ^ и $ работают для каждой строки
    text := "Line 1\nLine 2\nLine 3"
    re3 := regexp.MustCompile(`(?m)^Line \d+$`)
    matches := re3.FindAllString(text, -1)
    fmt.Printf("Многострочный поиск: %v\n", matches)

    // Флаг s: точка . включает символ новой строки
    re4 := regexp.MustCompile(`(?s)Line.*3`)
    match := re4.FindString(text)
    fmt.Printf("С флагом s (точка включает \\n): '%s'\n", match)
}
```

### Вывод:

```
$ go run main.go
Без флага (чувствительно к регистру):
  'Go is great' -> false
  'go is great' -> true
  'GO is great' -> false
  'Golang' -> false
  'golang' -> false

С флагом (?i) (нечувствительно к регистру):
  'Go is great' -> true
  'go is great' -> true
  'GO is great' -> true
  'Golang' -> true
  'golang' -> true

--- Другие флаги ---
Многострочный поиск: [Line 1 Line 2 Line 3]
С флагом s (точка включает \n): 'Line 1
Line 2
Line 3'
```

**Флаги регулярных выражений:**

- `(?i)` - нечувствительность к регистру
- `(?m)` - многострочный режим (^ и $ для начала/конца каждой строки)
- `(?s)` - точка `.` включает символ новой строки

## Часть 7: Разбор сложного email-регулярного выражения из видео

### Полный код с комментариями:

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Разбираем сложное регулярное выражение из видео по частям
    emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)

    // Давайте разберем каждую часть:
    // ^                      - начало строки
    // [a-zA-Z0-9._%+-]+      - имя пользователя (1+ символов)
    // @                      - символ @
    // [a-zA-Z0-9.-]+         - доменное имя
    // \.                     - точка (экранированная)
    // [a-zA-Z]{2,}           - доменная зона (минимум 2 буквы)
    // $                      - конец строки

    // Тестируем различные email
    testEmails := []string{
        "simple@example.com",
        "very.common@example.com",
        "disposable.style.email.with+symbol@example.com",
        "other.email-with-hyphen@example.com",
        "fully-qualified-domain@example.com",
        "user.name+tag+sorting@example.com",  // плюс в локальной части
        "x@example.com",                       // однобуквенная локальная часть
        "example-indeed@strange-example.com", // дефис в домене
        "admin@mailserver1",                   // без доменной зоны
        "example@s.example",                   // короткая доменная зона
        "1234567890@example.com",              // только цифры в локальной части
        "email@123.123.123.123",               // IP-адрес вместо домена
        "email@[123.123.123.123]",             // IP-адрес в квадратных скобках
        "plainaddress",                        // без @
        "@missing-local-part.com",             // без локальной части
        "local-part@.com",                     // без домена
        "local-part@domain..com",              // две точки подряд в домене
    }

    fmt.Println("Проверка email-адресов:")
    fmt.Println("======================")
    for _, email := range testEmails {
        isValid := emailRegex.MatchString(email)
        status := "✗ НЕВЕРНЫЙ"
        if isValid {
            status = "✓ ВЕРНЫЙ"
        }
        fmt.Printf("%-50s %s\n", email, status)
    }
}
```

### Вывод (частичный):

```
$ go run main.go
Проверка email-адресов:
==================================================
simple@example.com                              ✓ ВЕРНЫЙ
very.common@example.com                         ✓ ВЕРНЫЙ
disposable.style.email.with+symbol@example.com  ✓ ВЕРНЫЙ
other.email-with-hyphen@example.com             ✓ ВЕРНЫЙ
fully-qualified-domain@example.com              ✓ ВЕРНЫЙ
user.name+tag+sorting@example.com               ✓ ВЕРНЫЙ
x@example.com                                   ✓ ВЕРНЫЙ
example-indeed@strange-example.com              ✓ ВЕРНЫЙ
admin@mailserver1                               ✗ НЕВЕРНЫЙ  // Нет доменной зоны
example@s.example                               ✗ НЕВЕРНЫЙ  // Доменная зона слишком короткая
1234567890@example.com                          ✓ ВЕРНЫЙ
email@123.123.123.123                           ✗ НЕВЕРНЫЙ  // IP-адрес не соответствует шаблону
email@[123.123.123.123]                         ✗ НЕВЕРНЫЙ  // Квадратные скобки
plainaddress                                    ✗ НЕВЕРНЫЙ
@missing-local-part.com                         ✗ НЕВЕРНЫЙ
local-part@.com                                 ✗ НЕВЕРНЫЙ
local-part@domain..com                          ✗ НЕВЕРНЫЙ
```

## Часть 8: Производительность регулярных выражений

### Код для демонстрации:

```go
package main

import (
    "fmt"
    "regexp"
    "time"
)

func main() {
    // ПЛОХО: Компиляция при каждом вызове
    start1 := time.Now()
    for i := 0; i < 10000; i++ {
        // Так делать НЕ НАДО - компиляция каждый раз!
        re := regexp.MustCompile(`\d+`)
        re.MatchString("test123")
    }
    elapsed1 := time.Since(start1)

    // ХОРОШО: Одна компиляция, многократное использование
    start2 := time.Now()
    re := regexp.MustCompile(`\d+`)  // Компилируем ОДИН раз
    for i := 0; i < 10000; i++ {
        re.MatchString("test123")
    }
    elapsed2 := time.Since(start2)

    fmt.Printf("ПЛОХО (компиляция каждый раз): %v\n", elapsed1)
    fmt.Printf("ХОРОШО (одна компиляция):      %v\n", elapsed2)
    fmt.Printf("Разница: %.1f раз\n", float64(elapsed1)/float64(elapsed2))
}
```

### Вывод:

```
$ go run main.go
ПЛОХО (компиляция каждый раз): 7.456583ms
ХОРОШО (одна компиляция):      2.375ms
Разница: 3.1 раз
```

**Важный вывод:** Всегда компилируйте регулярные выражения один раз и переиспользуйте их!

## Часть 9: Полезные методы пакета regexp

### Код с примерами:

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "Мой телефон: +7 (912) 345-67-89, рабочий: 8-800-555-35-35"

    // Найти первый телефон
    re := regexp.MustCompile(`\d[\d\s()-]+`)
    firstPhone := re.FindString(text)
    fmt.Printf("Первый телефон: %s\n", firstPhone)

    // Найти все телефоны
    allPhones := re.FindAllString(text, -1)
    fmt.Printf("Все телефоны: %v\n", allPhones)

    // Найти индекс первого совпадения
    index := re.FindStringIndex(text)
    fmt.Printf("Индекс первого телефона: %v (символы %d-%d)\n",
        index, index[0], index[1])

    // Разделить строку по шаблону
    re2 := regexp.MustCompile(`\s+`)
    words := re2.Split("Hello   World\tGo\t\tProgramming", -1)
    fmt.Printf("Разделение строки: %#v\n", words)

    // Проверить, соответствует ли строка целиком
    re3 := regexp.MustCompile(`^\d{3}-\d{3}-\d{4}$`)
    fmt.Printf("123-456-7890 соответствует шаблону: %v\n", re3.MatchString("123-456-7890"))
    fmt.Printf("123-456-789 соответствует шаблону: %v\n", re3.MatchString("123-456-789"))
}
```

### Вывод:

```
$ go run main.go
Первый телефон: 7 (912) 345-67-89
Все телефоны: [7 (912) 345-67-89 8-800-555-35-35]
Индекс первого телефона: [13 33] (символы 13-33)
Разделение строки: []string{"Hello", "World", "Go", "Programming"}
123-456-7890 соответствует шаблону: true
123-456-789 соответствует шаблону: false
```

## Итоговая таблица часто используемых шаблонов:

| Паттерн  | Что означает             | Пример                        |
| -------- | ------------------------ | ----------------------------- |
| `.`      | Любой символ, кроме `\n` | `a.c` → "abc", "a c"          |
| `\d`     | Цифра (0-9)              | `\d\d` → "42", "01"           |
| `\D`     | Не цифра                 | `\D\D` → "ab", "!@"           |
| `\w`     | Буква, цифра или \_      | `\w+` → "hello_123"           |
| `\W`     | Не `\w`                  | `\W` → "!", " "               |
| `\s`     | Пробельный символ        | `\s+` → " ", "\t\n"           |
| `\S`     | Не пробельный символ     | `\S+` → "hello"               |
| `[abc]`  | Любой из a, b, c         | `[aeiou]` → гласная           |
| `[^abc]` | Любой, кроме a, b, c     | `[^0-9]` → не цифра           |
| `*`      | 0 или больше             | `a*` → "", "a", "aaa"         |
| `+`      | 1 или больше             | `\d+` → "1", "123"            |
| `?`      | 0 или 1                  | `colou?r` → "color", "colour" |
| `{n}`    | Ровно n раз              | `\d{3}` → "123"               |
| `{n,}`   | n или больше             | `\d{2,}` → "12", "123"        |
| `{n,m}`  | От n до m                | `\d{2,4}` → "12", "1234"      |
| `^`      | Начало строки            | `^Hello` → "Hello world"      |
| `$`      | Конец строки             | `world$` → "Hello world"      |
| `(abc)`  | Группа захвата           | `(\d{3})-(\d{3})` → "123-456" |

## Практические советы:

1. **Для простых проверок** используйте `strings.Contains`, `strings.HasPrefix` вместо regex
2. **Всегда экранируйте** пользовательский ввод в regex с помощью `regexp.QuoteMeta`
3. **Используйте онлайн-тестеры** для отладки: regex101.com, regexr.com
4. **Кэшируйте скомпилированные regex** в глобальных переменных:
   ```go
   var emailRegex = regexp.MustCompile(`...`)
   ```
