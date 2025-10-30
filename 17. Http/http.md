### O que é HTTP?

HTTP (HyperText Transfer Protocol) é o protocolo fundamental que sustenta toda a World Wide Web. Criado por Tim Berners-Lee em 1989, HTTP é um protocolo de comunicação cliente-servidor baseado em texto que define como mensagens são formatadas e transmitidas, e quais ações servidores web e navegadores devem executar em resposta a diversos comandos.

HTTP funciona no modelo requisição-resposta: um cliente (navegador, aplicativo mobile, script) envia uma requisição para um servidor, que processa essa requisição e retorna uma resposta. Essa comunicação acontece sobre o protocolo TCP/IP, geralmente na porta 80 (HTTP) ou 443 (HTTPS).

### Por que HTTP é importante?

HTTP é a linguagem universal da internet moderna. Praticamente toda interação na web usa HTTP: carregar páginas web, fazer login em sistemas, enviar formulários, consumir APIs REST, fazer uploads de arquivos, streaming de vídeo, e muito mais. Dominar HTTP é essencial porque:

1. **Ubiquidade**: Toda aplicação web, API, microserviço e aplicativo mobile usa HTTP para comunicação
2. **Simplicidade**: Protocolo baseado em texto, legível por humanos, fácil de debugar
3. **Flexibilidade**: Suporta diversos tipos de conteúdo (JSON, XML, HTML, imagens, vídeos)
4. **Stateless**: Cada requisição é independente, facilitando escalabilidade
5. **Padronização**: Especificação aberta seguida universalmente

### Quando usar HTTP?

HTTP é usado sempre que você precisa:
- Consumir ou criar APIs RESTful
- Fazer integrações com serviços externos (pagamento, envio de emails, autenticação)
- Criar aplicações web (front-end comunicando com back-end)
- Implementar webhooks para receber notificações
- Fazer scraping de dados da web
- Realizar comunicação entre microserviços
- Criar clientes para APIs públicas (GitHub, Twitter, Google, etc.)

Em Go, a biblioteca padrão `net/http` é incrivelmente poderosa e permite criar tanto clientes HTTP (fazer requisições) quanto servidores HTTP (receber requisições) sem dependências externas.

### Aplicações práticas

HTTP em Go é usado em cenários do mundo real como:
- **Backend de aplicações**: Criar APIs REST que aplicativos mobile e front-ends consomem
- **Integrações**: Conectar seu sistema com serviços de pagamento, SMS, email
- **Automação**: Scripts que fazem requisições para coletar dados ou executar ações
- **Monitoramento**: Health checks, testes de disponibilidade de serviços
- **Microserviços**: Comunicação entre diferentes serviços de uma arquitetura distribuída

---

## Exemplos de Código

### 1. Requisição GET simples

```go
package main

import (
    "fmt"
    "io"
    "net/http"
)

func main() {
    // Fazer uma requisição GET para uma API pública
    resp, err := http.Get("https://api.github.com")
    if err != nil {
        fmt.Println("Erro ao fazer requisição:", err)
        return
    }
    defer resp.Body.Close() // IMPORTANTE: sempre fechar o Body
    
    // Ler o corpo da resposta
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("Erro ao ler resposta:", err)
        return
    }
    
    // Exibir informações da resposta
    fmt.Println("Status Code:", resp.StatusCode)
    fmt.Println("Status:", resp.Status)
    fmt.Println("\nResposta:")
    fmt.Println(string(body))
}
```

**Explicação:**
- `http.Get()` é a forma mais simples de fazer uma requisição GET
- `resp.Body` é um `io.ReadCloser` que deve SEMPRE ser fechado com `defer resp.Body.Close()`
- `io.ReadAll()` lê todo o conteúdo do body para um slice de bytes
- O status code (200, 404, 500, etc.) indica o resultado da requisição

### 2. Cliente HTTP com timeout

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "time"
)

func main() {
    // Criar um cliente HTTP customizado com timeout
    client := &http.Client{
        Timeout: 10 * time.Second, // Aguardar no máximo 10 segundos
    }
    
    // URL que demora para responder (simulação)
    url := "https://httpbin.org/delay/2" // Demora 2 segundos para responder
    
    fmt.Println("Fazendo requisição com timeout de 10s...")
    start := time.Now()
    
    resp, err := client.Get(url)
    if err != nil {
        fmt.Println("❌ Erro:", err)
        return
    }
    defer resp.Body.Close()
    
    duration := time.Since(start)
    
    body, _ := io.ReadAll(resp.Body)
    
    fmt.Printf("✅ Resposta recebida em %v\n", duration)
    fmt.Println("Status:", resp.Status)
    fmt.Println("Tamanho da resposta:", len(body), "bytes")
}
```

**Explicação:**
- Criar um `http.Client` customizado permite configurar comportamentos
- `Timeout` define quanto tempo aguardar por uma resposta completa
- Sem timeout, uma requisição pode travar indefinidamente
- Timeouts são essenciais em produção para evitar que serviços lentos travem sua aplicação

### 3. Fazendo múltiplas requisições

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "sync"
)

func fetchURL(url string, wg *sync.WaitGroup) {
    defer wg.Done() // Sinaliza que esta goroutine terminou
    
    resp, err := http.Get(url)
    if err != nil {
        fmt.Printf("❌ Erro ao acessar %s: %v\n", url, err)
        return
    }
    defer resp.Body.Close()
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        fmt.Printf("❌ Erro ao ler %s: %v\n", url, err)
        return
    }
    
    fmt.Printf("✅ %s - Status: %d, Tamanho: %d bytes\n", 
        url, resp.StatusCode, len(body))
}

func main() {
    urls := []string{
        "https://api.github.com",
        "https://httpbin.org/get",
        "https://jsonplaceholder.typicode.com/posts/1",
    }
    
    var wg sync.WaitGroup
    
    fmt.Println("Fazendo requisições paralelas...\n")
    
    for _, url := range urls {
        wg.Add(1)
        go fetchURL(url, &wg) // Requisições em paralelo
    }
    
    wg.Wait() // Aguardar todas terminarem
    fmt.Println("\n✅ Todas as requisições foram completadas!")
}
```

**Explicação:**
- Goroutines permitem fazer múltiplas requisições HTTP em paralelo
- `sync.WaitGroup` coordena quando todas as goroutines terminaram
- Fazer requisições em paralelo é muito mais rápido que sequencial
- Útil para fazer health checks, agregar dados de múltiplas APIs, etc.

### 4. Criando requisições customizadas

```go
package main

import (
    "fmt"
    "io"
    "net/http"
)

func main() {
    // Criar uma requisição manualmente (mais controle)
    req, err := http.NewRequest("GET", "https://api.github.com/users/golang", nil)
    if err != nil {
        fmt.Println("Erro ao criar requisição:", err)
        return
    }
    
    // Adicionar headers customizados
    req.Header.Set("User-Agent", "MeuAppGo/1.0")
    req.Header.Set("Accept", "application/json")
    
    // Executar a requisição
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Println("Erro ao executar requisição:", err)
        return
    }
    defer resp.Body.Close()
    
    body, _ := io.ReadAll(resp.Body)
    
    fmt.Println("=== Requisição Customizada ===")
    fmt.Println("Status:", resp.Status)
    fmt.Println("Content-Type:", resp.Header.Get("Content-Type"))
    fmt.Println("\nResposta:")
    fmt.Println(string(body)[:300] + "...") // Primeiros 300 caracteres
}
```

**Explicação:**
- `http.NewRequest()` cria uma requisição com controle total
- Permite adicionar headers customizados antes de enviar
- `client.Do(req)` executa a requisição criada
- Útil quando você precisa mais controle sobre a requisição

### 5. Verificando status codes

```go
package main

import (
    "fmt"
    "net/http"
)

func checkAPIStatus(url string) {
    resp, err := http.Get(url)
    if err != nil {
        fmt.Printf("❌ %s - Erro de conexão: %v\n", url, err)
        return
    }
    defer resp.Body.Close()
    
    // Interpretar diferentes status codes
    switch {
    case resp.StatusCode >= 200 && resp.StatusCode < 300:
        fmt.Printf("✅ %s - Sucesso (%d %s)\n", url, resp.StatusCode, resp.Status)
    case resp.StatusCode >= 300 && resp.StatusCode < 400:
        fmt.Printf("↪️  %s - Redirecionamento (%d %s)\n", url, resp.StatusCode, resp.Status)
    case resp.StatusCode >= 400 && resp.StatusCode < 500:
        fmt.Printf("⚠️  %s - Erro do cliente (%d %s)\n", url, resp.StatusCode, resp.Status)
    case resp.StatusCode >= 500:
        fmt.Printf("❌ %s - Erro do servidor (%d %s)\n", url, resp.StatusCode, resp.Status)
    }
}

func main() {
    urls := []string{
        "https://httpbin.org/status/200", // Sucesso
        "https://httpbin.org/status/404", // Not Found
        "https://httpbin.org/status/500", // Server Error
    }
    
    fmt.Println("Verificando status de APIs...\n")
    
    for _, url := range urls {
        checkAPIStatus(url)
    }
}
```

**Explicação:**
- Status codes indicam o resultado da requisição
- **2xx**: Sucesso (200 OK, 201 Created, 204 No Content)
- **3xx**: Redirecionamento (301 Moved, 302 Found)
- **4xx**: Erro do cliente (400 Bad Request, 401 Unauthorized, 404 Not Found)
- **5xx**: Erro do servidor (500 Internal Server Error, 503 Service Unavailable)
- Sempre verifique o status code antes de processar a resposta

### 6. Servidor HTTP básico (bônus)

```go
package main

import (
    "fmt"
    "net/http"
)

// Handler que responde a requisições
func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Olá! Você acessou: %s\n", r.URL.Path)
    fmt.Fprintf(w, "Método: %s\n", r.Method)
}

func main() {
    // Registrar handler para uma rota
    http.HandleFunc("/", helloHandler)
    
    fmt.Println("🚀 Servidor rodando em http://localhost:8080")
    fmt.Println("Acesse http://localhost:8080 no seu navegador")
    
    // Iniciar servidor na porta 8080
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        fmt.Println("Erro ao iniciar servidor:", err)
    }
}
```

**Explicação:**
- Go permite criar servidores HTTP com poucas linhas
- `http.HandleFunc()` registra uma função para uma rota específica
- `http.ListenAndServe()` inicia o servidor na porta especificada
- Este é um servidor HTTP completo e funcional!

---

## Best Practices

### ✅ Sempre fechar o Body da resposta
```go
resp, err := http.Get(url)
if err != nil {
    return err
}
defer resp.Body.Close() // Essencial para evitar memory leaks
```

### ✅ Configurar timeouts apropriados
```go
client := &http.Client{
    Timeout: 30 * time.Second, // Nunca deixe sem timeout!
}
```

### ✅ Reutilizar clientes HTTP
```go
// ❌ NÃO fazer isso
func fazerRequisicao() {
    client := &http.Client{} // Cliente novo a cada chamada
    client.Get(url)
}

// ✅ FAZER isso
var httpClient = &http.Client{
    Timeout: 10 * time.Second,
}

func fazerRequisicao() {
    httpClient.Get(url) // Reutilizar cliente
}
```

### ✅ Verificar status codes antes de processar
```go
resp, err := http.Get(url)
if err != nil {
    return err
}
defer resp.Body.Close()

if resp.StatusCode != 200 {
    return fmt.Errorf("erro: status %d", resp.StatusCode)
}

// Processar resposta apenas se status for 200
```

### ✅ Tratar erros de rede adequadamente
```go
resp, err := http.Get(url)
if err != nil {
    // Verificar se é timeout
    if os.IsTimeout(err) {
        fmt.Println("Timeout ao conectar")
    }
    return err
}
```

---

## Erros Comuns a Evitar

### ❌ Esquecer de fechar o Body
```go
// ERRADO - memory leak!
resp, _ := http.Get(url)
body, _ := io.ReadAll(resp.Body)
// Body nunca foi fechado!
```

### ❌ Não configurar timeout
```go
// ERRADO - pode travar indefinidamente
client := &http.Client{} // Sem timeout!
resp, _ := client.Get("http://site-muito-lento.com")
```

### ❌ Ignorar erros
```go
// ERRADO - ignora completamente erros
resp, _ := http.Get(url)
body, _ := io.ReadAll(resp.Body)
```

### ❌ Não verificar status code
```go
// ERRADO - processa mesmo se der erro 404 ou 500
resp, _ := http.Get(url)
defer resp.Body.Close()
body, _ := io.ReadAll(resp.Body)
// E se for um erro 404? 500? Vai processar dados inválidos
```

### ❌ Criar cliente para cada requisição
```go
// ERRADO - muito ineficiente
for i := 0; i < 1000; i++ {
    client := &http.Client{} // Cliente novo 1000 vezes!
    client.Get(url)
}
```

---

## Padrões Comuns

### Pattern: Health Check
```go
func checkHealth(url string) bool {
    client := &http.Client{Timeout: 5 * time.Second}
    resp, err := client.Get(url + "/health")
    if err != nil {
        return false
    }
    defer resp.Body.Close()
    return resp.StatusCode == 200
}
```

### Pattern: Retry com Backoff
```go
func getWithRetry(url string, maxRetries int) (*http.Response, error) {
    var resp *http.Response
    var err error
    
    for i := 0; i < maxRetries; i++ {
        resp, err = http.Get(url)
        if err == nil && resp.StatusCode == 200 {
            return resp, nil
        }
        
        // Aguardar antes de tentar novamente
        time.Sleep(time.Duration(i+1) * time.Second)
    }
    
    return nil, fmt.Errorf("falhou após %d tentativas", maxRetries)
}
```

### Pattern: Request com Context (cancelamento)
```go
func fetchWithCancel(ctx context.Context, url string) error {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return err
    }
    
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    // Processar resposta...
    return nil
}
```

---