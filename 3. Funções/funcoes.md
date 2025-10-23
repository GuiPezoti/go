## 3. Functions (Funções)

Funções são cidadãs de primeira classe em Go. Elas podem retornar múltiplos valores (uma característica que torna o tratamento de erros muito mais elegante), aceitar um número variável de argumentos, e até ter retornos nomeados que tornam o código mais documentado. Go não tem sobrecarga de funções, mas a simplicidade resultante torna o código mais previsível.

Funções são usadas para organizar código reutilizável e promover a modularidade. A convenção de retornar erro como último valor é amplamente adotada na comunidade Go. Funções variádicas são perfeitas para criar APIs flexíveis (como `fmt.Println`), e retornos nomeados podem tornar funções complexas mais legíveis ao deixar explícito o que está sendo retornado.

```go
// Função básica
func saudacao(nome string) {
    fmt.Println("Olá,", nome)
}

// Função com retorno
func soma(a, b int) int {
    return a + b
}

// Múltiplos retornos
func dividir(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("divisão por zero")
    }
    return a / b, nil
}

// Retornos nomeados
func retangulo(largura, altura float64) (area, perimetro float64) {
    area = largura * altura
    perimetro = 2 * (largura + altura)
    return // retorna automaticamente area e perimetro
}

// Funções variádicas
func somaVarios(numeros ...int) int {
    total := 0
    for _, num := range numeros {
        total += num
    }
    return total
}

// Uso: somaVarios(1, 2, 3, 4, 5)
```