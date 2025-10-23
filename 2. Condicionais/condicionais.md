## 2. Conditionals (Condicionais)

As estruturas condicionais permitem que seu programa tome decisões baseadas em condições. Go simplifica bastante essa parte: não há parênteses obrigatórios ao redor das condições (diferente de C/Java), mas as chaves são sempre obrigatórias, mesmo para blocos de uma linha. Isso torna o código mais consistente e evita bugs comuns.

O `switch` em Go é particularmente poderoso: ele não requer `break` (não há fall-through por padrão), pode trabalhar sem uma expressão (funcionando como um if-else-if mais limpo), e aceita múltiplos valores por case. A capacidade de inicializar uma variável dentro do `if` é extremamente útil para trabalhar com funções que retornam múltiplos valores, como ao tratar erros.

```go
// If básico
if x > 10 {
    fmt.Println("Maior que 10")
}

// If-else
if idade >= 18 {
    fmt.Println("Maior de idade")
} else {
    fmt.Println("Menor de idade")
}

// If com inicialização
if num := calcular(); num > 0 {
    fmt.Println("Positivo:", num)
} else {
    fmt.Println("Negativo ou zero:", num)
}

// Switch
switch dia {
case "segunda", "terça":
    fmt.Println("Início da semana")
case "sexta":
    fmt.Println("Sextou!")
default:
    fmt.Println("Outro dia")
}

// Switch sem condição (substitui if-else-if)
switch {
case nota >= 90:
    fmt.Println("A")
case nota >= 80:
    fmt.Println("B")
default:
    fmt.Println("C")
}
```