## **Часть 1: Что такое структура на самом деле в памяти?**

### Код 1: Базовое определение структуры и ее представление в памяти

```go
package main

import (
    "fmt"
    "unsafe"
)

// Структура определяет КАК данные будут расположены в памяти
type Person struct {
    firstName string  // 16 байт (на 64-битной системе)
    lastName  string  // 16 байт
    age       int     // 8 байт
}

func main() {
    // Создаем экземпляр структуры
    p := Person{
        firstName: "John",
        lastName:  "Doe",
        age:       30,
    }

    fmt.Println("=== СТРУКТУРА В ПАМЯТИ ===")
    fmt.Printf("Размер структуры Person: %d байт\n", unsafe.Sizeof(p))
    fmt.Printf("Смещение firstName: %d байт\n", unsafe.Offsetof(p.firstName))
    fmt.Printf("Смещение lastName: %d байт\n", unsafe.Offsetof(p.lastName))
    fmt.Printf("Смещение age: %d байт\n", unsafe.Offsetof(p.age))

    // Что на самом деле хранится в памяти?
    fmt.Println("\n=== РАСПОЛОЖЕНИЕ В ПАМЯТИ ===")
    fmt.Printf("Адрес структуры: %p\n", &p)
    fmt.Printf("Адрес firstName: %p\n", &p.firstName)
    fmt.Printf("Адрес lastName: %p\n", &p.lastName)
    fmt.Printf("Адрес age: %p\n", &p.age)
}
```

**Вывод программы (пример):**

```
=== СТРУКТУРА В ПАМЯТИ ===
Размер структуры Person: 40 байт
Смещение firstName: 0 байт
Смещение lastName: 16 байт
Смещение age: 32 байт

=== РАСПОЛОЖЕНИЕ В ПАМЯТИ ===
Адрес структуры: 0xc0000a6000
Адрес firstName: 0xc0000a6000
Адрес lastName: 0xc0000a6010
Адрес age: 0xc0000a6020
```

**Что происходит под капотом:**

```
ПАМЯТЬ для структуры Person (64-битная система):
Адрес           | Поле        | Размер | Значение
0xc0000a6000    | firstName   | 16 байт| [указатель на "John" + длина]
0xc0000a6010    | lastName    | 16 байт| [указатель на "Doe" + длина]
0xc0000a6020    | age         | 8 байт | 30
0xc0000a6028    | padding     | 8 байт| (выравнивание)
```

## **Часть 2: Инициализация структур - разные способы**

### Код 2: Все способы создания и инициализации структур

```go
package main

import "fmt"

type Point struct {
    X, Y int
}

type Circle struct {
    Center Point
    Radius float64
}

func main() {
    fmt.Println("=== СПОСОБЫ ИНИЦИАЛИЗАЦИИ ===")

    // 1. Нулевое значение (zero value)
    var p1 Person  // Все поля получают нулевые значения
    fmt.Printf("1. Нулевая структура: %+v\n", p1)
    // Вывод: {firstName: lastName: age:0}

    // 2. Создание с указанием значений (порядок важен)
    p2 := Person{"John", "Doe", 30}
    fmt.Printf("2. Позиционная инициализация: %+v\n", p2)

    // 3. Именованная инициализация (рекомендуется)
    p3 := Person{
        firstName: "Jane",
        lastName:  "Smith",
        age:       25,
    }
    fmt.Printf("3. Именованная инициализация: %+v\n", p3)

    // 4. Частичная инициализация (остальные поля - нулевые)
    p4 := Person{firstName: "Bob"}
    fmt.Printf("4. Частичная инициализация: %+v\n", p4)

    // 5. Создание через new() - возвращает указатель
    p5 := new(Person)
    p5.firstName = "Alice"
    p5.age = 28
    fmt.Printf("5. Через new(): %+v\n", *p5)

    // 6. Создание указателя напрямую
    p6 := &Person{
        firstName: "Charlie",
        age:       35,
    }
    fmt.Printf("6. Указатель напрямую: %+v\n", *p6)

    // 7. Вложенные структуры
    c := Circle{
        Center: Point{X: 10, Y: 20},
        Radius: 5.5,
    }
    fmt.Printf("\n7. Вложенная структура: %+v\n", c)
    fmt.Printf("   Доступ к X: %d\n", c.Center.X)
}
```

**Вывод программы:**

```
=== СПОСОБЫ ИНИЦИАЛИЗАЦИИ ===
1. Нулевая структура: {firstName: lastName: age:0}
2. Позиционная инициализация: {firstName:John lastName:Doe age:30}
3. Именованная инициализация: {firstName:Jane lastName:Smith age:25}
4. Частичная инициализация: {firstName:Bob lastName: age:0}
5. Через new(): {firstName:Alice lastName: age:28}
6. Указатель напрямую: {firstName:Charlie lastName: age:35}
7. Вложенная структура: {Center:{X:10 Y:20} Radius:5.5}
   Доступ к X: 10
```

## **Часть 3: Методы структур - получатели по значению и по указателю**

### Код 3: Разница между value receiver и pointer receiver

```go
package main

import "fmt"

type Counter struct {
    value int
}

// Метод с получателем по ЗНАЧЕНИЮ (value receiver)
// Работает с КОПИЕЙ структуры
func (c Counter) IncrementByValue() {
    c.value++ // Изменяет копию, оригинал не меняется
    fmt.Printf("Inside IncrementByValue: value = %d (адрес: %p)\n",
        c.value, &c)
}

// Метод с получателем по УКАЗАТЕЛЮ (pointer receiver)
// Работает с ОРИГИНАЛЬНОЙ структурой
func (c *Counter) IncrementByPointer() {
    c.value++ // Изменяет оригинал
    fmt.Printf("Inside IncrementByPointer: value = %d (адрес: %p)\n",
        c.value, c)
}

// Метод только для чтения (value receiver - безопасно)
func (c Counter) GetValue() int {
    return c.value
}

func main() {
    fmt.Println("=== МЕТОДЫ: VALUE RECEIVER ===")
    counter1 := Counter{value: 10}

    fmt.Printf("Before: value = %d (адрес: %p)\n",
        counter1.value, &counter1)
    counter1.IncrementByValue()
    fmt.Printf("After: value = %d\n\n", counter1.value)
    // Значение не изменилось!

    fmt.Println("=== МЕТОДЫ: POINTER RECEIVER ===")
    counter2 := Counter{value: 10}

    fmt.Printf("Before: value = %d (адрес: %p)\n",
        counter2.value, &counter2)
    counter2.IncrementByPointer() // Go автоматически преобразует в (&counter2)
    fmt.Printf("After: value = %d\n\n", counter2.value)
    // Значение изменилось!

    // Автоматические преобразования
    fmt.Println("=== АВТОМАТИЧЕСКИЕ ПРЕОБРАЗОВАНИЯ ===")
    counter3 := Counter{value: 100}
    ptr := &counter3

    // Метод с value receiver - можно вызывать и у значения, и у указателя
    counter3.GetValue()  // OK
    ptr.GetValue()       // OK - Go автоматически разыменовывает указатель (*ptr).GetValue()

    // Метод с pointer receiver - можно вызывать и у значения, и у указателя
    counter3.IncrementByPointer()  // OK - Go автоматически берет адрес (&counter3)
    ptr.IncrementByPointer()       // OK
}
```

**Вывод программы:**

```
=== МЕТОДЫ: VALUE RECEIVER ===
Before: value = 10 (адрес: 0xc0000180a8)
Inside IncrementByValue: value = 11 (адрес: 0xc0000180c0)
After: value = 10

=== МЕТОДЫ: POINTER RECEIVER ===
Before: value = 10 (адрес: 0xc0000180d8)
Inside IncrementByPointer: value = 11 (адрес: 0xc0000180d8)
After: value = 11

=== АВТОМАТИЧЕСКИЕ ПРЕОБРАЗОВАНИЯ ===
```

**Что происходит под капотом:**

1. **Value receiver**: `func (c Counter) Method()`

   - Go создает КОПИЮ структуры
   - Метод работает с копией
   - Изменения не сохраняются

2. **Pointer receiver**: `func (c *Counter) Method()`

   - Go передает АДРЕС структуры (8 байт)
   - Метод работает с оригиналом
   - Изменения сохраняются

3. **Автоматические преобразования**:
   - Вызов метода у указателя: `ptr.Method()` → Go разыменовывает: `(*ptr).Method()`
   - Вызов метода у значения: `val.Method()` → Go берет адрес: `(&val).Method()`

## **Часть 4: Встраивание структур (Embedding)**

### Код 4: Композиция через встраивание

```go
package main

import "fmt"

// Базовая структура
type ContactInfo struct {
    email   string
    phone   string
    address string
}

// Встраивание ContactInfo в Person
type Person struct {
    firstName string
    lastName  string
    age       int
    ContactInfo  // Встраивание (embedding)
}

// Отдельная структура (без встраивания)
type PersonWithoutEmbedding struct {
    firstName string
    lastName  string
    age       int
    contact   ContactInfo  // Поле, а не встраивание
}

func main() {
    fmt.Println("=== ВСТРАИВАНИЕ СТРУКТУР ===")

    // Создаем Person с встраиванием
    person := Person{
        firstName: "John",
        lastName:  "Doe",
        age:       30,
        ContactInfo: ContactInfo{
            email:   "john@example.com",
            phone:   "+1234567890",
            address: "123 Main St",
        },
    }

    // Поля ContactInfo становятся доступны напрямую!
    fmt.Println("Email:", person.email)       // Вместо person.ContactInfo.email
    fmt.Println("Phone:", person.phone)       // Вместо person.ContactInfo.phone
    fmt.Println("Address:", person.address)   // Вместо person.ContactInfo.address

    // Но можно и через полный путь
    fmt.Println("Email (полный путь):", person.ContactInfo.email)

    fmt.Println("\n=== БЕЗ ВСТРАИВАНИЯ ===")
    person2 := PersonWithoutEmbedding{
        firstName: "Jane",
        lastName:  "Smith",
        age:       25,
        contact: ContactInfo{
            email: "jane@example.com",
        },
    }

    // Без встраивания нужен полный путь
    // person2.email - ОШИБКА, такого поля нет
    fmt.Println("Email:", person2.contact.email)  // Только так

    fmt.Println("\n=== МЕТОДЫ ВСТРАИВАЕМЫХ СТРУКТУР ===")

    // Добавляем метод к ContactInfo
    contact := ContactInfo{
        email: "test@example.com",
        phone: "+1234567890",
    }
    fmt.Println("Contact summary:", contact.GetSummary())

    // Метод автоматически "продвигается" в Person
    person3 := Person{
        firstName: "Bob",
        ContactInfo: contact,
    }
    fmt.Println("Person contact summary:", person3.GetSummary())
}

// Метод для ContactInfo
func (c ContactInfo) GetSummary() string {
    return fmt.Sprintf("Email: %s, Phone: %s", c.email, c.phone)
}
```

**Вывод программы:**

```
=== ВСТРАИВАНИЕ СТРУКТУР ===
Email: john@example.com
Phone: +1234567890
Address: 123 Main St
Email (полный путь): john@example.com

=== БЕЗ ВСТРАИВАНИЯ ===
Email: jane@example.com

=== МЕТОДЫ ВСТРАИВАЕМЫХ СТРУКТУР ===
Contact summary: Email: test@example.com, Phone: +1234567890
Person contact summary: Email: test@example.com, Phone: +1234567890
```

**Что происходит под капотом:**

1. **Встраивание** - это композиция, а не наследование
2. Поля и методы встроенной структуры "продвигаются" (promoted) во внешнюю
3. В памяти это выглядит так:

```go
// Person с встраиванием:
[firstName][lastName][age][email][phone][address]

// Person без встраивания:
[firstName][lastName][age][contact]
                     где contact = [email][phone][address]
```

## **Часть 5: Анонимные структуры и анонимные поля**

### Код 5: Анонимные структуры и их использование

```go
package main

import "fmt"

func main() {
    fmt.Println("=== АНОНИМНЫЕ СТРУКТУРЫ ===")

    // Анонимная структура - без объявления типа
    user := struct {
        id       int
        username string
        isActive bool
    }{
        id:       1,
        username: "john_doe",
        isActive: true,
    }

    fmt.Printf("Анонимная структура: %+v\n", user)
    fmt.Printf("Тип: %T\n", user)  // struct { id int; username string; isActive bool }

    fmt.Println("\n=== ПЕРЕДАЧА АНОНИМНЫХ СТРУКТУР В ФУНКЦИИ ===")

    // Часто используется для одноразовых данных
    printUserInfo(struct {
        name string
        age  int
    }{
        name: "Alice",
        age:  30,
    })

    fmt.Println("\n=== АНОНИМНЫЕ ПОЛЯ (АНОНИМНОЕ ВСТРАИВАНИЕ) ===")

    type Config struct {
        string  // Анонимное поле (тип = имя поля)
        int     // Анонимное поле
    }

    config := Config{
        string: "production",
        int:    8080,
    }

    // Доступ через тип (так как нет имени)
    fmt.Printf("String field: %s\n", config.string)
    fmt.Printf("Int field: %d\n", config.int)

    // Но так делать не рекомендуется - лучше использовать именованные поля
}

func printUserInfo(user struct {
    name string
    age  int
}) {
    fmt.Printf("Имя: %s, Возраст: %d\n", user.name, user.age)
}
```

**Вывод программы:**

```
=== АНОНИМНЫЕ СТРУКТУРЫ ===
Анонимная структура: {id:1 username:john_doe isActive:true}
Тип: struct { id int; username string; isActive bool }

=== ПЕРЕДАЧА АНОНИМНЫХ СТРУКТУР В ФУНКЦИИ ===
Имя: Alice, Возраст: 30

=== АНОНИМНЫЕ ПОЛЯ (АНОНИМНОЕ ВСТРАИВАНИЕ) ===
String field: production
Int field: 8080
```

## **Часть 6: Сравнение структур и теги**

### Код 6: Сравнение и теги структур

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    // Теги структур (struct tags) - метаданные для пакетов
    ID        int    `json:"id" db:"user_id" validate:"required,min=1"`
    FirstName string `json:"first_name" db:"first_name"`
    LastName  string `json:"last_name,omitempty" db:"last_name"`
    Age       int    `json:"age" validate:"gte=0,lte=130"`
    Password  string `json:"-"` // Тире означает "не сериализовать"
}

func main() {
    fmt.Println("=== СРАВНЕНИЕ СТРУКТУР ===")

    // Структуры сравниваются ПОЛЕ за ПОЛЕМ
    u1 := User{ID: 1, FirstName: "John", Age: 30}
    u2 := User{ID: 1, FirstName: "John", Age: 30}
    u3 := User{ID: 2, FirstName: "Jane", Age: 25}

    fmt.Printf("u1 == u2: %v\n", u1 == u2) // true
    fmt.Printf("u1 == u3: %v\n", u1 == u3) // false

    // Сравнение с нулевым значением
    var zeroUser User
    fmt.Printf("zeroUser == User{}: %v\n", zeroUser == User{}) // true

    fmt.Println("\n=== ТЕГИ СТРУКТУР ===")

    user := User{
        ID:        1,
        FirstName: "John",
        LastName:  "", // Пустая строка
        Age:       30,
        Password:  "secret123",
    }

    // Сериализация в JSON
    jsonData, err := json.MarshalIndent(user, "", "  ")
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }

    fmt.Println("JSON представление:")
    fmt.Println(string(jsonData))
    // Обратите внимание:
    // 1. Имена полей изменены (snake_case)
    // 2. LastName отсутствует (omitempty)
    // 3. Password отсутствует (json:"-")

    fmt.Println("\n=== ДЕСЕРИАЛИЗАЦИЯ ИЗ JSON ===")

    jsonStr := `{"id": 2, "first_name": "Jane", "age": 25}`
    var newUser User
    json.Unmarshal([]byte(jsonStr), &newUser)
    fmt.Printf("Десериализовано: %+v\n", newUser)
}
```

**Вывод программы:**

```
=== СРАВНЕНИЕ СТРУКТУР ===
u1 == u2: true
u1 == u3: false
zeroUser == User{}: true

=== ТЕГИ СТРУКТУР ===
JSON представление:
{
  "id": 1,
  "first_name": "John",
  "age": 30
}

=== ДЕСЕРИАЛИЗАЦИЯ ИЗ JSON ===
Десериализовано: {ID:2 FirstName:Jane LastName: Age:25 Password:}
```

## **Часть 7: Что происходит под капотом - детали реализации**

### Код 7: Внутреннее устройство структур

```go
package main

import (
    "fmt"
    "reflect"
    "unsafe"
)

type Employee struct {
    id     int
    name   string
    salary float64
}

func main() {
    fmt.Println("=== ВНУТРЕННЕЕ УСТРОЙСТВО ===")

    emp := Employee{
        id:     101,
        name:   "John Doe",
        salary: 50000.50,
    }

    // Размер и выравнивание
    fmt.Printf("Размер Employee: %d байт\n", unsafe.Sizeof(emp))
    fmt.Printf("Выравнивание Employee: %d байт\n", unsafe.Alignof(emp))

    // Информация о типах через рефлексию
    t := reflect.TypeOf(emp)
    fmt.Printf("\nТип: %v\n", t)
    fmt.Printf("Количество полей: %d\n", t.NumField())

    // Информация о каждом поле
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        fmt.Printf("Поле %d: %s, тип: %v, теги: %v\n",
            i, field.Name, field.Type, field.Tag)
    }

    // Доступ к полям через рефлексию
    v := reflect.ValueOf(emp)
    fmt.Printf("\nЗначение поля 'name': %v\n", v.FieldByName("name"))

    fmt.Println("\n=== ПЕРЕДАЧА СТРУКТУР В ФУНКЦИИ ===")

    // Передача по значению (копирование)
    fmt.Printf("До вызова: emp.id = %d\n", emp.id)
    modifyByValue(emp)
    fmt.Printf("После вызова: emp.id = %d (не изменился)\n", emp.id)

    // Передача по указателю (без копирования)
    modifyByPointer(&emp)
    fmt.Printf("После pointer вызова: emp.id = %d (изменился)\n", emp.id)

    fmt.Println("\n=== ОПТИМИЗАЦИЯ: ПУСТЫЕ СТРУКТУРЫ ===")

    // Пустая структура занимает 0 байт!
    empty := struct{}{}
    fmt.Printf("Размер пустой структуры: %d байт\n", unsafe.Sizeof(empty))

    // Часто используется в каналах для сигналов
    signal := make(chan struct{})
    go func() {
        // Отправляем сигнал (не передавая данных)
        signal <- struct{}{}
    }()

    <-signal // Ждем сигнал
    fmt.Println("Сигнал получен!")
}

func modifyByValue(e Employee) {
    e.id = 999 // Изменяет копию
}

func modifyByPointer(e *Employee) {
    e.id = 888 // Изменяет оригинал
}
```

**Вывод программы:**

```
=== ВНУТРЕННЕЕ УСТРОЙСТВО ===
Размер Employee: 32 байт
Выравнивание Employee: 8 байт

Тип: main.Employee
Количество полей: 3
Поле 0: id, тип: int, теги:
Поле 1: name, тип: string, теги:
Поле 2: salary, тип: float64, теги:

Значение поля 'name': John Doe

=== ПЕРЕДАЧА СТРУКТУР В ФУНКЦИИ ===
До вызова: emp.id = 101
После вызова: emp.id = 101 (не изменился)
После pointer вызова: emp.id = 888 (изменился)

=== ОПТИМИЗАЦИЯ: ПУСТЫЕ СТРУКТУРЫ ===
Размер пустой структуры: 0 байт
Сигнал получен!
```

## **Шпаргалка по структурам:**

```
ОСНОВЫ:
  type Name struct {       // Объявление
      Field1 Type1
      Field2 Type2
  }

ИНИЦИАЛИЗАЦИЯ:
  var p Person                 // Нулевые значения
  p := Person{}               // Нулевые значения
  p := Person{Field: value}   // Именованная
  p := new(Person)            // Указатель

ДОСТУП К ПОЛЯМ:
  p.Field = value             // Установка
  value = p.Field             // Получение

МЕТОДЫ:
  func (p Person) Method()    // Value receiver (копия)
  func (p *Person) Method()   // Pointer receiver (оригинал)

ВСТРАИВАНИЕ:
  type Child struct {
      Parent                  // Поля Parent доступны напрямую
  }

СРАВНЕНИЕ:
  s1 == s2                    // Сравнивает все поля

ТЕГИ:
  `json:"name"`               // Метаданные для JSON
  `db:"column_name"`          // Метаданные для БД
  `validate:"required"`       // Валидация

ОСОБЕННОСТИ:
  - Структуры передаются по значению (копируются)
  - Используйте указатели для больших структур
  - Методы могут быть с value или pointer receiver
  - Встраивание ≠ наследование (это композиция)
  - Пустая struct{} занимает 0 байт
```

**Ключевые выводы:**

1. **Структуры** - это способ группировки данных в памяти
2. **Методы** - функции, привязанные к типу (value или pointer receiver)
3. **Встраивание** - композиция (не наследование!)
4. **Сравнение** работает поле за полем
5. **Теги** - метаданные для сериализации и валидации
6. **Передача по значению** копирует всю структуру, **по указателю** - только адрес
