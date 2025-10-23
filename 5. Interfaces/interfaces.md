## 5. Interfaces

Interfaces são uma das características mais poderosas e únicas do Go. Elas definem comportamentos através de conjuntos de métodos, mas diferente de outras linguagens, a implementação é implícita - você não precisa declarar que está implementando uma interface. Se um tipo tem todos os métodos de uma interface, ele automaticamente satisfaz aquela interface. Isso promove um design desacoplado e flexível.

Interfaces são fundamentais para criar código testável, extensível e seguir princípios SOLID. Você pode usar interfaces pequenas (como `io.Reader` e `io.Writer`) para criar componentes reutilizáveis que funcionam com qualquer implementação. A interface vazia `interface{}` (ou `any` em versões recentes) aceita qualquer tipo, sendo útil para funções genéricas. Type assertions e type switches permitem trabalhar com tipos concretos quando necessário.

```go
// Definindo interface
type Forma interface {
    Area() float64
    Perimetro() float64
}

// Implementando interface (implícito)
type Retangulo struct {
    Largura, Altura float64
}

func (r Retangulo) Area() float64 {
    return r.Largura * r.Altura
}

func (r Retangulo) Perimetro() float64 {
    return 2 * (r.Largura + r.Altura)
}

type Circulo struct {
    Raio float64
}

func (c Circulo) Area() float64 {
    return math.Pi * c.Raio * c.Raio
}

func (c Circulo) Perimetro() float64 {
    return 2 * math.Pi * c.Raio
}

// Usando interface
func imprimirInfo(f Forma) {
    fmt.Printf("Área: %.2f, Perímetro: %.2f\n", f.Area(), f.Perimetro())
}

// Interface vazia (aceita qualquer tipo)
func imprimirQualquerCoisa(v interface{}) {
    fmt.Println(v)
}

// Type assertion
var i interface{} = "hello"
s := i.(string)        // panic se não for string
s, ok := i.(string)    // safe, ok indica sucesso

// Type switch
func tipoDeValor(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Inteiro: %d\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    default:
        fmt.Printf("Tipo desconhecido: %T\n", v)
    }
}
```