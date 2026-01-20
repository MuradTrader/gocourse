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
