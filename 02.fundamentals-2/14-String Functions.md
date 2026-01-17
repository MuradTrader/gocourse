Отлично! Я объясню **максимально подробно**, показывая вывод каждого куска кода и объясняя, что происходит на уровне памяти и процессора.

## Давайте пройдемся по каждой части кода:

### 1. Создание строки и проверка длины

```go
package main

import "fmt"

func main() {
    str := "hello go!"
    fmt.Println(len(str)) // Длина строки
}
```

**Что происходит:**

- В памяти создается массив байт: `[104 101 108 108 111 32 103 111 33]`
- Каждый символ кодируется в ASCII (или UTF-8):
  - `h` = 104
  - `e` = 101
  - `l` = 108
  - `o` = 111
  - ` ` (пробел) = 32
  - `g` = 103
  - `o` = 111
  - `!` = 33

**Вывод:**

```
9
```

Потому что 9 символов = 9 байт (все символы ASCII).

### 2. Конкатенация через `+`

```go
str1 := "Hello"
str2 := "World"
result := str1 + " " + str2
fmt.Println(result)
```

**В памяти:**

1. Создается `str1` → байты: `[72 101 108 108 111]`
2. Создается `str2` → байты: `[87 111 114 108 100]`
3. **Важно:** Оператор `+` создает НОВУЮ область памяти!
4. Выделяется память на 5 + 1 + 5 = 11 байт
5. Копируются байты: `[72 101 108 108 111 32 87 111 114 108 100]`

**Вывод:**

```
Hello World
```

### 3. Доступ к символам по индексу

```go
str := "Hello"
fmt.Println(str[0])   // ASCII код 'H'
fmt.Printf("%c\n", str[0]) // Сам символ 'H'
```

**Что происходит:**

- `str[0]` возвращает **байт**, а не символ
- В Go строки - это просто массивы байт

**Вывод:**

```
72    // ASCII код
H     // Символ (после форматирования)
```

### 4. Срезы строк (подстроки)

```go
str := "hello go!"
substr := str[1:5]
fmt.Println(substr)
```

**Под капотом:**

```
Исходная строка в памяти:
Индекс: 0  1  2  3  4  5  6  7  8
Байты:  h  e  l  l  o     g  o  !
       104 101 108 108 111 32 103 111 33

str[1:5] означает:
- Начать с индекса 1 (байт 101 = 'e')
- Закончить ДО индекса 5 (байт 32 = пробел)
```

**Вывод:**

```
ello
```

### 5. Преобразование числа в строку

```go
import "strconv"

num := 18
str3 := strconv.Itoa(num)
fmt.Println(str3)
fmt.Println(len(str3))
```

**Что происходит внутри `Itoa`:**

1. Число 18 делится на 10: остаток 8 → символ '8'
2. Частное 1 делится на 10: остаток 1 → символ '1'
3. Собираем в обратном порядке: "18"

**Вывод:**

```
18   // Строка "18"
2    // Длина строки
```

### 6. Разделение строки

```go
import "strings"

fruits := "apple, orange, banana"
parts := strings.Split(fruits, ",")
fmt.Println(parts)
```

**В памяти:**

```
Исходная строка: "apple, orange, banana"
После Split с разделителем ",":

parts[0] = "apple"        // срез оригинальной строки [0:5]
parts[1] = " orange"      // срез оригинальной строки [6:13]
parts[2] = " banana"      // срез оригинальной строки [14:21]
```

**Вывод:**

```
[apple  orange  banana]
```

Обратите внимание на пробелы перед orange и banana!

### 7. Объединение строк

```go
countries := []string{"Germany", "France", "Italy"}
joined := strings.Join(countries, ", ")
fmt.Println(joined)
```

**Как работает Join:**

1. Вычисляет общую длину: 7 + 2 + 6 + 2 + 5 = 22 символа
2. Создает буфер на 22 байта
3. Последовательно копирует:
   - "Germany" (7 байт)
   - ", " (2 байта)
   - "France" (6 байт)
   - ", " (2 байта)
   - "Italy" (5 байт)

**Вывод:**

```
Germany, France, Italy
```

### 8. Проверка содержания подстроки

```go
str := "hello go!"
fmt.Println(strings.Contains(str, "go"))    // true
fmt.Println(strings.Contains(str, "go?"))   // false
```

**Алгоритм поиска:**

1. Ищет первый символ 'g' в строке
2. Находит на позиции 6
3. Проверяет следующий символ 'o'
4. Совпадение найдено!

**Вывод:**

```
true
false
```

### 9. Замена подстроки

```go
str := "hello go!"
replaced := strings.Replace(str, "go", "world", 1)
fmt.Println(replaced)
```

**Процесс замены:**

```
Исходная:  h  e  l  l  o     g  o  !
           0  1  2  3  4  5  6  7  8

Найти "go" (позиции 6-7)
Заменить на "world" (5 символов)
Новая строка: "hello world!"
```

**Вывод:**

```
hello world!
```

### 10. Удаление пробелов

```go
strWithSpace := "  Hello!  "
fmt.Printf("'%s'\n", strWithSpace)
trimmed := strings.TrimSpace(strWithSpace)
fmt.Printf("'%s'\n", trimmed)
```

**Вывод:**

```
'  Hello!  '   // С пробелами
'Hello!'       // Без пробелов
```

### 11. Изменение регистра

```go
str := "Hello Everyone!"
lower := strings.ToLower(str)
upper := strings.ToUpper(str)
fmt.Println(lower)
fmt.Println(upper)
```

**Преобразование символов:**

- 'H' (72) → 'h' (104): разница 32 в ASCII
- 'E' (69) → 'e' (101): разница 32

**Вывод:**

```
hello everyone!
HELLO EVERYONE!
```

### 12. Повторение строки

```go
repeated := strings.Repeat("foo ", 3)
fmt.Println(repeated)
```

**Вывод:**

```
foo foo foo
```

### 13. Подсчет вхождений

```go
str := "hello"
fmt.Println(strings.Count(str, "l"))
```

**Вывод:**

```
2  // Две буквы 'l' в "hello"
```

### 14. Проверка начала/окончания

```go
str := "hello"
fmt.Println(strings.HasPrefix(str, "he"))  // true
fmt.Println(strings.HasSuffix(str, "lo"))  // true
fmt.Println(strings.HasSuffix(str, "la"))  // false
```

**Вывод:**

```
true
true
false
```

### 15. Регулярные выражения

```go
import "regexp"

str5 := "hello 123 go 456"
re := regexp.MustCompile(`\d+`)
matches := re.FindAllString(str5, -1)
fmt.Println(matches)
```

**Что делает `\d+`:**

- `\d` - любая цифра (0-9)
- `+` - одно или больше повторений

**Вывод:**

```
[123 456]  // Нашел две последовательности цифр
```

### 16. Подсчет символов UTF-8

```go
import "unicode/utf8"

str6 := "Привет"
fmt.Println("Байт:", len(str6))
fmt.Println("Символов:", utf8.RuneCountInString(str6))
```

**Важно:** Кириллические символы занимают 2 байта в UTF-8

**Вывод:**

```
Байт: 12      // 6 символов × 2 байта
Символов: 6   // 6 символов
```

### 17. strings.Builder (ВНИМАНИЕ! Важная тема)

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    // 1. Создаем Builder
    var builder strings.Builder

    // 2. Добавляем строки
    builder.WriteString("Hello")
    builder.WriteRune(' ')  // Добавляем пробел как руну
    builder.WriteString("World")

    // 3. Получаем результат
    result := builder.String()
    fmt.Println(result)

    // 4. Сбрасываем Builder
    builder.Reset()

    // 5. Строим новую строку
    builder.WriteString("Starting fresh")
    result = builder.String()
    fmt.Println(result)
}
```

**Что происходит в памяти:**

1. При создании Builder выделяет начальный буфер (обычно 64 байта)
2. `WriteString` добавляет байты в буфер
3. Если буфер заполняется, он автоматически увеличивается
4. `String()` создает новую строку из буфера

**Вывод:**

```
Hello World
Starting fresh
```

### 18. Демонстрация иммутабельности строк

```go
package main

import "fmt"

func main() {
    original := "Hello"
    fmt.Printf("Адрес оригинальной строки: %p\n", &original)

    // Пытаемся "изменить" строку
    modified := original + " World"
    fmt.Printf("Адрес измененной строки: %p\n", &modified)

    fmt.Println("Оригинал:", original)
    fmt.Println("Измененная:", modified)

    // Докажем, что подстроки делят память
    base := "Hello World"
    sub1 := base[0:5]  // "Hello"
    sub2 := base[6:]   // "World"

    // Go не позволяет получить указатель на байт строки напрямую,
    // но мы можем доказать через преобразование в []byte
    byteSlice := []byte(base)
    fmt.Printf("Первый символ base: %c\n", byteSlice[0])
    byteSlice[0] = 'h'  // Можем изменить байт в slice
    fmt.Println("После изменения байта в slice:", string(byteSlice))
    fmt.Println("Но оригинальная строка остается:", base)
}
```

**Вывод:**

```
Адрес оригинальной строки: 0xc000010250
Адрес измененной строки: 0xc000010260
Оригинал: Hello
Измененная: Hello World
Первый символ base: H
После изменения байта в slice: hello World
Но оригинальная строка остается: Hello World
```

### Полный пример со всеми операциями:

```go
package main

import (
    "fmt"
    "strings"
    "strconv"
    "regexp"
    "unicode/utf8"
)

func main() {
    // 1. Основные операции
    str := "Hello Go!"
    fmt.Println("1. Длина строки:", len(str))

    // 2. Конкатенация
    result := "Hello" + " " + "World"
    fmt.Println("2. Конкатенация:", result)

    // 3. Доступ по индексу
    fmt.Printf("3. Первый символ: %c (ASCII: %d)\n", str[0], str[0])

    // 4. Подстрока
    fmt.Println("4. Подстрока [1:5]:", str[1:5])

    // 5. Число в строку
    numStr := strconv.Itoa(42)
    fmt.Println("5. Число как строка:", numStr, "Длина:", len(numStr))

    // 6. Split
    data := "apple,orange,banana"
    parts := strings.Split(data, ",")
    fmt.Println("6. Split:", parts)

    // 7. Join
    joined := strings.Join([]string{"Go", "Python", "Java"}, " | ")
    fmt.Println("7. Join:", joined)

    // 8. Contains
    fmt.Println("8. Contains 'Go':", strings.Contains("Hello Go!", "Go"))

    // 9. Replace
    replaced := strings.Replace("I like cats and cats", "cats", "dogs", 2)
    fmt.Println("9. Replace:", replaced)

    // 10. Trim
    trimmed := strings.TrimSpace("  Hello  ")
    fmt.Printf("10. Trim: '%s'\n", trimmed)

    // 11. Upper/Lower
    fmt.Println("11. Upper:", strings.ToUpper("hello"))
    fmt.Println("    Lower:", strings.ToLower("HELLO"))

    // 12. Repeat
    fmt.Println("12. Repeat:", strings.Repeat("Na ", 4) + "Batman!")

    // 13. Count
    fmt.Println("13. Count 'l':", strings.Count("hello world", "l"))

    // 14. HasPrefix/HasSuffix
    filename := "document.pdf"
    fmt.Println("14. Is PDF?", strings.HasSuffix(filename, ".pdf"))

    // 15. Регулярки
    text := "Order 123, Order 456, Order 789"
    re := regexp.MustCompile(`\d+`)
    numbers := re.FindAllString(text, -1)
    fmt.Println("15. Найденные числа:", numbers)

    // 16. UTF-8
    russian := "Привет мир"
    fmt.Printf("16. Русский текст: %s\n", russian)
    fmt.Println("   Байт:", len(russian))
    fmt.Println("   Символов:", utf8.RuneCountInString(russian))

    // 17. Builder
    var builder strings.Builder
    for i := 1; i <= 3; i++ {
        builder.WriteString(fmt.Sprintf("Часть %d ", i))
    }
    fmt.Println("17. Builder результат:", builder.String())
}
```

**Вывод этой программы:**

```
1. Длина строки: 9
2. Конкатенация: Hello World
3. Первый символ: H (ASCII: 72)
4. Подстрока [1:5]: ello
5. Число как строка: 42 Длина: 2
6. Split: [apple orange banana]
7. Join: Go | Python | Java
8. Contains 'Go': true
9. Replace: I like dogs and dogs
10. Trim: 'Hello'
11. Upper: HELLO
    Lower: hello
12. Repeat: Na Na Na Na Batman!
13. Count 'l': 3
14. Is PDF? true
15. Найденные числа: [123 456 789]
16. Русский текст: Привет мир
   Байт: 19
   Символов: 10
17. Builder результат: Часть 1 Часть 2 Часть 3
```

## Ключевые выводы:

1. **Строки - это байты** - всегда помните, что `len()` возвращает байты, а не символы
2. **Иммутабельность** - строки нельзя изменить, можно только создать новые
3. **Срезы эффективны** - подстроки используют ту же память
4. **Builder для производительности** - используйте при частой конкатенации
5. **UTF-8 везде** - Go использует UTF-8 для всех строк
