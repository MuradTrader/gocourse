## Introduction: Why It’s Important to Understand the Difference Between `go run` and `go build`

- **What is Go?** Go (or Golang) is a programming language and a set of tools that let you write a program (code) and turn it into an executable program (a binary) that can be run on a computer.

- **What does the `go run` command do?** It compiles your code on the fly and immediately runs it, leaving no file behind that you could run again and again.

- **What does the `go build` command do?** It compiles your code and saves the result as a permanent program (an application file) that you can run as many times as you like.

If these terms (compilation, binary, HDD/SSD, RAM) don’t mean anything to you yet — don’t worry. We’ll break everything down step by step in what follows.

---

## 1. Why do people even ask: “Why use `go run` if `go build` exists?”

> **The author says:**
> “Why do we use `go run` when we could simply run `go build`? It’s simple. `go build` creates a ‘permanent’ program (a binary), whereas `go run` creates a ‘temporary’ program, immediately runs it, and doesn’t leave anything on the disk.

### 1.1. A Brief Overview of «Permanent» and «Temporary» Binaries

1. **`go build` (permanent binary)**

- When you run `go build`, the Go utility reads your files with the `.go` extension (source files), converts them into machine code (instructions for the processor), and writes the resulting file (called a «binary») next to your source files (usually in the same folder).

- This file remains on the disk until you delete it. When you run such a program again, no recompilation is needed — you simply launch the already compiled program.

Вот корректный и естественный перевод на английский 👇

---

**2. go run (temporary binary)**

- When you run `go run`, the Go toolchain also reads the source files and compiles them into machine code, but it does not place the result in the directory where your `.go` files are located. Instead, Go quickly creates a **temporary executable file** somewhere “for a short time” (for example, in the `/tmp` directory, or it may even keep it only in memory).

- As soon as the compiled program finishes its execution, Go automatically deletes this temporary file. In the directory with your `.go` source files, nothing remains except the source files themselves.

<details>
  <summary><b>📖 Подробное разборное объяснение (кликни)</b></summary>

## Заголовок

### **2. go run (temporary binary)**

- **go run** — команда Go
  - **go** — инструмент (Go tool)
  - **run** — _запускать_

- **temporary** — временный
  - от слова **temp** = временно
  - часто встречается в IT: `temp file`, `temporary directory`

- **binary** — бинарный файл (исполняемый файл)
  - программа в виде машинного кода
  - то, что реально запускается ОС

📌 **temporary binary** = _временный исполняемый файл_

---

## Предложение 1

### **When you run go run,**

- **When** — когда
  👉 вводит условие / момент времени
- **you** — ты / вы
- **run** — запускаешь
- **go run** — команда

📌 **When you run go run**
= _Когда ты запускаешь команду go run_

---

### **the Go toolchain also reads the source files**

- **the** — _определённый артикль_
  👉 речь о конкретном инструменте Go

- **Go toolchain** — инструменты Go
  - **tool** — инструмент
  - **chain** — цепочка
    👉 _набор инструментов компиляции_

- **also** — тоже, также
  👉 важное слово, показывает **дополнение**

- **reads** — читает
  - **read → reads** (потому что `toolchain` — это **it**)

- **the source files** — исходные файлы
  - **source** — исходный
  - **files** — файлы

📌
**the Go toolchain also reads the source files**
= _инструменты Go также читают исходные файлы_

---

### **and compiles them into machine code,**

- **and** — и
- **compiles** — компилирует
  - compile → compiles (3-е лицо)

- **them** — их
  👉 заменяет `source files`
- **into** — в (в результате превращения!)
  👉 очень важно:
  - **into** = превращение

- **machine code** — машинный код

📌
**compiles them into machine code**
= _компилирует их в машинный код_

---

### **but it does not place the result**

- **but** — но (противопоставление)
- **it** — он / она / это
  👉 здесь **Go toolchain**
- **does not** — не делает
  👉 вспомогательный глагол
- **place** — размещать, класть
- **the result** — результат

📌
= _но он не размещает результат_

---

### **in the directory where your .go files are located**

- **in** — в
- **the directory** — папка, директория
- **where** — где
  👉 связывает части предложения
- **your** — твои
- **.go files** — go-файлы
- **are located** — находятся
  - **locate** — располагать

📌
= _в папке, где находятся твои .go-файлы_

---

## Предложение 2

### **Instead, Go quickly creates a temporary executable file**

- **Instead** — вместо этого
  👉 очень важное слово!
- **quickly** — быстро
- **creates** — создаёт
- **a** — неопределённый артикль
- **temporary** — временный
- **executable file** — исполняемый файл
  - **execute** — выполнять

📌
= _Вместо этого Go быстро создаёт временный исполняемый файл_

---

### **somewhere “for a short time”**

- **somewhere** — где-то
- **for** — на (период времени)
- **a short time** — короткое время

📌
= _где-то на короткое время_

---

### **(for example, in the /tmp directory,**

- **for example** — например
- **in** — в
- **/tmp directory** — папка /tmp

---

### **or it may even keep it only in memory)**

- **or** — или
- **it** — Go
- **may** — может
  👉 вероятность
- **even** — даже
- **keep** — держать, хранить
- **it** — его (файл)
- **only** — только
- **in memory** — в памяти (RAM)

📌
= _или он может даже держать его только в памяти_

---

## Предложение 3

### **As soon as the compiled program finishes its execution,**

- **As soon as** — как только
- **the compiled program** — скомпилированная программа
- **finishes** — завершает
- **its** — её
- **execution** — выполнение

📌
= _Как только скомпилированная программа завершает выполнение_

---

### **Go automatically deletes this temporary file.**

- **automatically** — автоматически
- **deletes** — удаляет
- **this** — этот
- **temporary file** — временный файл

---

## Предложение 4

### **In the directory with your .go source files,**

- **In** — в
- **the directory** — папке
- **with** — с
- **your** — твоими
- **source files** — исходниками

---

### **nothing remains except the source files themselves.**

- **nothing** — ничего
- **remains** — остаётся
- **except** — кроме
- **themselves** — сами по себе
  👉 усиление

📌
= _ничего не остаётся, кроме самих исходников_

---

## 💡 Главное, что ты должен вынести

1. **into** — превращение
2. **instead** — вместо этого
3. **as soon as** — как только
4. **may** — может (вероятность)
5. **it / them / themselves** — замена слов, чтобы не повторяться

---
