## 11. Packages e Modules

O sistema de módulos do Go organiza código em pacotes reutilizáveis e gerencia dependências de forma eficiente. Um módulo é uma coleção de pacotes versionados juntos, e todo projeto Go moderno deve ter um arquivo `go.mod` que define o módulo e suas dependências. Pacotes são diretórios contendo arquivos `.go` com a mesma declaração de `package` no topo.

A organização em pacotes promove modularidade, reutilização e encapsulamento. A convenção de nomenclatura é simples: identificadores com primeira letra maiúscula são exportados (públicos), minúscula são privados ao pacote. Isso elimina a necessidade de palavras-chave como `public` ou `private`. O Go Modules resolve dependências transitivas, gerencia versões e garante builds reproduzíveis. Comandos como `go get`, `go mod tidy` e `go build` tornam o gerenciamento de dependências transparente.

```go
// Estrutura de projeto
// myproject/
//   go.mod
//   main.go
//   utils/
//     helper.go
//   models/
//     user.go

// go.mod (arquivo de módulo)
/*
module github.com/usuario/myproject

go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
)
*/

// Declarando package
package main  // package main = executável
package utils // outros packages são bibliotecas

// Importando packages
import "fmt"
import "math"

// Import múltiplo
import (
    "fmt"
    "time"
    "github.com/usuario/myproject/utils"
)

// Alias de import
import (
    f "fmt"
    matematica "math"
)

// Import blank (apenas side effects)
import _ "github.com/lib/pq"

// Exportação (visibilidade)
// Maiúscula = público (exportado)
type Usuario struct {
    Nome  string  // público
    idade int     // privado
}

func Processar() {}  // público
func validar() {}    // privado

// Comandos úteis
// go mod init github.com/usuario/projeto
// go mod tidy
// go get github.com/algum/package
// go build
// go run main.go

// Init function (executa antes de main)
func init() {
    // inicialização do package
}
```