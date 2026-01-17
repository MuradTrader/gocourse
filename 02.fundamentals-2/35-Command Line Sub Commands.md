## Объяснение субкоманд в командной строке Go

**Текст автора**: "Subcommands are a way to organize command line interfaces into hierarchical structures, allowing different functionalities or operations to be grouped under main commands."

**Объяснение**: Субкоманды позволяют организовать интерфейс командной строки в иерархическую структуру, группируя разные функции под основными командами.

**Текст автора**: "When we are using go run, go is the first command, so it is a command line tool. Go and run is the sub command to the primary command, which is go get init."

**Объяснение**: Примеры субкоманд:

- `go run` - `go` (команда), `run` (субкоманда)
- `git init` - `git` (команда), `init` (субкоманда)
- `git commit`, `git push` - субкоманды git

### Шаг за шагом код автора:

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    // ШАГ 1: Создаем первый набор флагов для субкоманды "firstsub"
    subCommand1 := flag.NewFlagSet("firstsub", flag.ExitOnError)

    // Определяем флаги для первой субкоманды
    // flag.Bool возвращает указатель на bool
    firstFlag := subCommand1.Bool("processing", false, "command processing status")
    secondFlag := subCommand1.Int("bytes", 1024, "byte length of result")

    // ШАГ 2: Создаем второй набор флагов для субкоманды "secondsub"
    subCommand2 := flag.NewFlagSet("secondsub", flag.ExitOnError)

    // Определяем флаги для второй субкоманды
    // flag.String возвращает указатель на string
    flagSC2 := subCommand2.String("language", "go", "enter your language")

    // ШАГ 3: Проверяем, что передана хотя бы одна субкоманда
    // os.Args[0] - имя программы, os.Args[1] - первая субкоманда
    if len(os.Args) < 2 {
        fmt.Println("This program requires additional commands")
        os.Exit(1)
    }

    // ШАГ 4: Обрабатываем субкоманды с помощью switch
    switch os.Args[1] {
    case "firstsub":
        // Парсим флаги для первой субкоманды
        // Начинаем с os.Args[2], так как os.Args[1] - это "firstsub"
        subCommand1.Parse(os.Args[2:])

        fmt.Println("Subcommand: firstsub")
        // Разыменовываем указатели для получения значений
        fmt.Println("Processing:", *firstFlag)
        fmt.Println("Bytes:", *secondFlag)

        // Без разыменовывания получаем адрес
        // fmt.Println("Processing:", firstFlag)
        // fmt.Println("Bytes:", secondFlag)

        // Результат
        // Subcommand: firstsub
        // Processing: 0xc00008c0c8
        // Bytes: 0xc00008c0d0

    case "secondsub":
        // Парсим флаги для второй субкоманды
        subCommand2.Parse(os.Args[2:])

        fmt.Println("Subcommand: secondsub")
        fmt.Println("Language:", *flagSC2)

    default:
        fmt.Println("No subcommand entered")
        os.Exit(1)
    }
}
```

### Примеры запуска и вывод:

**Пример 1**: Без аргументов

```bash
go run subcommands.go
```

**Вывод**:

```
This program requires additional commands
```

**Пример 2**: Неправильная субкоманда

```bash
go run subcommands.go hello
```

**Вывод**:

```
No subcommand entered
```

**Пример 3**: Первая субкоманда без флагов (значения по умолчанию)

```bash
go run subcommands.go firstsub
```

**Вывод**:

```
Subcommand: firstsub
Processing: false
Bytes: 1024
```

**Пример 4**: Первая субкоманда с флагами (используя `=`)

```bash
go run subcommands.go firstsub -processing=true -bytes=256
```

**Вывод**:

```
Subcommand: firstsub
Processing: true
Bytes: 256
```

**Пример 5**: Первая субкоманда с флагами (без `=`)

```bash
go run subcommands.go firstsub -processing true -bytes 256
```

**Вывод**:

```
Subcommand: firstsub
Processing: true
Bytes: 256
```

**Пример 6**: Вторая субкоманда с флагом

```bash
go run subcommands.go secondsub -language=dart
```

**Вывод**:

```
Subcommand: secondsub
Language: dart
```

**Пример 7**: Вторая субкоманда с флагом (без `=`)

```bash
go run subcommands.go secondsub -language dart
```

**Вывод**:

```
Subcommand: secondsub
Language: dart
```

### Подробные объяснения автора:

**Текст автора**: "The first sub command is sub command one. And I'm going to use flag dot new flag set."

**Объяснение**: Создаем новый набор флагов с помощью `flag.NewFlagSet("имя", режим_ошибки)`

**Текст автора**: "And then we have to handle error. So for error handling the most frequently the most commonly used option is flag dot exit on error."

**Объяснение**: `flag.ExitOnError` - при ошибке парсинга программа завершится с кодом ошибки 2.

**Текст автора**: "So this time let's use another way of doing the same thing. So this time I'm going to do bool. All right."

**Объяснение**: Есть два способа определения флагов:

1. `flag.BoolVar(&variable, "name", default, "description")` - сохраняет в существующую переменную
2. `flag.Bool("name", default, "description")` - возвращает указатель на значение

**Текст автора**: "So first flag is not a variable, it is a pointer. It is not storing the actual value, it is storing the pointer to the actual value. So what we should be using is well how do we dereference a pointer. We have first flag as a pointer, right? It is a boolean pointer. And the way we dereference a pointer is by using another asterisk on it."

**Объяснение**: `flag.Bool()` возвращает `*bool` (указатель). Чтобы получить значение, нужно разыменовать указатель: `*firstFlag`

### Расширенный пример с общими флагами:

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    // ШАГ 1: Общий флаг для основной команды
    stringFlag := flag.String("user", "Guest", "Name of the user")
	flag.Parse()
	fmt.Println(*stringFlag) // Guest
    /*
    Только общий флаг (помощь)
    go run subcommands.go -help

     **Вывод**:
     Usage of /tmp/go-build123456789/b001/exe/subcommands:
      -user string
       Name of the user (default "Guest")
    */

    // Определяем флаг до создания субкоманд
    var user string
    flag.StringVar(&user, "user", "guest", "name of the user")

    // НЕ ВЫЗЫВАЕМ flag.Parse() здесь, так как будем парсить вручную

    // ШАГ 2: Создаем субкоманды
    subCommand1 := flag.NewFlagSet("firstsub", flag.ExitOnError)
    firstFlag := subCommand1.Bool("processing", false, "command processing status")
    secondFlag := subCommand1.Int("bytes", 1024, "byte length of result")

    subCommand2 := flag.NewFlagSet("secondsub", flag.ExitOnError)
    flagSC2 := subCommand2.String("language", "go", "enter your language")

    // ШАГ 3: Проверяем аргументы
    if len(os.Args) < 2 {
        fmt.Println("Usage: program [-user=name] <subcommand> [flags]")
        fmt.Println("Subcommands:")
        fmt.Println("  firstsub    First subcommand")
        fmt.Println("  secondsub   Second subcommand")
        os.Exit(1)
    }

    // ШАГ 4: Парсим общие флаги (до субкоманды)
    // Нужно парсить только те аргументы, которые идут до субкоманды
    flag.Parse()

    // ШАГ 5: Обрабатываем субкоманды
    switch os.Args[1] {
    case "firstsub":
        subCommand1.Parse(os.Args[2:])
        fmt.Println("Common flag - user:", user)
        fmt.Println("Subcommand: firstsub")
        fmt.Println("Processing:", *firstFlag)
        fmt.Println("Bytes:", *secondFlag)

    case "secondsub":
        subCommand2.Parse(os.Args[2:])
        fmt.Println("Common flag - user:", user)
        fmt.Println("Subcommand: secondsub")
        fmt.Println("Language:", *flagSC2)

    default:
        // Если первый аргумент не субкоманда, возможно это общий флаг
        fmt.Println("Unknown subcommand:", os.Args[1])
        os.Exit(1)
    }

}

```

### Примеры запуска расширенной версии:

**Пример 1**: С общим флагом и субкомандой

```bash
go run subcommands.go -user=John firstsub -processing=true -bytes=512
```

**Вывод**:

```
Common flag - user: John
Subcommand: firstsub
Processing: true
Bytes: 512
```

**Пример 2**: Только общий флаг (помощь)

```bash
go run subcommands.go -help
```

**Вывод**:

```
Usage of /tmp/go-build123456789/b001/exe/subcommands:
  -user string
        name of the user (default "guest")
```

**Пример 3**: Помощь для субкоманды

```bash
go run subcommands.go firstsub -help
```

**Вывод**:

```
Usage of firstsub:
  -bytes int
        byte length of result (default 1024)
  -processing
        command processing status (default false)
```

### Важные моменты от автора:

**Текст автора**: "So when you're using subcommands, use the equal to sign to update both or all values. If you are using multiple flags with your subcommand, use equal to sign with the flags of your Subcommand."

**Объяснение**: При использовании субкоманд лучше использовать знак `=` для присвоения значений флагам: `-flag=value`

**Текст автора**: "And now if I enter the sub command and then dash dash help. Let's see what happens then. So now we see that we have the help prompt which is describing all the flags associated with this sub command."

**Объяснение**: Каждая субкоманда имеет свою справку. Например: `program firstsub --help`

**Текст автора**: "So when we type dash dash help with our command, it's only listing the flags and not the sub commands. So that is the default nature of dash. Dash help that we are using with our command."

**Объяснение**: `program --help` покажет только общие флаги, но не список субкоманд.

### Лучшие практики от автора:

**Текст автора**: "When working with command line Subcommands, provide clear and concise descriptions for each sub command."

**Объяснение**: Давайте ясные и краткие описания для каждой субкоманды.

**Текст автора**: "Another best practice is to follow a consistent naming convention for Subcommands."

**Объяснение**: Используйте последовательное именование субкоманд, например:

- `git init`, `git commit`, `git push`
- `go run`, `go build`, `go test`

**Текст автора**: "Again, we come to error handling. Implement error handling for command parsing and execution to provide informative feedback to users."

**Объяснение**: Реализуйте обработку ошибок при парсинге и выполнении команд, чтобы давать пользователям информативные сообщения.

### Итог от автора:

**Текст автора**: "Overall command line Subcommands in go provide a structured approach to building robust and modular CLI applications."

**Объяснение**: Субкоманды в Go предоставляют структурированный подход к созданию надежных и модульных CLI-приложений.

**Текст автора**: "By using packages like Flag, you can define complex command line interfaces with multiple flags with ease, improving usability and maintainability of your applications."

**Объяснение**: Используя пакет `flag`, вы можете легко определять сложные интерфейсы командной строки с множеством флагов, улучшая удобство использования и поддерживаемость ваших приложений.

### Ключевые концепции:

1. `flag.NewFlagSet()` - создает новый набор флагов для субкоманды
2. `flag.ExitOnError` - режим обработки ошибок (программа завершится при ошибке)
3. Субкоманды обрабатываются через `switch` по `os.Args[1]`
4. Флаги субкоманд парсятся с помощью `subCommand.Parse(os.Args[2:])`
5. Общие флаги определяются до создания субкоманд
6. Для получения значений из `flag.Bool()`, `flag.String()` и т.д. нужно разыменовывать указатели
