## Часть 1: Основы работы с сигналами

### Что такое сигналы?

Сигналы - это форма межпроцессного взаимодействия, используемая для уведомления процессов о определенных событиях или состояниях. Они обычно используются для обработки асинхронных событий, таких как прерывания или завершения работы.

### Для чего используются сигналы:

1. **Graceful shutdown** - позволяет программам корректно обрабатывать прерывания
2. **Очистка ресурсов** - обеспечение правильного освобождения ресурсов перед выходом
3. **Межпроцессное взаимодействие** - уведомление или коммуникация между разными процессами

## Код с объяснениями:

### 1. Создание канала для сигналов

```go
// Создаем канал для сигналов
sigChan := make(chan os.Signal, 1)
```

**Что происходит:**

- Создается буферизированный канал типа `os.Signal`
- Буфер размером 1 означает, что канал может хранить один сигнал
- Это нужно, чтобы сигналы не блокировали отправку

### 2. Настройка обработки сигналов

```go
// Уведомляем канал о сигналах прерывания или завершения
signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
```

**Что происходит:**

- `signal.Notify()` направляет указанные сигналы в наш канал
- `syscall.SIGINT` - сигнал прерывания (обычно Ctrl+C)
- `syscall.SIGTERM` - сигнал завершения

### 3. Горутина для получения сигналов

```go
go func() {
    sig := <-sigChan  // Получаем сигнал из канала
    fmt.Println("\nReceived signal:", sig)
    fmt.Println("Graceful exit")
    os.Exit(0)  // Выход с кодом 0 (успешное завершение)
}()
```

**Что происходит:**

- Создается анонимная функция, выполняющаяся в отдельной горутине
- `<-sigChan` блокируется, пока не придет сигнал
- При получении сигнала выводится сообщение и происходит выход

### 4. Основной цикл программы

```go
// Симулируем работу программы
for {
    fmt.Println("Working...")
    time.Sleep(time.Second)
}
```

**Что происходит:**

- Бесконечный цикл, имитирующий работу программы
- Каждую секунду выводится "Working..."
- Программа работает, пока не будет получен сигнал

## Запуск программы и тестирование

### Результат выполнения:

```bash
$ go run signals.go
Working...
Working...
Working...
^C  # Нажимаем Ctrl+C
Received signal: interrupt
Graceful exit
```

### Получение PID процесса:

```go
pid := os.Getpid()
fmt.Println("Process ID:", pid)
```

Вывод:

```bash
Process ID: 32692
```

### Отправка сигналов из терминала:

1. **SIGINT (прерывание):**

```bash
$ kill -s SIGINT 32692
```

2. **SIGTERM (завершение):**

```bash
$ kill -s SIGTERM 32692
```

3. **SIGHUP (обрыв связи):**

```bash
$ kill -s SIGHUP 32692
```

## Часть 2: Обработка разных сигналов по-разному

### Модифицированный код с обработкой разных сигналов:

```go
go func() {
    sig := <-sigChan
    switch sig {
    case syscall.SIGINT:
        fmt.Println("\nReceived SIGINT - Interrupt signal")
    case syscall.SIGTERM:
        fmt.Println("\nReceived SIGTERM - Terminate signal")
    case syscall.SIGHUP:
        fmt.Println("\nReceived SIGHUP - Hangup signal")
    }
    fmt.Println("Graceful exit")
    os.Exit(0)
}()
```

### Тестирование разных сигналов:

1. **При Ctrl+C:**

```bash
Received SIGINT - Interrupt signal
Graceful exit
```

2. **При отправке SIGTERM:**

```bash
Received SIGTERM - Terminate signal
Graceful exit
```

3. **При отправке SIGHUP:**

```bash
Received SIGHUP - Hangup signal
Graceful exit
```

## Часть 3: Пользовательские сигналы

### Добавление обработки SIGUSR1:

```go
func main() {

	// Получение PID процесса
	pid := os.Getpid()
	fmt.Println("Process ID:", pid)

	// Создаем канал для сигналов
	sigChan := make(chan os.Signal, 1)

	// Уведомляем канал о сигналах прерывания или завершения
	signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM, syscall.SIGHUP, syscall.SIGUSR1)

	go func() {
		for sig := range sigChan { // Получаем сигнал из канала
			switch sig {
			case syscall.SIGINT:
				fmt.Println("\nReceived SIGINT - Interrupt signal")
			case syscall.SIGTERM:
				fmt.Println("\nReceived SIGTERM - Terminate signal")
			case syscall.SIGHUP:
				fmt.Println("\nReceived SIGHUP - Hangup signal")
			case syscall.SIGUSR1:
				fmt.Println("\nReceived SIGUSR1 - User defined signal 1")
				// Продолжаем работу, не выходим
				continue
			}
			fmt.Println("Graceful exit")
			os.Exit(0) // Выход с кодом 0 (успешное завершение)
		}
	}()

	// Симулируем работу программы
	for {
		fmt.Println("Working...")
		time.Sleep(time.Second)
	}
}

```

### Отправка SIGUSR1:

```bash
$ kill -s SIGUSR1 <PID>
```

Вывод:

```bash
Received SIGUSR1 - User defined signal 1
# Программа продолжает работать
```

## Часть 4: Координация горутин с помощью сигналов

### Расширенный пример:

```go
// Создаем дополнительные каналы
sigChan := make(chan os.Signal, 1)
doneChan := make(chan bool, 1)

// Настраиваем обработку сигналов
signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM, syscall.SIGHUP)

// Горутина для обработки сигналов
go func() {
    sig := <-sigChan
    fmt.Println("\nReceived signal:", sig)
    doneChan <- true  // Отправляем сигнал завершения
}()

// Горутина для выполнения работы
go func() {
    for {
        select {
        case <-doneChan:
            fmt.Println("Stopping work due to signal")
            return  // Завершаем горутину
        default:
            fmt.Println("Working...")
            time.Sleep(time.Second)
        }
    }
}()

// Главная горутина ждет
for {
    time.Sleep(time.Second)
}
```

### Результат выполнения:

```bash
Working...
Working...
Working...
^C
Received signal: interrupt
Stopping work due to signal
```

## Часть 5: Сигналы SIGSTOP и SIGCONT

### Особенности SIGSTOP и SIGCONT:

- **SIGSTOP** - приостанавливает выполнение процесса (не может быть перехвачен)
- **SIGCONT** - возобновляет выполнение приостановленного процесса

### Пример использования:

```bash
# Приостанавливаем процесс
$ kill -s SIGSTOP <PID>

# Возобновляем процесс
$ kill -s SIGCONT <PID>
```

## Важные замечания:

### Типы сигналов:

1. **SIGINT** - сигнал прерывания (Ctrl+C)
2. **SIGTERM** - запрос на завершение (корректный shutdown)
3. **SIGHUP** - обрыв соединения, часто используется для перечитывания конфигураций
4. **SIGUSR1, SIGUSR2** - пользовательские сигналы для кастомной логики
5. **SIGKILL** - немедленное завершение (не может быть перехвачен)
6. **SIGSTOP** - приостановка выполнения (не может быть перехвачен)

### Системные различия:

- Команды работают на Unix-подобных системах (Linux, macOS)
- Windows имеет другую систему сигналов
- На Windows можно использовать Task Manager или команды:

  ```cmd
  # Список процессов
  tasklist

  # Завершение процесса
  taskkill /F /PID <PID>
  ```

### Best practices:

1. Всегда обрабатывайте SIGINT и SIGTERM для graceful shutdown
2. Освобождайте ресурсы перед выходом
3. Используйте буферизированные каналы для сигналов
4. Логируйте получение сигналов для отладки

Это полный разбор кода из вашего курса. Каждая часть объяснена так, как это делает автор, с примерами вывода в консоль и подробными комментариями о том, что происходит на каждом шаге.
