**Объяснение текста из видео:**

---

### **1. Что такое фильтрация строк (line filtering)?**

Фильтрация строк — это обработка или изменение строк текста на основе определённых критериев. Она включает построчное чтение текста и применение определённых операций или условий к каждой строке.

Это обычная задача при обработке текста, очистке данных и манипуляции с файлами.

---

### **2. Примеры фильтрации строк:**

1. **Фильтрация по содержанию:** Вывод только строк, содержащих определённое ключевое слово.
2. **Удаление пустых строк:** Исключение пустых строк из вывода.
3. **Преобразование содержимого строк:** Преобразование всего текста в строках в верхний или нижний регистр.
4. **Фильтрация по критериям:** Вывод только строк длиннее определённого количества символов.

---

### **3. Создание тестового файла**

Сначала создадим файл `example.txt` с содержимым:

```
Some important info

Another important piece of data

Go is a valuable addition to programming
```

---

### **4. Пример 1: Фильтрация строк по ключевому слову**

**Код:**

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
)

func main() {
    // Открываем файл
    file, err := os.Open("example.txt")
    if err != nil {
        fmt.Println("Error opening file:", err)
        return
    }
    defer file.Close()

    // Создаём сканер для построчного чтения
    scanner := bufio.NewScanner(file)

    // Ключевое слово для фильтрации
    keyword := "important"

    // Читаем и фильтруем строки
    for scanner.Scan() {
        line := scanner.Text()

        // Если строка содержит ключевое слово, выводим её
        if strings.Contains(line, keyword) {
            fmt.Println(line)
        }
    }

    // Проверяем ошибки сканирования
    if err := scanner.Err(); err != nil {
        fmt.Println("Error scanning file:", err)
        return
    }
}
```

**Что происходит:**

1. `os.Open("example.txt")` открывает файл для чтения
2. `bufio.NewScanner(file)` создаёт сканер для построчного чтения
3. `scanner.Scan()` читает файл построчно
4. `scanner.Text()` возвращает текущую строку
5. `strings.Contains(line, keyword)` проверяет, содержит ли строка ключевое слово "important"
6. Если содержит — выводим строку

**Вывод в консоли:**

```
Some important info
Another important piece of data
```

**Что заметили:**

- Пустые строки не выводятся (они не содержат "important")
- Строка "Go is a valuable addition to programming" не выводится (не содержит "important")

---

### **5. Пример 2: Замена ключевого слова в строках**

**Код (добавляем в основной цикл):**

```go
for scanner.Scan() {
    line := scanner.Text()

    if strings.Contains(line, keyword) {
        // Заменяем "important" на "necessary"
        updatedLine := strings.ReplaceAll(line, "important", "necessary")
        fmt.Println("Original:", line)
        fmt.Println("Updated: ", updatedLine)
        fmt.Println()
    }
}
```

**Что происходит:**

1. `strings.ReplaceAll(line, "important", "necessary")` заменяет все вхождения "important" на "necessary"
2. Выводим оригинальную и изменённую строку

**Вывод в консоли:**

```
Original: Some important info
Updated:  Some necessary info

Original: Another important piece of data
Updated:  Another necessary piece of data
```

---

### **6. Пример 3: Добавление нумерации строк**

**Код:**

```go
func main() {
    file, err := os.Open("example.txt")
    if err != nil {
        fmt.Println("Error opening file:", err)
        return
    }
    defer file.Close()

    scanner := bufio.NewScanner(file)
    keyword := "important"
    lineNumber := 1

    for scanner.Scan() {
        line := scanner.Text()

        if strings.Contains(line, keyword) {
            updatedLine := strings.ReplaceAll(line, "important", "necessary")

            // Выводим с нумерацией строк
            fmt.Printf("Line %d:\n", lineNumber)
            fmt.Printf("\tOriginal: %s\n", line)
            fmt.Printf("\tUpdated:  %s\n\n", updatedLine)
        }

        // Увеличиваем номер строки (для всех строк, включая пустые)
        lineNumber++
    }

    if err := scanner.Err(); err != nil {
        fmt.Println("Error scanning file:", err)
        return
    }
}
```

**Что происходит:**

1. `lineNumber` отслеживает номер текущей строки
2. `fmt.Printf` используется для форматированного вывода
3. `\t` добавляет отступ (табуляцию)
4. Номер строки увеличивается для каждой прочитанной строки (даже если она пустая)

**Вывод в консоли:**

```
Line 1:
    Original: Some important info
    Updated:  Some necessary info

Line 3:
    Original: Another important piece of data
    Updated:  Another necessary piece of data
```

**Обратите внимание:**

- Строки нумеруются 1, 3 (а не 1, 2)
- Это потому, что пустые строки (2 и 4) тоже учитываются в нумерации, но не выводятся

---

### **7. Пример 4: Фильтрация с сохранением оригинальной нумерации**

**Код (показываем одинаковые номера для оригинальной и изменённой строки):**

```go
lineNumber := 1
for scanner.Scan() {
    line := scanner.Text()

    if strings.Contains(line, keyword) {
        updatedLine := strings.ReplaceAll(line, "important", "necessary")

        // Одинаковый номер для обеих строк
        fmt.Printf("Line %d - Original: %s\n", lineNumber, line)
        fmt.Printf("Line %d - Updated:  %s\n\n", lineNumber, updatedLine)
    }

    lineNumber++
}
```

**Вывод в консоли:**

```
Line 1 - Original: Some important info
Line 1 - Updated:  Some necessary info

Line 3 - Original: Another important piece of data
Line 3 - Updated:  Another necessary piece of data
```

**Или с разными номерами:**

Для данного файла: `example.txt`

```
Some important info.

Another important piece of data.
This is going to be important.

Please finish this, this is important.

Go is an important language to learn in this era.

Go is a valuable addition to programming.
```

```go
lineNumber := 1
for scanner.Scan() {
    line := scanner.Text()

    if strings.Contains(line, keyword) {
        updatedLine := strings.ReplaceAll(line, "important", "necessary")

        fmt.Printf("Line %d - Original: %s\n", lineNumber, line)
        lineNumber++
        fmt.Printf("Line %d - Updated:  %s\n\n", lineNumber, updatedLine)
        lineNumber++
    }


}
```

```
1 Original line: Some important info.
2 Updated line: Some necessary info.
3 Original line: Another important piece of data.
4 Updated line: Another necessary piece of data.
5 Original line: This is going to be important.
6 Updated line: This is going to be necessary.
7 Original line: Please finish this, this is important.
8 Updated line: Please finish this, this is necessary.
9 Original line: Go is an important language to learn in this era.
10 Updated line: Go is an necessary language to learn in this era.

```

---

### **8. Лучшие практики фильтрации строк:**

1. **Используйте буферизованный ввод-вывод:** `bufio.Scanner` эффективен при работе с большими объёмами данных.
2. **Всегда обрабатывайте ошибки:** Проверяйте `scanner.Err()` после цикла.
3. **Файлы можно читать из разных источников:**
   - `os.Open()` — из файла
   - `os.Stdin` — стандартный ввод
   - Сетевые потоки
   - Другие реализации `io.Reader`

---

### **9. Практическое применение фильтрации строк:**

1. **Преобразование данных:** Конвертация между разными форматами данных, очистка логов.
2. **Обработка текста:** Извлечение информации, фильтрация определённых строк.
3. **Анализ данных:** Предварительная обработка данных перед дальнейшим анализом или хранением.

---

### **10. Полный пример с несколькими операциями фильтрации:**

**Код:**

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
)

func main() {
    file, err := os.Open("example.txt")
    if err != nil {
        fmt.Println("Error opening file:", err)
        return
    }
    defer file.Close()

    scanner := bufio.NewScanner(file)
    keyword := "important"
    lineNumber := 1

    fmt.Println("=== Filtered lines containing '" + keyword + "' ===")

    for scanner.Scan() {
        line := scanner.Text()

        // Пропускаем пустые строки
        if len(strings.TrimSpace(line)) == 0 {
            lineNumber++
            continue
        }

        // Проверяем наличие ключевого слова
        if strings.Contains(line, keyword) {
            // Преобразуем в верхний регистр
            upperLine := strings.ToUpper(line)

            // Заменяем ключевое слово
            replacedLine := strings.ReplaceAll(line, keyword, "CRITICAL")

            // Выводим результаты
            fmt.Printf("Line %d:\n", lineNumber)
            fmt.Printf("  Original:    %s\n", line)
            fmt.Printf("  Upper case:  %s\n", upperLine)
            fmt.Printf("  Replaced:    %s\n\n", replacedLine)
        }

        lineNumber++
    }

    if err := scanner.Err(); err != nil {
        fmt.Println("Error scanning file:", err)
        return
    }
}
```

**Вывод в консоли:**

```
=== Filtered lines containing 'important' ===
Line 1:
  Original:    Some important info
  Upper case:  SOME IMPORTANT INFO
  Replaced:    Some CRITICAL info

Line 3:
  Original:    Another important piece of data
  Upper case:  ANOTHER IMPORTANT PIECE OF DATA
  Replaced:    Another CRITICAL piece of data
```

---

### **11. Итог:**

Фильтрация строк в Go предоставляет простой подход для обработки входных данных построчно, применения преобразований и генерации вывода. Понимание того, как эффективно использовать `bufio.Scanner`, обрабатывать ошибки и реализовывать логику обработки, гарантирует, что программы фильтрации строк будут эффективными, надёжными и подходящими для различных задач в разработке программного обеспечения.
