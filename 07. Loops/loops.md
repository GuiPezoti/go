## 7. Loops

Go simplifica loops drasticamente: existe apenas um tipo de loop - `for`. Mas não se preocupe, esse único `for` é versátil o suficiente para cobrir todos os casos que você precisaria de `while`, `do-while` ou `foreach` em outras linguagens. A sintaxe é limpa, sem parênteses obrigatórios, e extremamente legível.

Loops são fundamentais para processar coleções, repetir operações, e implementar algoritmos. O `range` é particularmente útil para iterar sobre slices, arrays, maps, strings e channels de forma idiomática. Você pode usar `break` para sair de um loop, `continue` para pular para a próxima iteração, e até criar loops infinitos para servidores ou workers que rodam continuamente.

```go
// For clássico
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// While (não existe palavra-chave while)
contador := 0
for contador < 5 {
    fmt.Println(contador)
    contador++
}

// Loop infinito
for {
    // usar break para sair
    if condicao {
        break
    }
}

// Range sobre slice
numeros := []int{1, 2, 3, 4, 5}
for indice, valor := range numeros {
    fmt.Printf("Índice: %d, Valor: %d\n", indice, valor)
}

// Ignorando o índice
for _, valor := range numeros {
    fmt.Println(valor)
}

// Apenas o índice
for i := range numeros {
    fmt.Println(i)
}

// Range sobre map
frutas := map[string]int{"maçã": 5, "banana": 3}
for nome, quantidade := range frutas {
    fmt.Printf("%s: %d\n", nome, quantidade)
}

// Range sobre string (runes)
for i, char := range "Olá" {
    fmt.Printf("Posição %d: %c\n", i, char)
}

// Continue e break
for i := 0; i < 10; i++ {
    if i%2 == 0 {
        continue // pula pares
    }
    if i > 7 {
        break // para no 7
    }
    fmt.Println(i)
}
```