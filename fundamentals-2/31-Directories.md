## Объяснение работы с директориями в Go

**Текст автора**: "Directories or folders are containers used to organize files on computer's file system."

**Объяснение**: Директории (или папки) - это контейнеры для организации файлов в файловой системе.

**Текст автора**: "In go, the OS package provides functions for interacting with directories and performing file system operations."

**Объяснение**: В Go пакет `os` предоставляет функции для работы с директориями и файловой системой.

### Ключевые функции:

- `os.Mkdir()` - создает директорию
- `os.MkdirAll()` - создает директории рекурсивно
- `os.ReadDir()` - читает содержимое директории
- `os.Remove()` - удаляет файл или пустую директорию
- `os.RemoveAll()` - удаляет рекурсивно

### Шаг за шагом код автора:

```go
package main

import (
    "fmt"
    "os"
    "path/filepath"
)

// Функция для обработки ошибок
func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    // ШАГ 1: Создание директории с помощью os.Mkdir()
    // Автор создает функцию checkError для обработки ошибок
    // Вместо: if err != nil { fmt.Println(err) }

    // Создаем директорию "subdir" с правами 0755
    // 0755 в Linux означает:
    // - Владелец: чтение, запись, выполнение (7 = 111 в двоичной)
    // - Группа: чтение, выполнение (5 = 101 в двоичной)
    // - Остальные: чтение, выполнение (5 = 101 в двоичной)

    err := os.Mkdir("subdir", 0755)
    checkError(err)

    // Альтернативный способ - одной строкой:
    // checkError(os.Mkdir("subdir", 0755))

    fmt.Println("Директория 'subdir' создана")
    // Вывод: Директория 'subdir' создана

    // ШАГ 2: Создание еще одной директории
    checkError(os.Mkdir("subdir1", 0755))
    fmt.Println("Директория 'subdir1' создана")
    // Вывод: Директория 'subdir1' создана

    // ШАГ 3: Удаление директории с помощью defer
    defer os.RemoveAll("subdir1")
    // defer означает, что эта команда выполнится ПЕРЕД выходом из функции main
    // То есть директория создастся, а потом сразу удалится

    // ШАГ 4: Создание файла в директории
    // os.WriteFile создает файл с содержимым
    // Мы создаем пустой файл в директории subdir

    checkError(os.WriteFile("subdir/file1.txt", []byte{}, 0755))
    fmt.Println("Файл 'subdir/file1.txt' создан")
    // Вывод: Файл 'subdir/file1.txt' создан

    // ШАГ 5: Создание вложенных директорий с помощью os.MkdirAll()
    // MkdirAll создает все директории в пути, если их не существует

    checkError(os.MkdirAll("subdir/parent/child", 0755))
    fmt.Println("Вложенные директории созданы")
    // Вывод: Вложенные директории созданы

    // ШАГ 6: Создание нескольких дочерних директорий
    checkError(os.MkdirAll("subdir/parent/child1", 0755))
    checkError(os.MkdirAll("subdir/parent/child2", 0755))
    checkError(os.MkdirAll("subdir/parent/child3", 0755))

    // ШАГ 7: Создание файлов в разных директориях
    checkError(os.WriteFile("subdir/parent/file2.txt", []byte{}, 0755))
    checkError(os.WriteFile("subdir/parent/child/file3.txt", []byte{}, 0755))

    // ШАГ 8: Чтение содержимого директории с помощью os.ReadDir()
    // ReadDir возвращает срез DirEntry объектов

    entries, err := os.ReadDir("subdir/parent")
    checkError(err)

    fmt.Println("\nСодержимое директории 'subdir/parent':")
    for _, entry := range entries {
        fmt.Println("  Имя:", entry.Name())
        fmt.Println("  Это директория?", entry.IsDir())
        fmt.Println("  Тип:", entry.Type())
        fmt.Println()
    }
    // Вывод:
    // Содержимое директории 'subdir/parent':
    //   Имя: child
    //   Это директория? true
    //   Тип: d---------
    //
    //   Имя: child1
    //   Это директория? true
    //   Тип: d---------
    //
    //   Имя: child2
    //   Это директория? true
    //   Тип: d---------
    //
    //   Имя: child3
    //   Это директория? true
    //   Тип: d---------
    //
    //   Имя: file2.txt
    //   Это директория? false
    //   Тип: ----------

    // ШАГ 9: Смена текущей директории с помощью os.Chdir()
    checkError(os.Chdir("subdir/parent/child"))
    fmt.Println("Перешли в директорию: subdir/parent/child")

    // ШАГ 10: Чтение текущей директории (используем ".")
    entries, err = os.ReadDir(".")
    checkError(err)

    fmt.Println("\nСодержимое текущей директории:")
    for _, entry := range entries {
        fmt.Println("  ", entry.Name())
    }
    // Вывод:
    // Содержимое текущей директории:
    //    file3.txt

    // ШАГ 11: Переход на несколько уровней вверх
    checkError(os.Chdir("../../.."))

    // ШАГ 12: Получение текущей рабочей директории с помощью os.Getwd()
    currentDir, err := os.Getwd()
    checkError(err)

    fmt.Println("\nТекущая директория:", currentDir)
    // Вывод: Текущая директория: /путь/к/вашему/project

    // ШАГ 13: Рекурсивный обход директорий с помощью filepath.WalkDir()
    // WalkDir более эффективен, чем Walk

    fmt.Println("\nРекурсивный обход директории 'subdir':")
    err = filepath.WalkDir("subdir", func(path string, d os.DirEntry, err error) error {
        if err != nil {
            return err
        }
        fmt.Println("  ", path)
        return nil
    })
    checkError(err)
    // Вывод:
    // Рекурсивный обход директории 'subdir':
    //    subdir
    //    subdir/file1.txt
    //    subdir/parent
    //    subdir/parent/child
    //    subdir/parent/child/file3.txt
    //    subdir/parent/child1
    //    subdir/parent/child2
    //    subdir/parent/child3
    //    subdir/parent/file2.txt

    // ШАГ 14: Удаление директорий
    // os.RemoveAll удаляет директорию и всё её содержимое

    fmt.Println("\nУдаляем директорию 'subdir'...")
    checkError(os.RemoveAll("subdir"))
    fmt.Println("Директория 'subdir' удалена")
    // Вывод:
    // Удаляем директорию 'subdir'...
    // Директория 'subdir' удалена

    // ШАГ 15: Разница между Remove и RemoveAll
    // Remove удаляет только пустые директории или файлы
    // RemoveAll удаляет рекурсивно

    // Создаем тестовую директорию с файлом
    checkError(os.Mkdir("testdir", 0755))
    checkError(os.WriteFile("testdir/file.txt", []byte{}, 0755))

    // Попытка удалить с помощью Remove (не сработает, так как директория не пуста)
    // err = os.Remove("testdir")
    // if err != nil {
    //     fmt.Println("Ошибка при удалении:", err)
    //     // Вывод: Ошибка при удалении: directory not empty
    // }

    // Удаляем с помощью RemoveAll
    checkError(os.RemoveAll("testdir"))
    fmt.Println("Директория 'testdir' удалена с помощью RemoveAll")
    // Вывод: Директория 'testdir' удалена с помощью RemoveAll
}
```

### Пояснения автора к коду:

**Текст автора**: "When talking about os dot mkdir call, it creates parent directories if they do not exist"

**Объяснение**: `os.Mkdir()` создает директорию, но если родительских директорий нет - вернет ошибку. Для создания вложенных директорий нужно использовать `os.MkdirAll()`.

**Текст автора**: "Permissions are of type OS dot file mode. So they are in a numerical format. And in Linux the numerical format of a file that I own is 0755"

**Объяснение**: Права доступа в Linux:

- `0755` = владелец: чтение+запись+выполнение (7), группа: чтение+выполнение (5), остальные: чтение+выполнение (5)
- Первая цифра `0` - специальные права (setuid, setgid, sticky bit)

**Текст автора**: "In LS hyphen l we are able to see the permissions as well. So these 0755 this permission is this."

**Объяснение**: В выводе `ls -l`:

- `drwxr-xr-x` = директория с правами 0755
  - `d` - это директория
  - `rwx` - владелец: чтение (r), запись (w), выполнение (x)
  - `r-x` - группа: чтение (r), выполнение (x), нет записи (-)
  - `r-x` - остальные: чтение (r), выполнение (x), нет записи (-)

**Текст автора**: "The difference between walk dir and walk is that walk dir is often preferred for performance reasons."

**Объяснение**:

- `filepath.WalkDir()` использует `os.DirEntry` и более эффективен
- `filepath.Walk()` использует `os.FileInfo` и может быть медленнее

**Текст автора**: "Always check and handle errors returned by directory operations."

**Объяснение**: Всегда проверяйте ошибки, которые возвращают функции работы с директориями.

### Практическое применение:

1. **Организация файлов** - создание структурированной файловой системы
2. **Навигация по файловой системе** - перемещение между директориями
3. **Пакетная обработка** - выполнение операций над множеством файлов

**Текст автора**: "Overall, directories play a vital role in organizing and managing files on a computer's file system."

**Объяснение**: Директории играют важную роль в организации и управлении файлами в файловой системе компьютера. Используя пакет `os` в Go, вы можете эффективно выполнять задачи файловой системы.
