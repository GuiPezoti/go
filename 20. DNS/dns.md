### O que é DNS?

DNS (Domain Name System) é o "sistema de endereçamento" da internet, funcionando como uma agenda telefônica global que traduz nomes de domínio legíveis por humanos (como `google.com`) em endereços IP numéricos (como `142.250.185.46`) que os computadores usam para se comunicar. Sem DNS, teríamos que memorizar sequências de números para acessar cada site.

O DNS é um sistema distribuído e hierárquico composto por milhões de servidores ao redor do mundo. Quando você digita um endereço no navegador, uma cadeia de consultas DNS acontece em milissegundos, passando por diferentes níveis de servidores (resolver local, root servers, TLD servers, authoritative nameservers) até encontrar o IP correto do destino.

### Por que DNS é importante?

DNS é fundamental para o funcionamento da internet moderna porque:

1. **Usabilidade**: Permite usar nomes memoráveis ao invés de IPs numéricos
2. **Flexibilidade**: Permite mudar servidores sem alterar o nome do serviço
3. **Distribuição de carga**: Um domínio pode resolver para múltiplos IPs (load balancing)
4. **Redundância**: Múltiplos servidores DNS garantem disponibilidade
5. **Geolocalização**: Pode retornar IPs diferentes baseado na localização do usuário
6. **Diagnóstico**: Problemas de DNS são uma causa comum de falhas de conectividade

### Quando trabalhar com DNS?

Você interage com DNS quando precisa:
- Diagnosticar problemas de conectividade de rede
- Implementar health checks e monitoramento de serviços
- Configurar failover automático entre servidores
- Descobrir serviços disponíveis em uma rede
- Validar configurações de domínio
- Implementar resolução customizada de nomes
- Trabalhar com ambientes multi-região
- Fazer reverse lookup (IP para hostname)

### Aplicações práticas

Em Go, o pacote `net` oferece ferramentas para trabalhar com DNS:
- **Lookup de IPs**: Resolver nomes de domínio para endereços IP
- **Reverse lookup**: Descobrir hostname a partir de um IP
- **Registros específicos**: Buscar MX (email), NS (nameservers), CNAME, TXT
- **Resolvers customizados**: Controlar timeouts e servidores DNS usados
- **Validação de domínios**: Verificar se domínios existem e estão acessíveis

---

## Exemplos de Código

### 1. Lookup básico de IP

```go
package main

import (
    "fmt"
    "net"
)

func main() {
    // Resolver um nome de domínio para endereços IP
    hostname := "google.com"
    
    fmt.Printf("Resolvendo %s...\n\n", hostname)
    
    ips, err := net.LookupIP(hostname)
    if err != nil {
        fmt.Println("Erro ao resolver DNS:", err)
        return
    }
    
    fmt.Printf("Endereços IP encontrados para %s:\n", hostname)
    for i, ip := range ips {
        ipType := "IPv4"
        if ip.To4() == nil {
            ipType = "IPv6"
        }
        fmt.Printf("  %d. %s (%s)\n", i+1, ip.String(), ipType)
    }
    
    fmt.Printf("\nTotal: %d endereços encontrados\n", len(ips))
}
```

**Explicação:**
- `net.LookupIP()` retorna todos os IPs associados ao domínio
- Um domínio pode ter múltiplos IPs (IPv4 e IPv6)
- `ip.To4()` verifica se é IPv4 (retorna nil para IPv6)
- Múltiplos IPs permitem load balancing e redundância

### 2. Lookup de host específico (apenas IPv4 ou IPv6)

```go
package main

import (
    "fmt"
    "net"
)

func main() {
    hostname := "github.com"
    
    // Buscar apenas IPv4
    fmt.Println("=== Lookup IPv4 ===")
    ipv4Addrs, err := net.LookupHost(hostname)
    if err != nil {
        fmt.Println("Erro:", err)
    } else {
        for _, addr := range ipv4Addrs {
            fmt.Printf("  - %s\n", addr)
        }
    }
    
    // Buscar IPs e filtrar por tipo
    fmt.Println("\n=== Separando IPv4 e IPv6 ===")
    ips, err := net.LookupIP(hostname)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    
    var ipv4List []net.IP
    var ipv6List []net.IP
    
    for _, ip := range ips {
        if ip.To4() != nil {
            ipv4List = append(ipv4List, ip)
        } else {
            ipv6List = append(ipv6List, ip)
        }
    }
    
    fmt.Println("\nEndereços IPv4:")
    for _, ip := range ipv4List {
        fmt.Printf("  - %s\n", ip.String())
    }
    
    fmt.Println("\nEndereços IPv6:")
    for _, ip := range ipv6List {
        fmt.Printf("  - %s\n", ip.String())
    }
}
```

**Explicação:**
- `net.LookupHost()` retorna strings de endereços
- Você pode filtrar IPs por tipo checando `To4()`
- Útil quando sua aplicação suporta apenas IPv4 ou IPv6
- Alguns serviços têm apenas um tipo de endereço

### 3. Reverse DNS Lookup (IP para hostname)

```go
package main

import (
    "fmt"
    "net"
)

func reverseLookup(ip string) {
    fmt.Printf("\nReverse lookup para %s:\n", ip)
    
    names, err := net.LookupAddr(ip)
    if err != nil {
        fmt.Printf("  ❌ Erro: %v\n", err)
        return
    }
    
    if len(names) == 0 {
        fmt.Println("  ⚠️  Nenhum hostname encontrado")
        return
    }
    
    fmt.Println("  Hostnames encontrados:")
    for _, name := range names {
        fmt.Printf("    - %s\n", name)
    }
}

func main() {
    // IPs públicos conhecidos
    testIPs := []string{
        "8.8.8.8",           // Google Public DNS
        "1.1.1.1",           // Cloudflare DNS
        "208.67.222.222",    // OpenDNS
        "142.250.185.46",    // Google.com
    }
    
    fmt.Println("=== Reverse DNS Lookup ===")
    
    for _, ip := range testIPs {
        reverseLookup(ip)
    }
}
```

**Explicação:**
- `net.LookupAddr()` faz reverse lookup (IP → hostname)
- Nem todos os IPs têm registros PTR configurados
- Útil para identificar origem de conexões
- Servidores DNS e CDNs geralmente têm reverse records

### 4. Lookup de registros MX (Mail Exchange)

```go
package main

import (
    "fmt"
    "net"
    "sort"
)

func main() {
    domains := []string{
        "gmail.com",
        "outlook.com",
        "github.com",
    }
    
    for _, domain := range domains {
        fmt.Printf("\n=== Registros MX para %s ===\n", domain)
        
        mxRecords, err := net.LookupMX(domain)
        if err != nil {
            fmt.Printf("❌ Erro: %v\n", err)
            continue
        }
        
        if len(mxRecords) == 0 {
            fmt.Println("⚠️  Nenhum registro MX encontrado")
            continue
        }
        
        // Ordenar por prioridade (menor = mais prioritário)
        sort.Slice(mxRecords, func(i, j int) bool {
            return mxRecords[i].Pref < mxRecords[j].Pref
        })
        
        fmt.Println("Servidores de email (por prioridade):")
        for i, mx := range mxRecords {
            fmt.Printf("  %d. %s (prioridade: %d)\n", i+1, mx.Host, mx.Pref)
        }
    }
}
```

**Explicação:**
- Registros MX indicam servidores de email de um domínio
- `Pref` (preferência) indica prioridade: menor = mais prioritário
- Múltiplos registros MX fornecem redundância
- Útil para validar configuração de email de domínios

### 5. Lookup de Name Servers (NS)

```go
package main

import (
    "fmt"
    "net"
)

func main() {
    domains := []string{
        "google.com",
        "github.com",
        "cloudflare.com",
    }
    
    for _, domain := range domains {
        fmt.Printf("\n=== Name Servers de %s ===\n", domain)
        
        nsRecords, err := net.LookupNS(domain)
        if err != nil {
            fmt.Printf("❌ Erro: %v\n", err)
            continue
        }
        
        if len(nsRecords) == 0 {
            fmt.Println("⚠️  Nenhum name server encontrado")
            continue
        }
        
        fmt.Printf("Servidores DNS autoritativos (%d encontrados):\n", len(nsRecords))
        for i, ns := range nsRecords {
            fmt.Printf("  %d. %s\n", i+1, ns.Host)
        }
    }
}
```

**Explicação:**
- Registros NS identificam os servidores DNS autoritativos do domínio
- Esses servidores têm a informação oficial sobre o domínio
- Múltiplos NS fornecem redundância
- Útil para diagnosticar problemas de propagação DNS

### 6. Lookup de registros CNAME

```go
package main

import (
    "fmt"
    "net"
)

func lookupCNAME(hostname string) {
    fmt.Printf("\nVerificando CNAME para %s:\n", hostname)
    
    cname, err := net.LookupCNAME(hostname)
    if err != nil {
        fmt.Printf("  ❌ Erro: %v\n", err)
        return
    }
    
    // Se retorna o próprio hostname, não tem CNAME
    if cname == hostname+"." || cname == hostname {
        fmt.Println("  ℹ️  Não é um CNAME (é um registro A direto)")
    } else {
        fmt.Printf("  ➜ Aponta para: %s\n", cname)
    }
}

func main() {
    // Testar diferentes tipos de registros
    hosts := []string{
        "www.github.com",      // Geralmente é CNAME
        "github.com",          // Geralmente é A record
        "www.google.com",      // Pode ser CNAME
    }
    
    fmt.Println("=== Lookup de CNAME ===")
    
    for _, host := range hosts {
        lookupCNAME(host)
    }
}
```

**Explicação:**
- CNAME (Canonical Name) é um alias que aponta para outro domínio
- `www.example.com` frequentemente é CNAME para `example.com`
- Útil para CDNs e redirecionamentos
- Se não é CNAME, retorna o próprio hostname

### 7. Lookup com timeout customizado

```go
package main

import (
    "context"
    "fmt"
    "net"
    "time"
)

func lookupWithTimeout(hostname string, timeout time.Duration) {
    fmt.Printf("\nResolvendo %s (timeout: %v)...\n", hostname, timeout)
    
    // Criar resolver customizado
    resolver := &net.Resolver{
        PreferGo: true, // Usar implementação Go pura
    }
    
    // Context com timeout
    ctx, cancel := context.WithTimeout(context.Background(), timeout)
    defer cancel()
    
    start := time.Now()
    
    // Lookup com context
    ips, err := resolver.LookupIP(ctx, "ip4", hostname)
    
    duration := time.Since(start)
    
    if err != nil {
        // Verificar se foi timeout
        if ctx.Err() == context.DeadlineExceeded {
            fmt.Printf("  ⏱️  Timeout! Demorou mais que %v\n", timeout)
        } else {
            fmt.Printf("  ❌ Erro: %v\n", err)
        }
        return
    }
    
    fmt.Printf("  ✅ Resolvido em %v\n", duration)
    fmt.Printf("  Endereços encontrados:\n")
    for _, ip := range ips {
        fmt.Printf("    - %s\n", ip.String())
    }
}

func main() {
    hostname := "google.com"
    
    // Testar com diferentes timeouts
    lookupWithTimeout(hostname, 5*time.Second)
    lookupWithTimeout(hostname, 100*time.Millisecond)
    
    // Simular site lento ou inexistente
    lookupWithTimeout("site-muito-lento-que-nao-existe.com", 2*time.Second)
}
```

**Explicação:**
- Timeouts evitam que lookups DNS travem sua aplicação
- Use `context.WithTimeout()` para controlar tempo máximo
- `PreferGo: true` usa implementação Go ao invés do sistema operacional
- Essencial em produção para evitar bloqueios

### 8. Health check com DNS

```go
package main

import (
    "fmt"
    "net"
    "time"
)

type ServiceHealth struct {
    Hostname  string
    Available bool
    IPs       []string
    Latency   time.Duration
    Error     string
}

func checkServiceHealth(hostname string) ServiceHealth {
    health := ServiceHealth{
        Hostname: hostname,
    }
    
    start := time.Now()
    
    ips, err := net.LookupIP(hostname)
    
    health.Latency = time.Since(start)
    
    if err != nil {
        health.Available = false
        health.Error = err.Error()
        return health
    }
    
    health.Available = true
    for _, ip := range ips {
        health.IPs = append(health.IPs, ip.String())
    }
    
    return health
}

func main() {
    services := []string{
        "google.com",
        "github.com",
        "site-que-nao-existe-12345.com",
        "cloudflare.com",
    }
    
    fmt.Println("=== Health Check de Serviços ===\n")
    
    for _, service := range services {
        health := checkServiceHealth(service)
        
        if health.Available {
            fmt.Printf("✅ %s\n", health.Hostname)
            fmt.Printf("   Latência DNS: %v\n", health.Latency)
            fmt.Printf("   IPs: %v\n", health.IPs)
        } else {
            fmt.Printf("❌ %s\n", health.Hostname)
            fmt.Printf("   Erro: %s\n", health.Error)
        }
        fmt.Println()
    }
}
```

**Explicação:**
- Health checks DNS verificam se serviços estão resolvendo
- Medir latência ajuda a identificar problemas de rede
- Útil para monitoramento e alertas
- Pode ser executado periodicamente

### 9. Resolver DNS customizado

```go
package main

import (
    "context"
    "fmt"
    "net"
    "time"
)

func main() {
    // Criar resolver que usa servidores DNS específicos
    resolver := &net.Resolver{
        PreferGo: true,
        Dial: func(ctx context.Context, network, address string) (net.Conn, error) {
            d := net.Dialer{
                Timeout: 5 * time.Second,
            }
            // Usar Google Public DNS (8.8.8.8:53)
            return d.DialContext(ctx, network, "8.8.8.8:53")
        },
    }
    
    hostname := "github.com"
    
    fmt.Printf("Resolvendo %s usando Google DNS (8.8.8.8)...\n\n", hostname)
    
    ctx := context.Background()
    
    ips, err := resolver.LookupIP(ctx, "ip4", hostname)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    
    fmt.Println("Endereços encontrados:")
    for _, ip := range ips {
        fmt.Printf("  - %s\n", ip.String())
    }
    
    // Comparar com resolver padrão do sistema
    fmt.Println("\n--- Usando DNS do sistema ---")
    systemIPs, _ := net.LookupIP(hostname)
    for _, ip := range systemIPs {
        fmt.Printf("  - %s\n", ip.String())
    }
}
```

**Explicação:**
- Você pode especificar servidores DNS customizados
- Útil para usar DNS públicos (Google, Cloudflare, OpenDNS)
- Permite contornar restrições de DNS do provedor
- `PreferGo: true` ignora configuração do sistema operacional

---

## Best Practices

### ✅ Sempre configure timeouts
```go
resolver := &net.Resolver{
    PreferGo: true,
    Dial: func(ctx context.Context, network, address string) (net.Conn, error) {
        d := net.Dialer{
            Timeout: 5 * time.Second, // Nunca sem timeout!
        }
        return d.DialContext(ctx, network, address)
    },
}
```

### ✅ Cache resultados DNS
```go
// Cache simples de DNS
var dnsCache = make(map[string][]net.IP)

func lookupWithCache(hostname string) ([]net.IP, error) {
    if ips, found := dnsCache[hostname]; found {
        return ips, nil
    }
    
    ips, err := net.LookupIP(hostname)
    if err != nil {
        return nil, err
    }
    
    dnsCache[hostname] = ips
    return ips, nil
}
```

### ✅ Trate múltiplos IPs adequadamente
```go
ips, err := net.LookupIP(hostname)
if err != nil {
    return err
}

// Tentar todos os IPs até conseguir conectar
for _, ip := range ips {
    conn, err := net.Dial("tcp", ip.String()+":80")
    if err == nil {
        // Sucesso!
        return conn
    }
    // Tentar próximo IP
}
```

### ✅ Use context para cancelamento
```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

resolver := &net.Resolver{}
ips, err := resolver.LookupIP(ctx, "ip4", hostname)
```

### ✅ Log latências DNS para diagnóstico
```go
start := time.Now()
ips, err := net.LookupIP(hostname)
latency := time.Since(start)

if latency > 100*time.Millisecond {
    log.Printf("⚠️  DNS lento para %s: %v", hostname, latency)
}
```

---

## Erros Comuns a Evitar

### ❌ Não configurar timeouts
```go
// ERRADO - pode travar indefinidamente
ips, _ := net.LookupIP(hostname)
```

### ❌ Assumir que sempre haverá apenas um IP
```go
// ERRADO - ignora outros IPs
ips, _ := net.LookupIP(hostname)
ip := ips[0] // E se len(ips) == 0? Panic!
```

### ❌ Não tratar erros de DNS
```go
// ERRADO - ignora completamente erros
ips, _ := net.LookupIP(hostname)
// E se hostname não existir?
```

### ❌ Fazer lookups DNS em loops sem cache
```go
// ERRADO - muito ineficiente
for i := 0; i < 1000; i++ {
    ips, _ := net.LookupIP("same-host.com") // 1000 lookups!
    // Deveria cachear o resultado
}
```

### ❌ Não verificar se o slice está vazio
```go
ips, err := net.LookupIP(hostname)
if err != nil {
    return err
}
// Deveria verificar: if len(ips) == 0 {...}
firstIP := ips[0] // Pode causar panic!
```

---

## Padrões Comuns

### Pattern: Resolver com fallback
```go
func resolveWithFallback(hostname string) ([]net.IP, error) {
    // Tentar resolver normalmente
    ips, err := net.LookupIP(hostname)
    if err == nil && len(ips) > 0 {
        return ips, nil
    }
    
    // Fallback: usar DNS público
    resolver := &net.Resolver{
        PreferGo: true,
        Dial: func(ctx context.Context, network, address string) (net.Conn, error) {
            d := net.Dialer{Timeout: 3 * time.Second}
            return d.DialContext(ctx, network, "8.8.8.8:53")
        },
    }
    
    return resolver.LookupIP(context.Background(), "ip", hostname)
}
```

### Pattern: Validar domínio antes de usar
```go
func isDomainValid(domain string) bool {
    _, err := net.LookupIP(domain)
    return err == nil
}
```

### Pattern: Resolver com retry
```go
func lookupWithRetry(hostname string, maxRetries int) ([]net.IP, error) {
    var lastErr error
    
    for attempt := 0; attempt < maxRetries; attempt++ {
        ips, err := net.LookupIP(hostname)
        if err == nil && len(ips) > 0 {
            return ips, nil
        }
        
        lastErr = err
        time.Sleep(time.Second * time.Duration(attempt+1))
    }
    
    return nil, fmt.Errorf("falhou após %d tentativas: %w", maxRetries, lastErr)
}
```

### Pattern: DNS debug/diagnóstico completo
```go
func diagnoseDNS(hostname string) {
    fmt.Printf("=== Diagnóstico DNS: %s ===\n\n", hostname)
    
    // IPs
    if ips, err := net.LookupIP(hostname); err == nil {
        fmt.Println("✅ IPs:", ips)
    } else {
        fmt.Println("❌ IPs:", err)
    }
    
    // CNAME
    if cname, err := net.LookupCNAME(hostname); err == nil {
        fmt.Println("✅ CNAME:", cname)
    } else {
        fmt.Println("❌ CNAME:", err)
    }
    
    // MX
    if mx, err := net.LookupMX(hostname); err == nil {
        fmt.Println("✅ MX:", mx)
    } else {
        fmt.Println("❌ MX:", err)
    }
    
    // NS
    if ns, err := net.LookupNS(hostname); err == nil {
        fmt.Println("✅ NS:", ns)
    } else {
        fmt.Println("❌ NS:", err)
    }
}
```

---

**Fim do Tópico 3**

*Próximo: Tópico 4 - URIs*