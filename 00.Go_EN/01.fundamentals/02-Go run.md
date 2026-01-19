## Introduction: Why It’s Important to Understand the Difference Between `go run` and `go build`

- **What is Go?** Go (or Golang) is a programming language and a set of tools that let you write a program (code) and turn it into an executable program (a binary) that can be run on a computer.

- **What does the `go run` command do?** It compiles your code on the fly and immediately runs it, leaving no file behind that you could run again and again.

- **What does the `go build` command do?** It compiles your code and saves the result as a permanent program (an application file) that you can run as many times as you like.

If these terms (compilation, binary, HDD/SSD, RAM) don’t mean anything to you yet — don’t worry. We’ll break everything down step by step in what follows.

---

## 1. Why do people even ask: “Why use `go run` if `go build` exists?”

> **The author says:**
> “Why do we use `go run` when we could simply run `go build`? It’s simple. `go build` creates a ‘permanent’ program (a binary), whereas `go run` creates a ‘temporary’ program, immediately runs it, and doesn’t leave anything on the disk.
