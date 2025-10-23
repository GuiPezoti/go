## 13. Channels (Canais)

Channels são o mecanismo de comunicação entre goroutines, permitindo que você envie e receba valores de forma sincronizada. São o coração da programação concorrente em Go, seguindo o princípio "não comunique compartilhando memória, compartilhe memória comunicando". Channels podem ser buffered (com capacidade) ou unbuffered (bloqueiam até o receptor estar pronto).

Channels transformam problemas complexos de concorrência em código legível e seguro. Use-os para coordenar trabalho entre goroutines, implementar pipelines de processamento, criar worker pools, ou controlar o fluxo de dados em sistemas concorrentes. A instrução `select` permite trabalhar com múltiplos channels simultaneamente, incluindo timeouts. Sempre feche channels quando terminar de enviar dados para sinalizar aos receptores que não há mais dados chegando.

```go
// Criando channels
ch := make(chan int)        // unbuffered
ch2 := make(chan int, 5)    // buffered (capacidade 5)

// Enviando e recebendo
ch <- 42          // envia
valor := <-ch     // recebe

// Goroutines
go func() {
    ch <- 100
}()

resultado := <-ch
fmt.Println(resultado)

// Channel com buffering
buffered := make(chan string, 2)
buffered <- "primeiro"
buffered <- "segundo"
// não bloqueia até aqui
fmt.Println(<-buffered)
fmt.Println(<-buffered)

// Fechando channels
close(ch)
valor, aberto := <-ch  // aberto = false se fechado

// Range sobre channel
jobs := make(chan int, 5)
go func() {
    for i := 1; i <= 5; i++ {
        jobs <- i
    }
    close(jobs)
}()

for job := range jobs {
    fmt.Println("Processando:", job)
}

// Select (múltiplos channels)
ch1 := make(chan string)
ch2 := make(chan string)

go func() { ch1 <- "primeiro" }()
go func() { ch2 <- "segundo" }()

select {
case msg1 := <-ch1:
    fmt.Println(msg1)
case msg2 := <-ch2:
    fmt.Println(msg2)
case <-time.After(1 * time.Second):
    fmt.Println("timeout")
}

// Pattern: Worker Pool
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Printf("Worker %d processando job %d\n", id, j)
        results <- j * 2
    }
}

jobs := make(chan int, 100)
results := make(chan int, 100)

// Inicia workers
for w := 1; w <= 3; w++ {
    go worker(w, jobs, results)
}

// Envia jobs
for j := 1; j <= 9; j++ {
    jobs <- j
}
close(jobs)

// Coleta resultados
for a := 1; a <= 9; a++ {
    <-results
}
```
