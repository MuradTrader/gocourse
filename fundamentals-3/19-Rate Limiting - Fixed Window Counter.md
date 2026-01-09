**1. Что такое алгоритм Fixed Window (фиксированного окна):**

Fixed Window алгоритм делит время на окна фиксированного размера. Он позволяет фиксированное количество запросов в каждом окне. Если количество запросов превышает лимит в пределах окна, последующие запросы отклоняются до сброса окна.

Этот алгоритм прост в реализации, но может приводить к всплескам трафика на границах окон.

## Структура RateLimiter (построчно)

### 1. Объявление структуры

```go
type RateLimiter struct {
    mu        sync.Mutex
    count     int
    limit     int
    window    time.Duration
    resetTime time.Time
}
```

**sync.Mutex** - это примитив синхронизации для защиты от гонки данных (race condition). Представьте, что несколько горутин одновременно пытаются изменить счетчик запросов. Мьютекс блокирует доступ, чтобы только одна горутина могла работать с данными структуры в один момент времени.

**count int** - счетчик текущих запросов в текущем окне.

**limit int** - максимальное количество разрешенных запросов в одном окне.

**window time.Duration** - длительность временного окна (например, 1 секунда, 2 секунды).

**resetTime time.Time** - время, когда текущее окно закончится и счетчик сбросится.

### 2. Конструктор NewRateLimiter

```go
func NewRateLimiter(limit int, window time.Duration) *RateLimiter {
    return &RateLimiter{
        limit:  limit,
        window: window,
    }
}
```

Когда мы создаем новый RateLimiter, мы устанавливаем только `limit` и `window`. Остальные поля инициализируются нулевыми значениями:

- `mu` - готов к использованию
- `count` - будет 0
- `resetTime` - будет нулевым временем (0001-01-01 00:00:00 UTC)

### 3. Метод Allow() - детальное объяснение

```go
func (rl *RateLimiter) Allow() bool {
	rl.mu.Lock()
	defer rl.mu.Unlock()

	now := time.Now()

	if now.After(rl.resetTime) {
		rl.resetTime = now.Add(rl.window)
		rl.count = 0
	}

	if rl.count < rl.limit {
		rl.count++
		return true
	}

	return false
}
```

**Шаг 1: Блокировка мьютекса**

```go
rl.mu.Lock()
defer rl.mu.Unlock()
```

Когда горутина вызывает `Allow()`, она сначала блокирует мьютекс. Если другая горутина уже заблокировала его, текущая горутина будет ждать.

**defer** гарантирует, что разблокировка произойдет ВСЕГДА в конце функции, даже если произойдет паника (panic).

**Шаг 2: Получение текущего времени**

```go
now := time.Now()
```

Запоминаем точное время вызова метода.

**Шаг 3: Проверка, не пора ли сбросить счетчик**

```go
if now.After(rl.resetTime) {
    rl.resetTime = now.Add(rl.window)
    rl.count = 0
}
```

Рассмотрим это подробно:

**Первое условие**: `now.After(rl.resetTime)`

В начале работы программы `rl.resetTime` равно нулевому времени. Любое текущее время будет после нулевого времени, поэтому условие выполнится.

**Что происходит при выполнении**:

- `rl.resetTime = now.Add(rl.window)` - устанавливаем время сброса в будущее
  Например: если сейчас 10:00:00, а окно 1 секунда, то `resetTime = 10:00:01`
- `rl.count = 0` - сбрасываем счетчик

**Дальнейшая работа**: при следующих вызовах проверяется, не наступило ли уже время сброса.

**Шаг 4: Проверка лимита**

```go
if rl.count < rl.limit {
    rl.count++
    return true
}

return false
```

Если счетчик меньше лимита:

1. Увеличиваем счетчик на 1
2. Возвращаем `true` (запрос разрешен)

Если счетчик достиг лимита:
Возвращаем `false` (запрос отклонен)

## Разбор конкретных примеров

### Пример 1: Лимит 5 запросов в секунду

```go
func main() {
	limiter := NewRateLimiter(5, time.Second)

	for i := 0; i < 10; i++ {
		if limiter.Allow() {
			fmt.Println("request allowed")
		} else {
			fmt.Println("request denied")
		}
		time.Sleep(200 * time.Millisecond)
	}
}

// result:

/*
request allowed
request allowed
request allowed
request allowed
request allowed
request allowed
request allowed
request allowed
request allowed
request allowed
*/
```

**Что происходит в цикле**:

| Время (мс) | Счетчик | resetTime | Результат |
| ---------- | ------- | --------- | --------- |
| 0          | 0 → 1   | 1000      | allowed   |
| 200        | 1 → 2   | 1000      | allowed   |
| 400        | 2 → 3   | 1000      | allowed   |
| 600        | 3 → 4   | 1000      | allowed   |
| 800        | 4 → 5   | 1000      | allowed   |
| 1000       | 5       | 1000      | denied    |

**Стоп!** Давайте разберем момент времени 1000мс:

Когда наступает 1000мс:

1. `now.After(rl.resetTime)` - 1000мс НЕ после 1000мс (они равны)
2. Поэтому сброса не происходит
3. `rl.count = 5` (лимит достигнут)
4. Возвращается `false` (denied)

Но в выводе мы видим все "allowed"! Почему?

**Давайте пересчитаем точнее**:

Каждый запрос происходит с задержкой 200мс:

- Запрос 1: 0мс
- Запрос 2: 200мс
- Запрос 3: 400мс
- Запрос 4: 600мс
- Запрос 5: 800мс
- **Сброс окна**: 1000мс
- Запрос 6: 1000мс (после сброса!)
- Запрос 7: 1200мс
- Запрос 8: 1400мс
- Запрос 9: 1600мс
- Запрос 10: 1800мс

**Ключевой момент**: На 1000мс окно сбрасывается ДО обработки запроса, поэтому:

1. На 1000мс: `now.After(rl.resetTime)` = true (1000мс после 1000мс? НЕТ, они равны!)
   Внимание: `After()` возвращает true только если время ПОСЛЕ указанного, не равное!

Правильный расчет:

- На 0мс: resetTime устанавливается на 1000мс
- На 1000мс: now = 1000мс, resetTime = 1000мс
- `now.After(1000мс)` = false (1000мс не после 1000мс)
- Значит, сброса не происходит до 1001мс!

Но автор говорит, что в первом примере все запросы разрешены. Давайте проверим:

Всего 10 запросов за 2 секунды (200мс × 10 = 2000мс)
Лимит: 5 запросов в секунду

Распределение по секундам:

- Секунда 1 (0-999мс): запросы в 0, 200, 400, 600, 800мс - 5 запросов ✓
- Секунда 2 (1000-1999мс): запросы в 1000, 1200, 1400, 1600, 1800мс - 5 запросов ✓

Все запросы в пределах лимита каждой секунды!

### Пример 2: Лимит 5 запросов в 2 секунды

```go
func main() {
	limiter := NewRateLimiter(5, 2*time.Second)

	for i := 0; i < 10; i++ {
		if limiter.Allow() {
			fmt.Println("request allowed")
		} else {
			fmt.Println("request denied")
		}
		time.Sleep(200 * time.Millisecond)
	}
}

// result:

/*
request allowed
request allowed
request allowed
request allowed
request allowed
request denied
request denied
request denied
request denied
request denied
*/
```

Окно: 2 секунды (2000мс)
Запросы каждые 200мс

| Время (мс) | В окне (0-1999мс) | Счетчик | Результат |
| ---------- | ----------------- | ------- | --------- |
| 0          | ✓                 | 1       | allowed   |
| 200        | ✓                 | 2       | allowed   |
| 400        | ✓                 | 3       | allowed   |
| 600        | ✓                 | 4       | allowed   |
| 800        | ✓                 | 5       | allowed   |
| 1000       | ✓                 | 5       | denied    |
| 1200       | ✓                 | 5       | denied    |
| 1400       | ✓                 | 5       | denied    |
| 1600       | ✓                 | 5       | denied    |
| 1800       | ✓                 | 5       | denied    |

В одном окне (2 секунды) помещается 10 запросов (каждые 200мс), но лимит только 5. Поэтому первые 5 разрешены, остальные 5 отклонены.

### Пример 3: Лимит 3 запроса в секунду

```go
func main() {
	limiter := NewRateLimiter(3, 1*time.Second)

	for i := 0; i < 10; i++ {
		if limiter.Allow() {
			fmt.Println("request allowed")
		} else {
			fmt.Println("request denied")
		}
		time.Sleep(200 * time.Millisecond)
	}
}

// result:

/*
request allowed
request allowed
request allowed
request denied
request denied
request allowed
request allowed
request allowed
request denied
request denied
*/
```

Запросы каждые 200мс:

| Время (мс) | Окно | Счетчик в окне | Результат |
| ---------- | ---- | -------------- | --------- |
| 0          | 1    | 1              | allowed   |
| 200        | 1    | 2              | allowed   |
| 400        | 1    | 3              | allowed   |
| 600        | 1    | 3 (лимит!)     | denied    |
| 800        | 1    | 3 (лимит!)     | denied    |
| 1000       | 2    | 1              | allowed   |
| 1200       | 2    | 2              | allowed   |
| 1400       | 2    | 3              | allowed   |
| 1600       | 2    | 3 (лимит!)     | denied    |
| 1800       | 2    | 3 (лимит!)     | denied    |

### Пример 4: Горутины с задержкой

```go
func main() {
	limiter := NewRateLimiter(3, 1*time.Second)

	for range 10 {
		go func() {
			if limiter.Allow() {
				fmt.Println("request allowed")
			} else {
				fmt.Println("request denied")
			}
		}()
		time.Sleep(200 * time.Millisecond)
	}
}

// result:

/*
request allowed
request allowed
request allowed
request denied
request denied
request allowed
request allowed
request allowed
request denied
request denied
*/
```

Код аналогичен примеру 3, но каждый запрос выполняется в отдельной горутине. Задержка 200мс между созданием горутин дает тот же результат, что и синхронные вызовы.

### Пример 5: Все горутины запускаются одновременно

```go
func main() {
	limiter := NewRateLimiter(3, 1*time.Second)

	var wg sync.WaitGroup

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			if limiter.Allow() {
				fmt.Println("request allowed")
			} else {
				fmt.Println("request denied")
			}
		}()
	}

	wg.Wait()
}

// result:

/*
request allowed
request allowed
request denied
request allowed
request denied
request denied
request denied
request denied
request denied
request denied
*/
```

**Ключевое отличие**: все 10 горутин запускаются практически одновременно.

Что происходит:

1. 10 горутин одновременно пытаются вызвать `Allow()`
2. Мьютекс обеспечивает последовательный доступ
3. Первая горутина:
   - Блокирует мьютекс
   - Сбрасывает счетчик (так как resetTime в прошлом)
   - Увеличивает счетчик с 0 до 1
   - Разблокирует мьютекс
   - Возвращает true
4. Вторая горутина:
   - Блокирует мьютекс
   - Проверяет время (окно еще не сбросилось)
   - Увеличивает счетчик с 1 до 2
   - Возвращает true
5. Третья горутина:
   - Аналогично, увеличивает счетчик с 2 до 3
   - Возвращает true
6. Все остальные горутины:
   - Видят, что счетчик = 3 (равно лимиту)
   - Возвращают false

**Почему порядок вывода может быть разным?**
Горутины выполняются конкурентно. Хотя мьютекс обеспечивает последовательный доступ к `Allow()`, вывод в консоль (`fmt.Println`) не синхронизирован. Поэтому сообщения могут выводиться в произвольном порядке, хотя логика ограничения работает правильно.

## Важные технические детали

### 1. О времени сброса

При создании RateLimiter, `resetTime` равно нулевому времени. При первом вызове `Allow()`:

```go
if now.After(rl.resetTime) {  // Всегда true для нулевого времени
    rl.resetTime = now.Add(rl.window)
    rl.count = 0
}
```

### 2. Граничные случаи

Если запрос приходит точно в момент `resetTime`, то `now.After(rl.resetTime)` вернет false, и сброс не произойдет. Это важно для точного соблюдения временных окон.

### 3. Потокобезопасность

Без `sync.Mutex` при конкурентном доступе могло бы произойти:

- Две горутины одновременно читают `count = 2`
- Обе увеличивают его до 3
- Обе разрешают запрос
- Итог: `count = 3`, но разрешено 4 запроса (нарушение лимита)

Мьютекс предотвращает это, заставляя горутины ждать своей очереди.

### 4. Алгоритм Fixed Window vs Token Bucket

**Fixed Window** (этот код):

- Простая реализация
- Может вызывать всплески на границах окон
- Пример: если лимит 1000 запросов в час, и все запросы делаются в последнюю минуту часа, а затем в первую минуту следующего часа

**Token Bucket** (предыдущая лекция):

- Более плавное распределение
- Позволяет накапливать "токены" для кратковременных всплесков
- Сложнее в реализации

Это максимально подробное объяснение каждого аспекта кода, как его объясняет автор курса.
