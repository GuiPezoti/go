## 8. Slices

Slices são a estrutura de dados mais versátil e usada em Go para trabalhar com sequências de elementos. Diferente de arrays (que têm tamanho fixo), slices são dinâmicos e podem crescer ou diminuir conforme necessário. Internamente, um slice é uma referência a um array subjacente, junto com informações de comprimento e capacidade.

Slices são perfeitos para listas, coleções, pilhas, filas e qualquer estrutura que precise crescer dinamicamente. A função `append` adiciona elementos e automaticamente realoca memória quando necessário. Você pode criar "fatias" de slices existentes, copiar dados entre slices, e até criar matrizes multidimensionais. Entender a diferença entre comprimento (elementos atuais) e capacidade (espaço alocado) é fundamental para otimizar performance.

```go
// Criando slices
var s []int                    // slice nil
s1 := []int{1, 2, 3}          // literal
s2 := make([]int, 5)          // comprimento 5, valores zero
s3 := make([]int, 3, 10)      // comprimento 3, capacidade 10

// Funções importantes
len(s1)  // comprimento
cap(s1)  // capacidade

// Append (adiciona elementos)
s1 = append(s1, 4)
s1 = append(s1, 5, 6, 7)
s1 = append(s1, s2...)  // spread de outro slice

// Slice de slice
numeros := []int{0, 1, 2, 3, 4, 5}
sub := numeros[1:4]     // [1, 2, 3]
sub2 := numeros[:3]     // [0, 1, 2]
sub3 := numeros[2:]     // [2, 3, 4, 5]
copia := numeros[:]     // cópia completa (shallow)

// Copy (cópia profunda)
origem := []int{1, 2, 3}
destino := make([]int, len(origem))
copy(destino, origem)

// Removendo elemento
func remover(slice []int, i int) []int {
    return append(slice[:i], slice[i+1:]...)
}

// Slice de structs
type Produto struct {
    Nome  string
    Preco float64
}

produtos := []Produto{
    {Nome: "Notebook", Preco: 3000},
    {Nome: "Mouse", Preco: 50},
}

// Slice bidimensional
matriz := [][]int{
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9},
}
```