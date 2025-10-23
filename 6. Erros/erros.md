## 6. Errors (Erros)

Go trata erros de forma explícita e pragmática. Em vez de exceções que podem ser lançadas de qualquer lugar, Go usa valores de erro retornados explicitamente. Isso força você a pensar sobre o que pode dar errado e como lidar com isso em cada ponto do código. Um erro é apenas um valor que implementa a interface `error`, que tem um único método: `Error() string`.

Essa abordagem torna o fluxo de erro visível e o código mais previsível. Você verá muitas funções retornando `(resultado, error)` - checar `if err != nil` se torna segunda natureza. Você pode criar erros simples com `errors.New()` ou `fmt.Errorf()`, ou definir tipos de erro customizados para casos mais complexos. A partir do Go 1.13, você pode encapsular erros com `%w` para manter o contexto enquanto adiciona informações.

```go
// Criando erros
import "errors"

func dividir(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("não é possível dividir por zero")
    }
    return a / b, nil
}

// Usando fmt.Errorf (com formatação)
func validarIdade(idade int) error {
    if idade < 0 {
        return fmt.Errorf("idade inválida: %d", idade)
    }
    return nil
}

// Tratando erros
resultado, err := dividir(10, 0)
if err != nil {
    fmt.Println("Erro:", err)
    return
}
fmt.Println("Resultado:", resultado)

// Erros customizados
type ErroValidacao struct {
    Campo string
    Valor interface{}
}

func (e *ErroValidacao) Error() string {
    return fmt.Sprintf("campo %s com valor inválido: %v", e.Campo, e.Valor)
}

// Wrapping errors (Go 1.13+)
import "fmt"

func processar() error {
    err := operacao()
    if err != nil {
        return fmt.Errorf("erro ao processar: %w", err)
    }
    return nil
}

// Verificando erros específicos
if errors.Is(err, ErrNotFound) {
    // tratar erro específico
}
```