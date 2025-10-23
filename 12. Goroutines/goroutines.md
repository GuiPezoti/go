## 12. Goroutines

Goroutines são threads leves gerenciadas pelo runtime do Go, e são uma das características mais poderosas e distintivas da linguagem. Diferente de threads do sistema operacional que são pesadas e caras, goroutines são extremamente leves - você pode facilmente ter milhares ou até milhões delas rodando simultaneamente. Para criar uma goroutine, você simplesmente adiciona a palavra-chave `go` antes de uma chamada de função.

O scheduler do Go multiplexa automaticamente goroutines em threads do sistema operacional, distribuindo o trabalho de forma eficiente. Isso permite escrever código concorrente de forma natural e intuitiva, sem se preocupar com a complexidade de gerenciar threads manualmente. Goroutines são perfeitas para operações I/O concorrentes (requisições HTTP, queries de banco de dados), processamento paralelo de dados, servidores que precisam lidar com múltiplas conexões simultâneas, e qualquer tarefa que possa se beneficiar de execução assíncrona.

```go
// Goroutine básica
func tarefa() {
    fmt.Println("Executando tarefa...")
}

// Iniciando uma goroutine
go tarefa()

// Goroutine com função anônima
go func() {
    fmt.Println("Goroutine anônima")
}()

// Goroutine com parâmetros
go func(nome string) {
    fmt.Printf("Olá, %s!\n", nome)
}("João")

// Múltiplas goroutines
for i := 1; i <= 5; i++ {
    go func(id int) {
        fmt.Printf("Goroutine %d executando\n", id)
    }(i) // importante: passar i como argumento
}

// Exemplo: Processamento concorrente
func processar(url string) {
    resp, err := http.Get(url)
    if err != nil {
        fmt.Printf("Erro ao acessar %s: %v\n", url, err)
        return
    }
    defer resp.Body.Close()
    fmt.Printf("%s - Status: %d\n", url, resp.StatusCode)
}

urls := []string{
    "https://google.com",
    "https://github.com",
    "https://golang.org",
}

for _, url := range urls {
    go processar(url) // cada requisição em paralelo
}

// IMPORTANTE: Goroutines não bloqueiam
// O programa pode terminar antes das goroutines completarem
// Use WaitGroup ou channels para sincronizar

// Exemplo com WaitGroup
import "sync"

var wg sync.WaitGroup

for i := 1; i <= 3; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        time.Sleep(time.Second)
        fmt.Printf("Goroutine %d finalizada\n", id)
    }(i)
}

wg.Wait() // Espera todas as goroutines terminarem
fmt.Println("Todas as goroutines finalizadas")

// Goroutines com retorno via channel
func calcular(n int, resultado chan<- int) {
    soma := 0
    for i := 1; i <= n; i++ {
        soma += i
    }
    resultado <- soma
}

ch := make(chan int)
go calcular(100, ch)
resultado := <-ch
fmt.Println("Resultado:", resultado)

// Pattern: Fan-out (distribuir trabalho)
func worker(id int, jobs <-chan int, wg *sync.WaitGroup) {
    defer wg.Done()
    for job := range jobs {
        fmt.Printf("Worker %d processando job %d\n", id, job)
        time.Sleep(time.Millisecond * 100)
    }
}

jobs := make(chan int, 10)
var wg sync.WaitGroup

// Inicia 3 workers
for w := 1; w <= 3; w++ {
    wg.Add(1)
    go worker(w, jobs, &wg)
}

// Envia 10 jobs
for j := 1; j <= 10; j++ {
    jobs <- j
}
close(jobs)

wg.Wait()

// Cuidados importantes:
// 1. Goroutines não são gratuitas (consomem ~2KB de stack inicial)
// 2. Sempre garanta que goroutines possam terminar (evite goroutine leaks)
// 3. Use channels ou mutexes para comunicação segura entre goroutines
// 4. Evite compartilhar memória sem sincronização (race conditions)
```