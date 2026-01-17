### 1. **Суть `recover` (дословно + дополнения)**

**Текст автора:**

> "Recover is a built-in function that regains control of a panicking goroutine. It's only useful inside defer functions."

**Разбор:**

- `recover()` — встроенная функция, которая **останавливает распространение паники**.

- **Работает только внутри `defer`-функций!** Если вызвать `recover()` вне `defer`, она вернет `nil` и не повлияет на панику.

- **Механизм:**

- При панике выполнение останавливается, начинается раскрутка стека.

- Если в процессе раскрутки встречается `defer` с вызовом `recover()`, паника останавливается.

- `recover()` возвращает значение, переданное в `panic()`.

- Управление возвращается в вызывающую функцию (программа продолжает работу).

---

### 2. **Пример кода с `recover`**

**Текст автора:**

```go

func process() {

defer func() {

if r := recover(); r != nil {

fmt.Println("Recovered:", r)

}

}()

fmt.Println("Start process")

panic("something went wrong")

fmt.Println("Process saved") // unreachable!

}

func main() {

process()

fmt.Println("Returned from process")

}

```

**Разбор:**

- **Шаги выполнения:**

1. `process()` начинает работу → печатает `"Start process"`.

2. Вызов `panic("something went wrong")`:

- Выполнение останавливается.

- Начинается раскрутка стека.

3. Выполняется `defer`-функция:

- `recover()` перехватывает панику → возвращает `"something went wrong"`.

- Условие `r != nil` выполняется → печатает `"Recovered: something went wrong"`.

4. Паника остановлена → управление возвращается в `main()`.

5. Выполняется `fmt.Println("Returned from process")`.

**Вывод программы:**

```

Start process

Recovered: something went wrong

Returned from process  // программа не упала!

```

**Ключевые моменты:**

- `fmt.Println("Process saved")` никогда не выполнится (код после `panic` недостижим).

- Без `recover` программа завершилась бы с аварийным сообщением.

- `recover` работает только в той же горутине, где вызвана паника.

---

### 3. **Как работает `recover` (технические детали)**

**Текст автора:**

> "If there is no panic, recover returns nil."

**Разбор:**

- **Поведение `recover()`:**

- **При панике:** возвращает значение, переданное в `panic()`.

- **Без паники:** возвращает `nil`.

- **Важно:** `recover()` останавливает **только текущую панику**. Если в `defer` происходит новая паника, она продолжится.

---

### 4. **Практическое применение**

**Текст автора:**

> "Use recover to handle unexpected errors, log panics, and prevent crashes."

**Разбор:**

- **Типичные сценарии:**

1. **Грациозное завершение:**

Завершить обработку HTTP-запроса при панике, но продолжить работу сервера.

```go

func HandleRequest(w http.ResponseWriter, r *http.Request) {

defer func() {

if err := recover(); err != nil {

log.Printf("Panic: %v", err)

http.Error(w, "Internal error", http.StatusInternalServerError)

}

}()

// Основная логика обработки

}

```

2. **Логирование ошибок:**

Записать детали паники в лог (ошибка, стек вызовов).

```go

defer func() {

if r := recover(); r != nil {

log.Printf("Panic: %v\nStack: %s", r, debug.Stack())

}

}()

```

3. **Очистка ресурсов:**

Гарантированно закрыть файлы/соединения даже при панике.

```go

file, _ := os.Open("data.txt")

defer file.Close()

defer func() {

if r := recover(); r != nil {

fmt.Println("Cleaning up after panic")

}

}()

```

---

### 5. **Лучшие практики (Best Practices)**

**Текст автора:**

> "Use recover sparingly. Prefer error handling for expected errors."

**Разбор:**

- **✅ Правильно:**

- Использовать `recover` для:

- Логирования критических ошибок.

- Предотвращения падения сервера/долгоживущих процессов.

- Очистки ресурсов.

- Всегда добавлять детали в лог (`debug.Stack()`, время, контекст).

- **❌ Избегать:**

- "Тихого" подавления паники (без логирования).

- Использования `recover` для обычной логики (например, проверки условий).

- Возврата к выполнению после паники без понимания причины (риск неконсистентного состояния).

**Золотое правило:**

> `panic`/`recover` — для **непредвиденных** ошибок (например, "инвариант нарушен"). Для ожидаемых ошибок используйте `error`.

---

### 6. **Ограничения `recover`**

- Работает только в **текущей горутине**. Паника в дочерней горутине не будет перехвачена `recover` в родительской (если нет своего `defer`+`recover`).

- Не восстанавливает выполнение с места паники — только завершает функцию с `defer`.

- После `recover` программа продолжает работу **следующей инструкцией после функции**, где была паника.

---

### Итог лекции

> "Recover enables programs to recover from panics gracefully. Use it with defer for logging and cleanup, but prefer standard error handling where possible."

**Ключевые тезисы:**

1. `recover` останавливает панику только внутри `defer`.

2. Всегда проверяйте `recover() != nil`.

3. Логируйте паники — не игнорируйте их!

4. Не злоупотребляйте: `panic`/`recover` ≠ замена `error`.
