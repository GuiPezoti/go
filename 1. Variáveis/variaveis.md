## 1. Variables (Variáveis)

Variáveis são os blocos fundamentais de qualquer programa. Em Go, você tem várias formas de declarar variáveis, desde a forma mais explícita até a sintaxe curta e prática. Go é uma linguagem estaticamente tipada, o que significa que o tipo de cada variável é conhecido em tempo de compilação, mas o compilador pode inferir tipos automaticamente para você.

Uma das grandes vantagens do Go é sua sintaxe limpa e direta. A declaração curta com `:=` é muito usada dentro de funções, enquanto `var` é útil quando você precisa declarar variáveis no escopo de pacote ou quando quer inicializar com o valor zero do tipo. Constantes são imutáveis e calculadas em tempo de compilação, sendo perfeitas para valores que nunca mudam como configurações, URLs de API, ou valores matemáticos.

```go
// Declaração explícita
var nome string = "João"
var idade int = 25

// Declaração com inferência de tipo
var cidade = "São Paulo"

// Declaração curta (apenas dentro de funções)
pais := "Brasil"

// Múltiplas variáveis
var x, y int = 10, 20
a, b := "Go", "Lang"

// Constantes
const PI = 3.14159
const (
    StatusOK = 200
    StatusNotFound = 404
)

// Tipos básicos
var (
    inteiro    int     = 42
    flutuante  float64 = 3.14
    texto      string  = "Hello"
    booleano   bool    = true
)
```