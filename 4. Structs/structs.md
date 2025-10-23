## 4. Structs

Structs são a forma de Go criar tipos de dados personalizados e estruturados. Diferente de linguagens orientadas a objetos tradicionais, Go não tem classes, mas structs combinadas com métodos fornecem funcionalidade similar de forma mais simples e explícita. Structs são tipos de valor, o que significa que quando você as passa para funções, uma cópia é criada por padrão.

Na prática, structs são usadas para modelar entidades do mundo real: usuários, produtos, pedidos, configurações, etc. Você pode compor structs dentro de outras structs (composição em vez de herança), anexar métodos a elas para criar comportamentos, e usar tags para controlar serialização JSON, validação, ou mapeamento de banco de dados. Métodos com receptores de ponteiro (`*T`) modificam o original, enquanto receptores de valor (`T`) trabalham com cópias.

```go
// Definição de struct
type Pessoa struct {
    Nome  string
    Idade int
    Email string
}

// Criando instâncias
p1 := Pessoa{Nome: "Ana", Idade: 30, Email: "ana@email.com"}
p2 := Pessoa{"Carlos", 25, "carlos@email.com"} // ordem importa
var p3 Pessoa // valores zero

// Acessando campos
fmt.Println(p1.Nome)
p1.Idade = 31

// Structs aninhadas
type Endereco struct {
    Rua    string
    Cidade string
}

type Usuario struct {
    Nome     string
    Endereco Endereco
}

u := Usuario{
    Nome: "João",
    Endereco: Endereco{
        Rua:    "Av Paulista",
        Cidade: "São Paulo",
    },
}

// Métodos em structs
func (p Pessoa) Apresentar() string {
    return fmt.Sprintf("Olá, sou %s e tenho %d anos", p.Nome, p.Idade)
}

// Método com ponteiro (modifica o original)
func (p *Pessoa) FazerAniversario() {
    p.Idade++
}
```