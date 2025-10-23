## 16. Enums (Padrões)

Go não tem suporte nativo a enumerações (enums) como outras linguagens, mas a comunidade desenvolveu padrões idiomáticos usando constantes com `iota`. O `iota` é um identificador especial que começa em 0 e incrementa automaticamente a cada constante em um bloco `const`. Isso permite criar conjuntos de valores relacionados de forma clara e manutenível.

Enums são úteis para representar estados finitos, tipos categóricos, flags de configuração, ou qualquer conjunto fixo de valores possíveis. Você pode implementar o método `String()` para ter representações legíveis, criar funções de validação, e até usar bitwise operations para flags combinadas. Embora não seja tão elegante quanto enums nativos, esse padrão é amplamente adotado e eficaz na prática. Você também pode usar strings tipadas quando a serialização (JSON, por exemplo) é importante.

```go
// Padrão com iota
type DiaSemana int

const (
    Domingo DiaSemana = iota  // 0
    Segunda                    // 1
    Terca                      // 2
    Quarta                     // 3
    Quinta                     // 4
    Sexta                      // 5
    Sabado                     // 6
)

// Implementando String() para melhor output
func (d DiaSemana) String() string {
    return [...]string{
        "Domingo", "Segunda", "Terça",
        "Quarta", "Quinta", "Sexta", "Sábado",
    }[d]
}

// Uso
hoje := Segunda
fmt.Println(hoje) // Output: Segunda

// Enum com valores específicos
type Status int

const (
    Pendente Status = iota + 1  // 1
    EmAndamento                  // 2
    Concluido                    // 3
    Cancelado                    // 4
)

// Método para validação
func (s Status) Valido() bool {
    switch s {
    case Pendente, EmAndamento, Concluido, Cancelado:
        return true
    }
    return false
}

func (s Status) String() string {
    switch s {
    case Pendente:
        return "Pendente"
    case EmAndamento:
        return "Em Andamento"
    case Concluido:
        return "Concluído"
    case Cancelado:
        return "Cancelado"
    default:
        return "Desconhecido"
    }
}

// Enum com strings (melhor para JSON)
type Prioridade string

const (
    Baixa  Prioridade = "baixa"
    Media  Prioridade = "media"
    Alta   Prioridade = "alta"
)

// Validação
func (p Prioridade) Valida() bool {
    switch p {
    case Baixa, Media, Alta:
        return true
    }
    return false
}

// Uso com JSON
type Tarefa struct {
    Titulo     string     `json:"titulo"`
    Prioridade Prioridade `json:"prioridade"`
    Status     Status     `json:"status"`
}

// Enum com bitwise (flags)
type Permissao uint

const (
    Leitura  Permissao = 1 << iota  // 1 (001)
    Escrita                          // 2 (010)
    Execucao                         // 4 (100)
)

// Operações com flags
func (p Permissao) Tem(flag Permissao) bool {
    return p&flag != 0
}

func (p Permissao) Adicionar(flag Permissao) Permissao {
    return p | flag
}

func (p Permissao) Remover(flag Permissao) Permissao {
    return p &^ flag
}

func (p Permissao) String() string {
    var perms []string
    if p&Leitura != 0 {
        perms = append(perms, "Leitura")
    }
    if p&Escrita != 0 {
        perms = append(perms, "Escrita")
    }
    if p&Execucao != 0 {
        perms = append(perms, "Execução")
    }
    if len(perms) == 0 {
        return "Nenhuma"
    }
    return strings.Join(perms, ", ")
}

// Uso
perms := Leitura | Escrita  // 3 (011)
fmt.Println(perms.Tem(Leitura))   // true
fmt.Println(perms.Tem(Execucao))  // false
fmt.Println(perms)                // Output: Leitura, Escrita

perms = perms.Adicionar(Execucao) // 7 (111)
perms = perms.Remover(Escrita)    // 5 (101)

// Exemplo completo: Sistema de usuários
type TipoUsuario int

const (
    Admin TipoUsuario = iota + 1
    Moderador
    Usuario
    Convidado
)

func (t TipoUsuario) String() string {
    switch t {
    case Admin:
        return "Administrador"
    case Moderador:
        return "Moderador"
    case Usuario:
        return "Usuário"
    case Convidado:
        return "Convidado"
    default:
        return "Desconhecido"
    }
}

func (t TipoUsuario) PodeEditar() bool {
    return t == Admin || t == Moderador
}

func (t TipoUsuario) PodeExcluir() bool {
    return t == Admin
}

func (t TipoUsuario) Nivel() int {
    return int(t)
}

// Enum com métodos mais complexos
type EstadoPedido int

const (
    Criado EstadoPedido = iota
    Confirmado
    Processando
    Enviado
    Entregue
    Cancelado
)

func (e EstadoPedido) String() string {
    return [...]string{
        "Criado", "Confirmado", "Processando",
        "Enviado", "Entregue", "Cancelado",
    }[e]
}

func (e EstadoPedido) ProximoEstado() []EstadoPedido {
    switch e {
    case Criado:
        return []EstadoPedido{Confirmado, Cancelado}
    case Confirmado:
        return []EstadoPedido{Processando, Cancelado}
    case Processando:
        return []EstadoPedido{Enviado, Cancelado}
    case Enviado:
        return []EstadoPedido{Entregue}
    default:
        return []EstadoPedido{}
    }
}

func (e EstadoPedido) PodeTransicionarPara(novo EstadoPedido) bool {
    for _, estado := range e.ProximoEstado() {
        if estado == novo {
            return true
        }
    }
    return false
}

// Padrão: Parse string para enum
func ParseTipoUsuario(s string) (TipoUsuario, error) {
    switch strings.ToLower(s) {
    case "admin", "administrador":
        return Admin, nil
    case "moderador":
        return Moderador, nil
    case "usuario", "usuário":
        return Usuario, nil
    case "convidado":
        return Convidado, nil
    default:
        return 0, fmt.Errorf("tipo de usuário inválido: %s", s)
    }
}

// Enum com valores personalizados
type CodigoHTTP int

const (
    OK                  CodigoHTTP = 200
    Created             CodigoHTTP = 201
    BadRequest          CodigoHTTP = 400
    Unauthorized        CodigoHTTP = 401
    Forbidden           CodigoHTTP = 403
    NotFound            CodigoHTTP = 404
    InternalServerError CodigoHTTP = 500
)

func (c CodigoHTTP) Mensagem() string {
    switch c {
    case OK:
        return "OK"
    case Created:
        return "Criado"
    case BadRequest:
        return "Requisição inválida"
    case Unauthorized:
        return "Não autorizado"
    case Forbidden:
        return "Proibido"
    case NotFound:
        return "Não encontrado"
    case InternalServerError:
        return "Erro interno do servidor"
    default:
        return "Desconhecido"
    }
}

func (c CodigoHTTP) EhSucesso() bool {
    return c >= 200 && c < 300
}

func (c CodigoHTTP) EhErroCliente() bool {
    return c >= 400 && c < 500
}

func (c CodigoHTTP) EhErroServidor() bool {
    return c >= 500 && c < 600
}
```

## Dicas Práticas para Enums

### Quando usar inteiros vs strings:
- **Inteiros com iota**: Melhor performance, menor uso de memória, comparações rápidas
- **Strings**: Melhor para JSON/serialização, mais legível em logs, não precisa de método String()

### Padrão para todos os enums:
1. Sempre implemente o método `String()`
2. Crie função de validação (`Valido()`)
3. Se precisar parse, crie função `ParseX(string)`
4. Para flags, use bitwise operations
5. Documente os valores possíveis

### Evite:
- Criar enums muito grandes (considere usar map ou config)
- Valores duplicados sem motivo
- Mudar valores de enums existentes em produção
- Esquecer de tratar o caso `default` em switches