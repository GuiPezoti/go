## 14. Mutexes

Mutexes (mutual exclusion) são primitivas de sincronização que protegem dados compartilhados em ambientes concorrentes. Quando múltiplas goroutines precisam acessar e modificar os mesmos dados, race conditions podem ocorrer. Um mutex garante que apenas uma goroutine pode acessar a seção crítica por vez, travando (`Lock`) antes de acessar e destravando (`Unlock`) depois.

Use mutexes quando channels não são a melhor solução - tipicamente para proteger estado compartilhado como caches, contadores, ou estruturas de dados complexas. `RWMutex` otimiza cenários onde leituras são frequentes e escritas raras, permitindo múltiplos leitores simultâneos. `WaitGroup` coordena a espera de múltiplas goroutines terminarem. Sempre use `defer` com `Unlock()` para garantir que o mutex seja liberado mesmo se houver panic. Evite deadlocks nunca travando múltiplos mutexes em ordem diferente.

```go
import "sync"

// Mutex básico
var (
    contador int
    mutex    sync.Mutex
)

func incrementar() {
    mutex.Lock()
    contador++
    mutex.Unlock()
}

// Defer com unlock (recomendado)
func incrementarSeguro() {
    mutex.Lock()
    defer mutex.Unlock()
    contador++
}

// RWMutex (múltiplos leitores, um escritor)
var (
    cache   = make(map[string]string)
    rwMutex sync.RWMutex
)

func ler(chave string) string {
    rwMutex.RLock()
    defer rwMutex.RUnlock()
    return cache[chave]
}

func escrever(chave, valor string) {
    rwMutex.Lock()
    defer rwMutex.Unlock()
    cache[chave] = valor
}

// WaitGroup (esperar goroutines)
var wg sync.WaitGroup

for i := 1; i <= 5; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        fmt.Printf("Worker %d\n", id)
    }(i)
}

wg.Wait()  // espera todas terminarem

// Once (executa apenas uma vez)
var once sync.Once

func inicializar() {
    once.Do(func() {
        fmt.Println("Inicializado apenas uma vez")
    })
}

// Exemplo completo: contador seguro
type ContadorSeguro struct {
    mu    sync.Mutex
    valor int
}

func (c *ContadorSeguro) Incrementar() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.valor++
}

func (c *ContadorSeguro) Valor() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.valor
}
```