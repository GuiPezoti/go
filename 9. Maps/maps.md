## 9. Maps

Maps são a implementação Go de tabelas hash - estruturas de dados que armazenam pares chave-valor com acesso extremamente rápido. São como dicionários em Python ou objetos em JavaScript. Maps não têm ordem garantida (diferente de slices), mas oferecem busca, inserção e remoção em tempo O(1) na maioria dos casos.

Use maps sempre que precisar associar chaves a valores: cache de dados, contadores, índices, configurações, etc. São perfeitos para casos onde você precisa verificar rapidamente se algo existe ou recuperar valores por identificadores únicos. Maps são tipos de referência, então quando você passa um map para uma função, modificações afetam o original. Lembre-se sempre de verificar se uma chave existe antes de usá-la para evitar valores zero inesperados.

```go
// Criando maps
var m map[string]int                    // map nil (não pode adicionar)
m1 := make(map[string]int)              // map vazio
m2 := map[string]int{                   // literal
    "idade":  25,
    "altura": 180,
}

// Adicionando/atualizando
m1["chave"] = 100
m1["outra"] = 200

// Lendo valores
valor := m1["chave"]
valor, existe := m1["chave"]  // safe read

if v, ok := m1["chave"]; ok {
    fmt.Println("Valor encontrado:", v)
} else {
    fmt.Println("Chave não existe")
}

// Deletando
delete(m1, "chave")

// Iterando
for chave, valor := range m2 {
    fmt.Printf("%s: %d\n", chave, valor)
}

// Map de structs
usuarios := map[string]Pessoa{
    "joao": {Nome: "João", Idade: 30},
    "ana":  {Nome: "Ana", Idade: 25},
}

// Map de slices
tags := map[string][]string{
    "go":     {"programação", "backend"},
    "python": {"ia", "data science"},
}

// Tamanho do map
tamanho := len(m2)
```