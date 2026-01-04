## **Работа с XML в Go**

### **1. Введение в XML и пакет encoding/xml**

XML (Extensible Markup Language) — это язык разметки, используемый для кодирования документов в формате, удобном для чтения как человеком, так и машиной. В Go для работы с XML предоставляется пакет `encoding/xml`.

**Код:**

```go
package main

import (
    "encoding/xml"
    "fmt"
    "log"
)

type person struct {
    XMLName xml.Name `xml:"person"`
    Name    string   `xml:"name"`
    Age     int      `xml:"age"`
    City    string   `xml:"city"`
    Email   string   `xml:"email"`
}

func main() {
    p := person{
        Name:  "John",
        Age:   30,
        City:  "London",
        Email: "john@example.com",
    }

    // Простое кодирование в XML
    xmlData, err := xml.Marshal(p)
    if err != nil {
        log.Fatal("Error marshaling data into XML:", err)
    }
    fmt.Println("Simple XML output:")
    fmt.Println(string(xmlData))

    fmt.Println("\n" + "="*50 + "\n")

    // Кодирование с отступами для лучшей читаемости
    xmlDataIndent, err := xml.MarshalIndent(p, "", "  ")
    if err != nil {
        log.Fatal("Error marshaling data into XML:", err)
    }
    fmt.Println("Indented XML output:")
    fmt.Println(string(xmlDataIndent))
}
```

**Объяснение:**

1. Создаем структуру `person` с полями и XML-тегами.
2. `XMLName` — специальное поле, которое определяет корневой элемент XML.
3. `xml.Marshal` кодирует структуру в XML в одну строку.
4. `xml.MarshalIndent` форматирует XML с отступами для лучшей читаемости.

**Вывод в консоли:**

```
Simple XML output:
<person><name>John</name><age>30</age><city>London</city><email>john@example.com</email></person>

==================================================

Indented XML output:
<person>
  <name>John</name>
  <age>30</age>
  <city>London</city>
  <email>john@example.com</email>
</person>
```

### **2. Поле XMLName и его назначение**

**Код:**

```go
package main

import (
    "encoding/xml"
    "fmt"
    "log"
)

// Структура БЕЗ поля XMLName
type personWithoutXMLName struct {
    Name  string `xml:"name"`
    Age   int    `xml:"age"`
}

// Структура С полем XMLName
type personWithXMLName struct {
    XMLName xml.Name `xml:"person"`
    Name    string   `xml:"name"`
    Age     int      `xml:"age"`
}

func main() {
    // Тест без XMLName
    p1 := personWithoutXMLName{
        Name: "Jane",
        Age:  25,
    }

    xmlData1, _ := xml.MarshalIndent(p1, "", "  ")
    fmt.Println("Without XMLName:")
    fmt.Println(string(xmlData1))

    fmt.Println("\n" + "="*50 + "\n")

    // Тест с XMLName
    p2 := personWithXMLName{
        Name: "Jane",
        Age:  25,
    }

    xmlData2, _ := xml.MarshalIndent(p2, "", "  ")
    fmt.Println("With XMLName:")
    fmt.Println(string(xmlData2))
}
```

**Объяснение:**

- Поле `XMLName` определяет имя корневого элемента XML.
- Без `XMLName` корневым элементом становится название структуры (`personWithoutXMLName`).
- С `XMLName` мы можем задать любое имя для корневого элемента.

**Вывод в консоли:**

```
Without XMLName:
<personWithoutXMLName>
  <name>Jane</name>
  <age>25</age>
</personWithoutXMLName>

==================================================

With XMLName:
<person>
  <name>Jane</name>
  <age>25</age>
</person>
```

### **3. Декодирование XML (Unmarshal)**

**Код:**

```go
package main

import (
    "encoding/xml"
    "fmt"
    "log"
)

type person struct {
    XMLName xml.Name `xml:"person"`
    Name    string   `xml:"name"`
    Age     int      `xml:"age"`
    City    string   `xml:"city,omitempty"`
    Email   string   `xml:"email,omitempty"`
}

func main() {
    // XML-данные в виде строки
    xmlRaw := `
<person>
    <name>Jane</name>
    <age>25</age>
</person>`

    var personXML person

    // Декодируем XML в структуру
    err := xml.Unmarshal([]byte(xmlRaw), &personXML)
    if err != nil {
        log.Fatal("Error unmarshalling XML:", err)
    }

    fmt.Printf("Decoded struct: %+v\n", personXML)
    fmt.Println("\nField values:")
    fmt.Println("Name:", personXML.Name)
    fmt.Println("Age:", personXML.Age)
    fmt.Println("City:", personXML.City)  // Пустая строка
    fmt.Println("Email:", personXML.Email) // Пустая строка
}
```

**Объяснение:**

1. Создаем строку с XML-данными.
2. `xml.Unmarshal` декодирует XML в структуру Go.
3. Поле `City` и `Email` будут пустыми, так как они отсутствуют в XML и имеют тег `omitempty`.

**Вывод в консоли:**

```
Decoded struct: {XMLName:{Space: Local:person} Name:Jane Age:25 City: Email:}

Field values:
Name: Jane
Age: 25
City:
Email:
```

### **4. Использование omitempty и - в XML-тегах**

**Код:**

```go
package main

import (
    "encoding/xml"
    "fmt"
)

type person struct {
    XMLName xml.Name `xml:"person"`
    Name    string   `xml:"name"`
    Age     int      `xml:"age,omitempty"`
    City    string   `xml:"city,omitempty"`
    Email   string   `xml:"-"` // Всегда игнорировать это поле
}

func main() {
    p := person{
        Name:  "John",
        Age:   0, // Нулевое значение
        City:  "", // Пустая строка
        Email: "john@example.com", // Будет проигнорировано
    }

    xmlData, _ := xml.MarshalIndent(p, "", "  ")
    fmt.Println(string(xmlData))

    fmt.Println("\n" + "="*50 + "\n")

    // Тест с заполненными полями
    p2 := person{
        Name:  "Jane",
        Age:   30,
        City:  "New York",
        Email: "jane@example.com",
    }

    xmlData2, _ := xml.MarshalIndent(p2, "", "  ")
    fmt.Println(string(xmlData2))
}
```

**Объяснение:**

- `omitempty` пропускает поле, если оно содержит нулевое значение.
- `-` полностью игнорирует поле при кодировании/декодировании.

**Вывод в консоли:**

```
<person>
  <name>John</name>
</person>

==================================================

<person>
  <name>Jane</name>
  <age>30</age>
  <city>New York</city>
</person>
```

### **5. Вложенные структуры в XML**

**Код:**

```go
package main

import (
    "encoding/xml"
    "fmt"
)

type address struct {
    City  string `xml:"city"`
    State string `xml:"state"`
}

type person struct {
    XMLName xml.Name `xml:"person"`
    Name    string   `xml:"name"`
    Age     int      `xml:"age"`
    Address address  `xml:"address"`
}

func main() {
    // Кодирование
    p := person{
        Name: "John",
        Age:  30,
        Address: address{
            City:  "Oakland",
            State: "California",
        },
    }

    xmlData, _ := xml.MarshalIndent(p, "", "  ")
    fmt.Println("Encoded XML:")
    fmt.Println(string(xmlData))

    fmt.Println("\n" + "="*50 + "\n")

    // Декодирование
    xmlRaw := `
<person>
    <name>Jane</name>
    <age>25</age>
    <address>
        <city>San Francisco</city>
        <state>California</state>
    </address>
</person>`

    var personXML person
    xml.Unmarshal([]byte(xmlRaw), &personXML)

    fmt.Println("Decoded struct:")
    fmt.Printf("%+v\n", personXML)
    fmt.Println("\nAccessing nested fields:")
    fmt.Println("City:", personXML.Address.City)
    fmt.Println("State:", personXML.Address.State)
}
```

**Вывод в консоли:**

```
Encoded XML:
<person>
  <name>John</name>
  <age>30</age>
  <address>
    <city>Oakland</city>
    <state>California</state>
  </address>
</person>

==================================================

Decoded struct:
{XMLName:{Space: Local:person} Name:Jane Age:25 Address:{City:San Francisco State:California}}

Accessing nested fields:
City: San Francisco
State: California
```

### **6. Атрибуты XML-элементов**

**Код:**

```go
package main

import (
    "encoding/xml"
    "fmt"
)

type book struct {
    XMLName xml.Name `xml:"book"`
    ISBN    string   `xml:"isbn,attr"`
    Title   string   `xml:"title,attr"`
    Author  string   `xml:"author,attr"`
}

func main() {
    b := book{
        ISBN:   "978-0-306-40615-7",
        Title:  "Go Boot Camp",
        Author: "Ashish",
    }

    xmlData, _ := xml.MarshalIndent(b, "", "  ")
    fmt.Println("Book with attributes:")
    fmt.Println(string(xmlData))

    fmt.Println("\n" + "="*50 + "\n")

    // Тест с префиксом
    xmlDataWithPrefix, _ := xml.MarshalIndent(b, "  ", "  ")
    fmt.Println("With prefix (two spaces):")
    fmt.Println(string(xmlDataWithPrefix))
}
```

**Объяснение:**

- `attr` в теге указывает, что поле должно быть атрибутом элемента, а не дочерним элементом.
- Префикс в `MarshalIndent` добавляется в начале каждой строки.

**Вывод в консоли:**

```
Book with attributes:
<book isbn="978-0-306-40615-7" title="Go Boot Camp" author="Ashish"></book>

==================================================

With prefix (two spaces):
    <book isbn="978-0-306-40615-7" title="Go Boot Camp" author="Ashish"></book>
```

### **7. Поле XMLName как структура**

**Код:**

```go
package main

import (
    "encoding/xml"
    "fmt"
)

type person struct {
    XMLName xml.Name `xml:"person"`
    Name    string   `xml:"name"`
}

func main() {
    p := person{
        Name: "John",
    }

    xmlData, _ := xml.MarshalIndent(p, "", "  ")
    fmt.Println("XML Output:")
    fmt.Println(string(xmlData))

    fmt.Println("\nXMLName fields:")
    fmt.Println("Local:", p.XMLName.Local)
    fmt.Println("Space:", p.XMLName.Space)
}
```

**Объяснение:**

- `XMLName` — это структура типа `xml.Name` с полями `Space` (пространство имен) и `Local` (локальное имя).
- По умолчанию `Space` пустое, а `Local` содержит имя корневого элемента.

**Вывод в консоли:**

```
XML Output:
<person>
  <name>John</name>
</person>

XMLName fields:
Local: person
Space:
```

### **8. Где используется XML в реальном мире**

Автор упоминает несколько областей применения XML:

1. **Android-разработка** — XML используется для разметки UI.
2. **SOAP-сервисы** — протокол для веб-сервисов, использующий XML.
3. **Платежные шлюзы и финансовые сервисы** — многие legacy-системы используют XML.
4. **Spring Framework (Java)** — конфигурация бинов и контекста приложения.
5. **RSS и Atom-ленты** — распространение контента.
6. **EDI (Electronic Data Interchange)** — обмен бизнес-документами.
7. **HL7 (здравоохранение)** — стандарт обмена медицинской информацией.
8. **FIXML (финансы)** — стандарт для электронной торговли.
9. **Правительственные порталы и ERP-системы** — многие legacy-системы используют XML.

### **Итог**

Пакет `encoding/xml` в Go предоставляет мощную поддержку для работы с XML. Ключевые моменты:

1. **XMLName** определяет корневой элемент.
2. **Теги struct** управляют кодированием/декодированием (`xml:"name"`, `omitempty`, `attr`, `-`).
3. **MarshalIndent** форматирует XML для читаемости.
4. **Unmarshal** декодирует XML в структуры Go.
5. **Атрибуты** задаются с помощью `attr` в тегах.
6. **Вложенные структуры** поддерживаются автоматически.

XML все еще широко используется в enterprise-системах, legacy-приложениях и специфических отраслях (финансы, здравоохранение), поэтому знание работы с XML в Go является важным навыком для разработчика.
