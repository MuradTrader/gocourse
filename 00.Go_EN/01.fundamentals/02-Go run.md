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

</details>

---

### **Why are “persistent” and “temporary” important?**

**Persistent binary (`go build`):**

- You can run it as many times as you want without thinking about rebuilding it.
- It can be copied (for example, to another server or to a friend), stored in multiple copies, and digitally signed (for security).
- This is convenient when you have already “finished” the program and want it to exist as a standalone file.

**Temporary binary (`go run`):**

- It is convenient during development, when you frequently change the code and want to test it immediately.
- It does not leave any “clutter” in the project directory: each time it is “build, run, delete.”
- If you are running a small utility once or twice, there is no need to think about where to place the program file — Go cleans everything up automatically.

<details>
  <summary><b>📖 Подробное разборное объяснение (кликни)</b></summary>

# 1️⃣ Заголовок — разбор ПО СЛОВАМ и ПО ПОРЯДКУ

### **Why are “persistent” and “temporary” important?**

❗ В английском **строгий порядок слов**. Смотрим слева направо.

### **Why**

- перевод: **почему**
- всегда ставится **в начале вопроса**
- означает: _по какой причине?_

### **are**

- это глагол **to be** (быть)
- **are**, потому что **два слова**:
  - persistent
  - temporary
    👉 множественное число

### **persistent**

- перевод: **постоянный**
- в IT: _то, что не исчезает само_

### **and**

- перевод: **и**
- соединяет два слова

### **temporary**

- перевод: **временный**
- противоположность persistent

### **important**

- перевод: **важны**
- отвечает на вопрос: _почему это имеет значение?_

📌 **Дословно по порядку:**

> **Why** — почему
> **are** — являются
> **persistent and temporary** — постоянный и временный
> **important** — важными

👉 **Нормальный русский:**

**Почему постоянное и временное важны?**

---

# 2️⃣ Persistent binary (go build)

### **Persistent**

- постоянный
- файл **остаётся** после сборки

### **binary**

- бинарный файл
- готовая программа

📌 **Persistent binary**
= _постоянный исполняемый файл_

---

# 3️⃣ Предложение №1 — ОЧЕНЬ ВАЖНО

### **You can run it as many times as you want**

Разбираем **по кирпичикам**.

### **You**

- перевод: **ты / вы**
- в документации = _любой пользователь_

### **can**

- перевод: **можешь**
- показывает **возможность**

### **run**

- перевод: **запускать**

### **it**

- перевод: **его**
- заменяет слово **binary**

📌 **You can run it**
= _Ты можешь запускать его_

---

### **as many times as you want**

❗ Это **готовая конструкция**, её не переводят слово в слово.

- **as many times as** = **сколько угодно раз**
- **you want** = **ты хочешь**

📌
**You can run it as many times as you want**
============================================

**Ты можешь запускать его сколько угодно раз**

---

# 4️⃣ without thinking about rebuilding it

### **without**

- перевод: **без**

### **thinking**

- перевод: **думая**

### **about**

- перевод: **о**

### **rebuilding**

- rebuild = пересобирать
- rebuilding = пересборка

### **it**

- его (бинарник)

📌
**without thinking about rebuilding it**
========================================

**не думая о его пересборке**

---

# 5️⃣ Следующее предложение

### **It can be copied**

### **It**

- это бинарник

### **can be**

- **может быть**
- пассивная форма (его делают, а не он сам)

### **copied**

- копируемый

📌
= **Его можно копировать**

---

### **stored in multiple copies**

### **stored**

- хранить

### **in**

- в

### **multiple**

- несколько

### **copies**

- копий

📌
= **хранить в нескольких копиях**

---

### **and digitally signed (for security)**

### **digitally**

- цифровым способом

### **signed**

- подписан

### **for**

- для

### **security**

- безопасности

📌
= **и подписывать цифровой подписью для безопасности**

---

# 6️⃣ This is convenient

### **This**

- это (всё выше)

### **is**

- есть

### **convenient**

- удобно

📌
= **Это удобно**

---

# 7️⃣ when you have already finished the program

### **when**

- когда

### **you have finished**

- Present Perfect
- означает: **уже закончил**

### **already**

- уже

### **the program**

- программу

📌
= **когда ты уже закончил программу**

---

# 8️⃣ and want it to exist as a standalone file

### **want**

- хотеть

### **it**

- её (программу)

### **to exist**

- существовать

### **as**

- как

### **standalone**

- отдельный, самостоятельный

### **file**

- файл

📌
= **и хочешь, чтобы она существовала как отдельный файл**

---

# 9️⃣ Temporary binary (go run)

### **Temporary**

- временный

### **binary**

- бинарник

📌
= **временный бинарник**

---

# 🔟 It is convenient during development

### **It is**

- это есть

### **convenient**

- удобно

### **during**

- во время

### **development**

- разработки

📌
= **Это удобно во время разработки**

---

# 1️⃣1️⃣ when you frequently change the code

### **frequently**

- часто

### **change**

- менять

### **the code**

- код

📌
= **когда ты часто меняешь код**

---

# 1️⃣2️⃣ and want to test it immediately

### **test**

- проверять

### **immediately**

- сразу

📌
= **и хочешь сразу его проверить**

---

# 1️⃣3️⃣ It does not leave any clutter

### **does not**

- не

### **leave**

- оставлять

### **any**

- никакой

### **clutter**

- мусор, хлам

📌
= **Он не оставляет мусор**

---

# 1️⃣4️⃣ Go cleans everything up automatically

### **cleans up**

- убирает

### **everything**

- всё

### **automatically**

- автоматически

📌
= **Go всё убирает автоматически**

---

## 🧠 СУПЕР ВАЖНО (коротко)

- **can** — можешь
- **it** — заменяет слова
- **without / during / when** — логика предложений
- **as many times as** — сколько угодно раз
- **Present Perfect** = уже сделал

---

</details>
