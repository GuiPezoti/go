## 15. Generics (Go 1.18+)

Generics permitem escrever funções e tipos que funcionam com qualquer tipo, eliminando duplicação de código e mantendo segurança de tipos. Adicionados no Go 1.18, generics finalmente resolvem o problema de ter que usar `interface{}` (perdendo type safety) ou duplicar código para cada tipo. Você define parâmetros de tipo entre colchetes `[T any]` e pode restringi-los usando constraints.

Generics são perfeitos para estruturas de dados reutilizáveis (listas, pilhas, filas, árvores), algoritmos que funcionam em múltiplos tipos (ordenação, busca, filtros), e utilitários genéricos (max, min, map, filter). As constraints permitem especificar quais operações devem estar disponíveis - por exemplo, `comparable` para tipos que podem ser comparados com `==`, ou interfaces customizadas para tipos que implementam métodos específicos. Isso torna o código mais seguro e expressivo do que usar `interface{}`.

```go
// Função genérica simples
func Primeiro[T any](slice []T) T {
    return slice[0]
}

// Uso
numeros := []int{1, 2, 3}
strings := []string{"a", "b", "c"}
fmt.Println(Primeiro(numeros))  // 1
fmt.Println(Primeiro(strings))  // "a"

// Constraints (restrições)
func Soma[T int | float64](a, b T) T {
    return a + b
}

// Interface como constraint
type Numero interface {
    int | int64 | float64
}

func Maximo[T Numero](a, b T) T {
    if a > b {
        return a
    }
    return b
}

// Constraint com métodos
type Ordenavel interface {
    comparable
}

func Contem[T comparable](slice []T, valor T) bool {
    for _, item := range slice {
        if item == valor {
            return true
        }
    }
    return false
}

// Tipos genéricos
type Pilha[T any] struct {
    items []T
}

func (p *Pilha[T]) Push(item T) {
    p.items = append(p.items, item)
}

func (p *Pilha[T]) Pop() (T, bool) {
    if len(p.items) == 0 {
        var zero T
        return zero, false
    }
    item := p.items[len(p.items)-1]
    p.items = p.items[:len(p.items)-1]
    return item, true
}

// Uso
pilhaInt := &Pilha[int]{}
pilhaInt.Push(1)
pilhaInt.Push(2)

pilhaString := &Pilha[string]{}
pilhaString.Push("hello")

// Map genérico
func Map[T any, U any](slice []T, fn func(T) U) []U {
    resultado := make([]U, len(slice))
    for i, v := range slice {
        resultado[i] = fn(v)
    }
    return resultado
}

// Uso
nums := []int{1, 2, 3, 4}
dobrados := Map(nums, func(n int) int { return n * 2 })

// Filter genérico
func Filter[T any](slice []T, fn func(T) bool) []T {
    resultado := []T{}
    for _, v := range slice {
        if fn(v) {
            resultado = append(resultado, v)
        }
    }
    return resultado
}

// Uso
pares := Filter(nums, func(n int) bool { return n%2 == 0 })

// Reduce genérico
func Reduce[T any, U any](slice []T, inicial U, fn func(U, T) U) U {
    resultado := inicial
    for _, v := range slice {
        resultado = fn(resultado, v)
    }
    return resultado
}

// Uso: soma de todos os elementos
soma := Reduce(nums, 0, func(acc, n int) int { return acc + n })

// Constraint complexa
type Numerico interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64
}

func Soma[T Numerico](valores ...T) T {
    var total T
    for _, v := range valores {
        total += v
    }
    return total
}

// Lista genérica encadeada
type Node[T any] struct {
    valor T
    prox  *Node[T]
}

type Lista[T any] struct {
    head *Node[T]
}

func (l *Lista[T]) Add(valor T) {
    novoNode := &Node[T]{valor: valor, prox: l.head}
    l.head = novoNode
}

func (l *Lista[T]) ForEach(fn func(T)) {
    atual := l.head
    for atual != nil {
        fn(atual.valor)
        atual = atual.prox
    }
}

// Par genérico (tuple)
type Par[T, U any] struct {
    Primeiro T
    Segundo  U
}

func NovoPar[T, U any](a T, b U) Par[T, U] {
    return Par[T, U]{Primeiro: a, Segundo: b}
}

// Uso
coordenada := NovoPar(10, 20)
pessoa := NovoPar("João", 30)

// Result type (similar ao Result do Rust)
type Result[T any] struct {
    valor T
    err   error
}

func Ok[T any](valor T) Result[T] {
    return Result[T]{valor: valor}
}

func Err[T any](err error) Result[T] {
    return Result[T]{err: err}
}

func (r Result[T]) IsOk() bool {
    return r.err == nil
}

func (r Result[T]) Unwrap() (T, error) {
    return r.valor, r.err
}

// Cache genérico thread-safe
type Cache[K comparable, V any] struct {
    mu    sync.RWMutex
    items map[K]V
}

func NovoCache[K comparable, V any]() *Cache[K, V] {
    return &Cache[K, V]{
        items: make(map[K]V),
    }
}

func (c *Cache[K, V]) Set(key K, value V) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.items[key] = value
}

func (c *Cache[K, V]) Get(key K) (V, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    value, ok := c.items[key]
    return value, ok
}
```