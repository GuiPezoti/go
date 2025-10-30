### O que é JSON?

JSON (JavaScript Object Notation) é um formato de dados leve, legível por humanos e fácil de parsear por máquinas. Criado como um subconjunto da linguagem JavaScript, JSON se tornou o formato padrão para troca de dados na web moderna, sendo independente de linguagem de programação.

JSON representa dados estruturados usando pares chave-valor (objetos), arrays, strings, números, booleanos e null. Sua simplicidade e expressividade o tornaram o formato dominante para APIs REST, arquivos de configuração, armazenamento de dados estruturados e comunicação entre sistemas distribuídos.

### Por que JSON é importante?

JSON é fundamental no desenvolvimento web moderno porque:

1. **Universalidade**: Suportado nativamente por praticamente todas as linguagens de programação
2. **Simplicidade**: Sintaxe minimalista, fácil de ler e escrever
3. **Leveza**: Menos verboso que XML, resultando em payloads menores e transferências mais rápidas
4. **Flexibilidade**: Suporta estruturas aninhadas e complexas mantendo legibilidade
5. **Padrão de APIs**: 99% das APIs REST modernas usam JSON para entrada e saída de dados

### Quando usar JSON?

JSON é usado sempre que você precisa:
- Comunicar com APIs REST (enviar e receber dados)
- Armazenar configurações de aplicações
- Persistir dados estruturados em arquivos
- Trocar mensagens entre microserviços
- Serializar objetos para armazenamento ou transmissão
- Implementar webhooks que recebem/enviam dados
- Trabalhar com bancos de dados NoSQL (MongoDB, CouchDB)

### Aplicações práticas

Em Go, o pacote `encoding/json` da biblioteca padrão oferece ferramentas poderosas para:
- **Marshal**: Converter structs Go em JSON (serialização)
- **Unmarshal**: Converter JSON em structs Go (desserialização)
- **Encoder/Decoder**: Trabalhar com streams de JSON (mais eficiente para grandes volumes)
- **Validação**: Garantir que dados JSON estão no formato esperado antes de processar

---

## Exemplos de Código

### 1. Desserialização básica: JSON para struct

```go
package main

import (
    "encoding/json"
    "fmt"
)

// Struct que representa um usuário
type User struct {
    ID       int    `json:"id"`
    Name     string `json:"name"`
    Email    string `json:"email"`
    IsActive bool   `json:"is_active"`
}

func main() {
    // JSON recebido de uma API (como string)
    jsonData := `{
        "id": 1,
        "name": "João Silva",
        "email": "joao@example.com",
        "is_active": true
    }`
    
    var user User
    
    // Unmarshal: converter JSON em struct Go
    err := json.Unmarshal([]byte(jsonData), &user)
    if err != nil {
        fmt.Println("Erro ao fazer unmarshal:", err)
        return
    }
    
    // Acessar os dados da struct
    fmt.Println("=== Usuário Desserializado ===")
    fmt.Printf("ID: %d\n", user.ID)
    fmt.Printf("Nome: %s\n", user.Name)
    fmt.Printf("Email: %s\n", user.Email)
    fmt.Printf("Ativo: %v\n", user.IsActive)
}
```

**Explicação:**
- Tags `json:"campo"` mapeiam campos da struct para nomes no JSON
- `json.Unmarshal()` recebe bytes e um ponteiro para a struct
- Go é case-sensitive, mas tags JSON permitem qualquer formato (snake_case, camelCase)
- Sempre passe um ponteiro (&user) para Unmarshal modificar a variável

### 2. Serialização básica: struct para JSON

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Product struct {
    ID          int     `json:"id"`
    Name        string  `json:"name"`
    Price       float64 `json:"price"`
    InStock     bool    `json:"in_stock"`
    Description string  `json:"description,omitempty"` // omitempty: não inclui se vazio
    Tags        []string `json:"tags,omitempty"`
}

func main() {
    // Criar um produto
    product := Product{
        ID:      101,
        Name:    "Teclado Mecânico",
        Price:   299.90,
        InStock: true,
        Tags:    []string{"periférico", "gaming"},
        // Description propositalmente omitida
    }
    
    // Marshal: converter struct em JSON (formato compacto)
    jsonBytes, err := json.Marshal(product)
    if err != nil {
        fmt.Println("Erro ao fazer marshal:", err)
        return
    }
    
    fmt.Println("JSON Compacto:")
    fmt.Println(string(jsonBytes))
    
    // MarshalIndent: JSON formatado (mais legível)
    jsonPretty, err := json.MarshalIndent(product, "", "  ")
    if err != nil {
        fmt.Println("Erro ao fazer marshal:", err)
        return
    }
    
    fmt.Println("\nJSON Formatado:")
    fmt.Println(string(jsonPretty))
}
```

**Explicação:**
- `json.Marshal()` retorna bytes, não string (use `string()` para converter)
- `json.MarshalIndent()` adiciona indentação para melhor legibilidade
- `omitempty` não inclui o campo no JSON se estiver vazio (zero value)
- Tags vazias serão arrays vazios `[]`, não null

### 3. Trabalhando com arrays JSON

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Task struct {
    ID        int    `json:"id"`
    Title     string `json:"title"`
    Completed bool   `json:"completed"`
}

func main() {
    // Array JSON (muito comum em APIs que retornam listas)
    jsonData := `[
        {"id": 1, "title": "Estudar Go", "completed": true},
        {"id": 2, "title": "Fazer exercícios", "completed": false},
        {"id": 3, "title": "Revisar HTTP", "completed": false},
        {"id": 4, "title": "Aprender JSON", "completed": true}
    ]`
    
    // Slice de structs para armazenar o array
    var tasks []Task
    
    err := json.Unmarshal([]byte(jsonData), &tasks)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    
    fmt.Println("=== Lista de Tarefas ===\n")
    
    completedCount := 0
    
    for _, task := range tasks {
        status := "❌ Pendente"
        if task.Completed {
            status = "✅ Completa"
            completedCount++
        }
        fmt.Printf("[%d] %s - %s\n", task.ID, task.Title, status)
    }
    
    fmt.Printf("\nProgresso: %d/%d tarefas completas\n", completedCount, len(tasks))
    
    // Serializar de volta para JSON
    updatedJSON, _ := json.MarshalIndent(tasks, "", "  ")
    fmt.Println("\nJSON atualizado:")
    fmt.Println(string(updatedJSON))
}
```

**Explicação:**
- Use slice de structs (`[]Task`) para arrays JSON
- Cada elemento do array vira um item no slice
- Útil para APIs que retornam listas: usuários, produtos, posts, etc.
- Pode iterar normalmente com `range`

### 4. JSON com requisições HTTP (GET)

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
)

type Post struct {
    UserID int    `json:"userId"`
    ID     int    `json:"id"`
    Title  string `json:"title"`
    Body   string `json:"body"`
}

func main() {
    // Fazer requisição para API que retorna JSON
    url := "https://jsonplaceholder.typicode.com/posts/1"
    
    resp, err := http.Get(url)
    if err != nil {
        fmt.Println("Erro na requisição:", err)
        return
    }
    defer resp.Body.Close()
    
    // Verificar status code
    if resp.StatusCode != 200 {
        fmt.Printf("Erro: Status %d\n", resp.StatusCode)
        return
    }
    
    // Decodificar JSON diretamente do Body da resposta
    var post Post
    err = json.NewDecoder(resp.Body).Decode(&post)
    if err != nil {
        fmt.Println("Erro ao decodificar JSON:", err)
        return
    }
    
    fmt.Println("=== Post Recebido ===")
    fmt.Printf("ID: %d\n", post.ID)
    fmt.Printf("User ID: %d\n", post.UserID)
    fmt.Printf("Título: %s\n", post.Title)
    fmt.Printf("Corpo: %s\n", post.Body[:100]+"...") // Primeiros 100 caracteres
}
```

**Explicação:**
- `json.NewDecoder()` é mais eficiente para ler de streams (como HTTP Body)
- `Decode()` lê diretamente do `io.Reader` sem precisar carregar tudo na memória
- Perfeito para respostas HTTP
- Sempre verifique o status code antes de decodificar

### 5. JSON com requisições HTTP (POST)

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

type CreateUserRequest struct {
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age"`
}

type CreateUserResponse struct {
    ID        int    `json:"id"`
    Name      string `json:"name"`
    Email     string `json:"email"`
    CreatedAt string `json:"createdAt"`
}

func main() {
    // Dados a serem enviados
    newUser := CreateUserRequest{
        Name:  "Maria Santos",
        Email: "maria@example.com",
        Age:   28,
    }
    
    // Serializar struct para JSON
    jsonData, err := json.Marshal(newUser)
    if err != nil {
        fmt.Println("Erro ao serializar:", err)
        return
    }
    
    fmt.Println("Enviando JSON:")
    fmt.Println(string(jsonData))
    
    // Fazer requisição POST com JSON
    url := "https://jsonplaceholder.typicode.com/users"
    resp, err := http.Post(
        url,
        "application/json", // Content-Type
        bytes.NewBuffer(jsonData),
    )
    if err != nil {
        fmt.Println("Erro na requisição:", err)
        return
    }
    defer resp.Body.Close()
    
    fmt.Printf("\nStatus: %d %s\n", resp.StatusCode, resp.Status)
    
    // Decodificar resposta JSON
    var response CreateUserResponse
    err = json.NewDecoder(resp.Body).Decode(&response)
    if err != nil {
        fmt.Println("Erro ao decodificar resposta:", err)
        return
    }
    
    fmt.Println("\n=== Usuário Criado ===")
    fmt.Printf("ID: %d\n", response.ID)
    fmt.Printf("Nome: %s\n", response.Name)
    fmt.Printf("Email: %s\n", response.Email)
    fmt.Printf("Criado em: %s\n", response.CreatedAt)
}
```

**Explicação:**
- Para enviar JSON, serialize com `Marshal()` e coloque em um `bytes.Buffer`
- Sempre defina `Content-Type: application/json` no POST
- A resposta também pode ser JSON (decodifique com `NewDecoder`)
- Estruturas diferentes para request e response são comuns

### 6. JSON aninhado (nested)

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Address struct {
    Street  string `json:"street"`
    City    string `json:"city"`
    Country string `json:"country"`
    ZipCode string `json:"zip_code"`
}

type Company struct {
    Name    string  `json:"name"`
    Address Address `json:"address"`
}

type Employee struct {
    ID       int     `json:"id"`
    Name     string  `json:"name"`
    Position string  `json:"position"`
    Salary   float64 `json:"salary"`
    Company  Company `json:"company"`
}

func main() {
    // JSON complexo e aninhado
    jsonData := `{
        "id": 42,
        "name": "Carlos Oliveira",
        "position": "Desenvolvedor Senior",
        "salary": 12000.50,
        "company": {
            "name": "Tech Corp",
            "address": {
                "street": "Av. Paulista, 1000",
                "city": "São Paulo",
                "country": "Brasil",
                "zip_code": "01310-100"
            }
        }
    }`
    
    var employee Employee
    err := json.Unmarshal([]byte(jsonData), &employee)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    
    fmt.Println("=== Informações do Funcionário ===")
    fmt.Printf("Nome: %s\n", employee.Name)
    fmt.Printf("Cargo: %s\n", employee.Position)
    fmt.Printf("Salário: R$ %.2f\n", employee.Salary)
    fmt.Println("\n=== Empresa ===")
    fmt.Printf("Nome: %s\n", employee.Company.Name)
    fmt.Printf("Endereço: %s, %s - %s\n", 
        employee.Company.Address.Street,
        employee.Company.Address.City,
        employee.Company.Address.Country)
    
    // Serializar de volta
    updatedJSON, _ := json.MarshalIndent(employee, "", "  ")
    fmt.Println("\n=== JSON Serializado ===")
    fmt.Println(string(updatedJSON))
}
```

**Explicação:**
- Structs aninhadas representam objetos JSON aninhados
- Acesso via ponto: `employee.Company.Address.City`
- Go mantém a hierarquia e estrutura do JSON
- Muito comum em respostas de APIs complexas

### 7. Trabalhando com campos opcionais e ponteiros

```go
package main

import (
    "encoding/json"
    "fmt"
)

type UserProfile struct {
    ID       int     `json:"id"`
    Username string  `json:"username"`
    Email    string  `json:"email"`
    Bio      *string `json:"bio"`              // Ponteiro para campo opcional
    Age      *int    `json:"age"`              // Pode ser null
    Premium  bool    `json:"premium"`
    Score    *int    `json:"score,omitempty"`  // Não aparece se nil
}

func main() {
    // JSON com campos null
    jsonData := `{
        "id": 123,
        "username": "gopher",
        "email": "gopher@golang.org",
        "bio": null,
        "age": 25,
        "premium": true
    }`
    
    var profile UserProfile
    err := json.Unmarshal([]byte(jsonData), &profile)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    
    fmt.Println("=== Perfil do Usuário ===")
    fmt.Printf("Username: %s\n", profile.Username)
    fmt.Printf("Email: %s\n", profile.Email)
    
    // Verificar se bio está definida (não é nil)
    if profile.Bio != nil {
        fmt.Printf("Bio: %s\n", *profile.Bio)
    } else {
        fmt.Println("Bio: (não definida)")
    }
    
    // Verificar age
    if profile.Age != nil {
        fmt.Printf("Idade: %d anos\n", *profile.Age)
    } else {
        fmt.Println("Idade: (não informada)")
    }
    
    // Score não foi enviado no JSON, então é nil
    if profile.Score != nil {
        fmt.Printf("Score: %d\n", *profile.Score)
    } else {
        fmt.Println("Score: (não disponível)")
    }
    
    // Adicionar bio e serializar
    bio := "Desenvolvedor Go apaixonado"
    profile.Bio = &bio
    
    score := 850
    profile.Score = &score
    
    updatedJSON, _ := json.MarshalIndent(profile, "", "  ")
    fmt.Println("\n=== JSON Atualizado ===")
    fmt.Println(string(updatedJSON))
}
```

**Explicação:**
- Use ponteiros (`*string`, `*int`) para campos que podem ser `null` no JSON
- `nil` em Go = `null` em JSON
- Sempre verifique se ponteiro não é `nil` antes de usar
- `omitempty` com ponteiro não serializa se for `nil`

### 8. JSON com map para estruturas dinâmicas

```go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    // JSON com estrutura desconhecida ou dinâmica
    jsonData := `{
        "status": "success",
        "code": 200,
        "data": {
            "user_id": 123,
            "permissions": ["read", "write", "delete"],
            "metadata": {
                "last_login": "2025-01-15",
                "ip_address": "192.168.1.1"
            }
        }
    }`
    
    // Use map[string]interface{} para JSON dinâmico
    var result map[string]interface{}
    
    err := json.Unmarshal([]byte(jsonData), &result)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    
    fmt.Println("=== Dados Dinâmicos ===")
    fmt.Printf("Status: %v\n", result["status"])
    fmt.Printf("Code: %.0f\n", result["code"]) // Números vêm como float64
    
    // Acessar dados aninhados
    if data, ok := result["data"].(map[string]interface{}); ok {
        fmt.Printf("User ID: %.0f\n", data["user_id"])
        
        // Array de strings
        if permissions, ok := data["permissions"].([]interface{}); ok {
            fmt.Println("Permissões:")
            for _, perm := range permissions {
                fmt.Printf("  - %v\n", perm)
            }
        }
        
        // Objeto aninhado
        if metadata, ok := data["metadata"].(map[string]interface{}); ok {
            fmt.Printf("Último login: %v\n", metadata["last_login"])
            fmt.Printf("IP: %v\n", metadata["ip_address"])
        }
    }
}
```

**Explicação:**
- `map[string]interface{}` aceita qualquer estrutura JSON
- Útil quando estrutura é desconhecida ou varia
- Requer type assertions para acessar valores
- Números sempre vêm como `float64`
- Menos type-safe, use apenas quando necessário

---

## Best Practices

### ✅ Use tags JSON apropriadas
```go
type User struct {
    ID        int    `json:"id"`
    FirstName string `json:"first_name"`      // snake_case para APIs
    Email     string `json:"email"`
    Password  string `json:"-"`               // Nunca serializar
    TempData  string `json:"temp,omitempty"`  // Omitir se vazio
}
```

### ✅ Sempre valide JSON recebido
```go
var user User
err := json.Unmarshal(jsonData, &user)
if err != nil {
    return fmt.Errorf("JSON inválido: %w", err)
}

// Validar campos obrigatórios
if user.Email == "" {
    return errors.New("email é obrigatório")
}
```

### ✅ Use Decoder para streams grandes
```go
// ❌ Para arquivos grandes (carrega tudo na memória)
data, _ := ioutil.ReadFile("huge.json")
json.Unmarshal(data, &result)

// ✅ Melhor para grandes volumes
file, _ := os.Open("huge.json")
defer file.Close()
json.NewDecoder(file).Decode(&result)
```

### ✅ Estruture tipos específicos para request/response
```go
// Request e Response separados (mais claro)
type CreateUserRequest struct {
    Name  string `json:"name"`
    Email string `json:"email"`
}

type CreateUserResponse struct {
    ID        int    `json:"id"`
    Name      string `json:"name"`
    CreatedAt string `json:"created_at"`
}
```

### ✅ Use ponteiros para campos opcionais
```go
type UpdateUser struct {
    Name  *string `json:"name,omitempty"`   // Pode atualizar ou não
    Email *string `json:"email,omitempty"`  // nil = não enviar
    Age   *int    `json:"age,omitempty"`
}
```

---

## Erros Comuns a Evitar

### ❌ Esquecer ponteiro no Unmarshal
```go
var user User
json.Unmarshal(data, user) // ERRADO! user não será modificado
json.Unmarshal(data, &user) // CORRETO!
```

### ❌ Não tratar erros de parsing
```go
// ERRADO - JSON inválido pode crashar
json.Unmarshal(invalidJSON, &user)
processUser(user) // user pode estar vazio/inválido!
```

### ❌ Tags JSON incorretas
```go
type User struct {
    UserName string `json:"username"` // API espera "user_name"
    // Vai falhar silenciosamente!
}
```

### ❌ Ignorar omitempty quando necessário
```go
type Product struct {
    ID    int     `json:"id"`
    Price float64 `json:"price"` // Se 0, vai enviar "price": 0
    // Deveria ser: `json:"price,omitempty"`
}
```

### ❌ Não verificar status code antes de decodificar
```go
resp, _ := http.Get(url)
defer resp.Body.Close()
json.NewDecoder(resp.Body).Decode(&result)
// E se for 404? 500? Vai decodificar mensagem de erro!
```

---

## Padrões Comuns

### Pattern: Response wrapper padrão
```go
type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   string      `json:"error,omitempty"`
}

func sendJSON(w http.ResponseWriter, data interface{}) {
    response := APIResponse{
        Success: true,
        Data:    data,
    }
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}
```

### Pattern: Validação de JSON
```go
func validateUserJSON(data []byte) (*User, error) {
    var user User
    
    if err := json.Unmarshal(data, &user); err != nil {
        return nil, fmt.Errorf("JSON inválido: %w", err)
    }
    
    if user.Email == "" {
        return nil, errors.New("email é obrigatório")
    }
    
    if user.Name == "" {
        return nil, errors.New("nome é obrigatório")
    }
    
    return &user, nil
}
```

### Pattern: Pretty print para debug
```go
func prettyPrint(data interface{}) {
    jsonBytes, err := json.MarshalIndent(data, "", "  ")
    if err != nil {
        log.Println("Erro ao serializar:", err)
        return
    }
    fmt.Println(string(jsonBytes))
}
```

### Pattern: JSON para arquivo
```go
func saveToJSON(filename string, data interface{}) error {
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    encoder := json.NewEncoder(file)
    encoder.SetIndent("", "  ") // Formatado
    return encoder.Encode(data)
}
```

---

**Fim do Tópico 2**

*Próximo: Tópico 3 - DNS*