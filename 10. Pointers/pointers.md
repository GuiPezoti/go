## 10. Pointers (Ponteiros)

Ponteiros armazenam endereços de memória em vez de valores diretos. Em Go, ponteiros são mais seguros que em C/C++ (não há aritmética de ponteiros), mas igualmente poderosos para otimizar performance e permitir modificações de dados. O operador `&` obtém o endereço de uma variável, e `*` acessa/modifica o valor apontado.

Ponteiros são essenciais em três cenários principais: quando você precisa modificar uma variável dentro de uma função (passagem por referência), quando quer evitar copiar structs grandes (performance), e quando precisa representar a ausência de valor (nil). Go automaticamente derreferencia ponteiros em alguns contextos (como ao acessar campos de structs), tornando o código mais limpo. Use ponteiros conscientemente - eles são poderosos, mas valor semântico deve guiar sua decisão, não apenas performance.

```go
// Declarando ponteiros
var p *int        // ponteiro para int
x := 42
p = &x            // & obtém o endereço

// Dereferenciando
fmt.Println(*p)   // * acessa o valor
*p = 21           // modifica o valor original

// Ponteiros em funções
func incrementar(n *int) {
    *n++  // modifica o valor original
}

num := 10
incrementar(&num)
fmt.Println(num)  // 11

// Ponteiros com structs
type Carro struct {
    Marca string
    Ano   int
}

func atualizarAno(c *Carro, novoAno int) {
    c.Ano = novoAno  // Go automaticamente derreferencia
}

carro := Carro{Marca: "Toyota", Ano: 2020}
atualizarAno(&carro, 2021)

// new() aloca memória
p1 := new(int)    // retorna *int
*p1 = 100

// Ponteiro vs Valor
// Valor: cópia independente
// Ponteiro: referência ao original

// Quando usar ponteiros:
// 1. Modificar o valor original
// 2. Evitar cópia de structs grandes
// 3. Representar ausência de valor (nil)

// Ponteiros podem ser nil
var ptr *int
if ptr == nil {
    fmt.Println("Ponteiro nulo")
}
```