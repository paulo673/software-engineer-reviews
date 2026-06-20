---
id: novidades-dotnet8-csharp12
title: "Novidades do .NET 8 e C# 12"
area: csharp-dotnet
difficulty: intermediario
prerequisites: [record-class-struct]
related: [garbage-collector, pipeline-middleware]
tags: [dotnet8, csharp12, linguagem, plataforma, novidades]
sources:
  - "https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12"
  - "https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-8/overview"
  - "https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#keyed-services"
status: revisado
last_updated: 2026-06-20
---

## Resumo

O .NET 8 é uma versão LTS (suporte de longo prazo) e o C# 12 é a linguagem que o acompanha. O C# 12 traz construtores primários para qualquer classe ou struct, expressões de coleção, parâmetros default em lambdas e aliases de qualquer tipo. O .NET 8 traz serviços com chave na injeção de dependência, `TimeProvider` para tempo testável, coleções congeladas e melhorias de Native AOT e GC. Importa saber o que cada um oferece para escrever código mais conciso e moderno.

## Explicação detalhada

### C# 12

**Construtores primários em qualquer tipo**: antes restritos a records, agora classes e structs comuns podem declarar parâmetros no cabeçalho do tipo. Esses parâmetros ficam disponíveis em todo o corpo, úteis para injeção de dependência sem campos repetitivos. Diferente do record, eles não geram propriedades públicas automaticamente nem igualdade por valor.

**Expressões de coleção**: sintaxe unificada `[...]` para criar arrays, listas, spans e outros tipos de coleção, com o operador spread `..` para concatenar sequências. Substitui várias formas antigas (`new[] { }`, `new List<int> { }`) por uma só.

**Parâmetros default em lambdas**: lambdas agora aceitam valores padrão e `params`, como métodos.

**Alias de qualquer tipo com `using`**: `using` pode dar nome a qualquer tipo, incluindo tuplas, arrays e tipos genéricos, não só tipos nomeados.

**`ref readonly` em parâmetros e inline arrays**: refinamentos para cenários de alto desempenho.

### .NET 8

**Keyed services na DI**: o contêiner nativo agora resolve serviços por chave, permitindo registrar várias implementações da mesma interface e escolher por nome. Usa `AddKeyedSingleton`, `AddKeyedScoped` e o atributo `[FromKeyedServices]`.

**`TimeProvider`**: uma abstração de tempo que torna código dependente de relógio testável. Em vez de `DateTime.UtcNow`, injeta-se um `TimeProvider` e nos testes usa-se um fake controlável.

**Coleções congeladas**: `FrozenDictionary<TKey, TValue>` e `FrozenSet<T>` são imutáveis e otimizadas para leitura muito mais rápida, ao custo de criação mais cara. Ideais para tabelas de lookup montadas uma vez e lidas muitas.

**Native AOT**: compilação ahead-of-time que gera um executável nativo, sem JIT em runtime, com inicialização mais rápida e menor footprint, útil para microsserviços e funções serverless.

**GC dinâmico (DATAS)**: ajuste dinâmico do tamanho do heap do Server GC conforme a carga, reduzindo o uso de memória.

## Por baixo dos panos

Construtores primários em classes não geram campos públicos: o compilador cria campos privados de apoio apenas para os parâmetros realmente usados fora do construtor, capturando-os. Isso difere do record, onde os parâmetros viram propriedades públicas.

Expressões de coleção têm tradução otimizada por destino: ao criar um `Span<T>` a partir de `[1, 2, 3]`, o compilador pode usar armazenamento na stack; ao criar uma `List<T>`, gera as chamadas de `Add` ou usa o builder apropriado. O tipo de destino guia a construção.

`FrozenDictionary` faz trabalho extra na construção (analisa as chaves para escolher a melhor estratégia de hashing e layout) para que cada leitura subsequente seja o mais rápida possível.

## Exemplos em C#

Construtor primário em classe, comum em serviços com DI:

```csharp
public class OrderService(IOrderRepository repository, ILogger<OrderService> logger)
{
    public async Task<Order> GetAsync(int id, CancellationToken ct)
    {
        logger.LogInformation("Buscando pedido {Id}", id);
        return await repository.FindAsync(id, ct)
            ?? throw new OrderNotFoundException(id);
    }
}
```

Expressões de coleção com spread:

```csharp
int[] primeiros = [1, 2, 3];
int[] seguintes = [4, 5];
int[] todos = [..primeiros, ..seguintes, 6];
```

Keyed services no .NET 8:

```csharp
builder.Services.AddKeyedSingleton<INotifier, EmailNotifier>("email");
builder.Services.AddKeyedSingleton<INotifier, SmsNotifier>("sms");

app.MapPost("/notify", (
    [FromKeyedServices("email")] INotifier notifier, string message) =>
{
    notifier.Send(message);
    return Results.Ok();
});
```

Tempo testável com `TimeProvider`:

```csharp
public class TokenService(TimeProvider time)
{
    public bool IsExpired(DateTimeOffset expiresAt) =>
        time.GetUtcNow() >= expiresAt;
}
```

## Tradeoffs

- Construtores primários reduzem boilerplate, mas escondem os campos e podem confundir quem espera o padrão clássico; em tipos complexos, o construtor tradicional ainda é mais claro.
- Expressões de coleção são concisas, porém o tipo de destino precisa estar claro para o compilador escolher a construção certa.
- Native AOT acelera startup e reduz memória, mas impõe restrições: reflexão limitada, sem geração de código em runtime, e nem toda biblioteca é compatível.
- Coleções congeladas trocam custo de criação por leitura rápida: só compensam quando a coleção é montada uma vez e lida muitas.

## Pegadinhas e erros comuns

- Achar que o construtor primário de uma classe gera propriedades públicas como no record: ele não gera; os parâmetros ficam acessíveis no corpo, mas não viram membros públicos automaticamente.
- Esperar igualdade por valor de uma classe só por usar construtor primário: isso é do record, não da classe.
- Adotar Native AOT sem checar compatibilidade das dependências e o uso de reflexão: pode quebrar em runtime.
- Usar `FrozenDictionary` para coleções que mudam: ela é imutável e cara de recriar.
- Continuar usando `DateTime.UtcNow` direto e depois sofrer para testar lógica dependente de tempo, quando `TimeProvider` resolveria.

## Quando usar e quando evitar

Use construtor primário em serviços e tipos simples para enxugar a injeção de dependência. Use expressões de coleção como sintaxe padrão de criação. Use keyed services quando houver múltiplas implementações da mesma interface selecionadas por contexto. Use `TimeProvider` em qualquer lógica dependente de relógio que precise ser testada. Use Native AOT em microsserviços e funções onde startup e footprint importam e as dependências são compatíveis. Evite forçar os recursos novos onde a forma clássica é mais legível.

## Perguntas de auto-teste

1. Construtor primário em uma classe gera propriedades públicas?
<details><summary>Resposta</summary>Não. Em classe ele apenas disponibiliza os parâmetros no corpo (capturados em campos privados conforme o uso). Geração de propriedades públicas e igualdade por valor são comportamento de record.</details>

2. O que faz o operador `..` em expressões de coleção?
<details><summary>Resposta</summary>É o spread: insere os elementos de outra sequência dentro da nova coleção, permitindo concatenar.</details>

3. Para que serve `TimeProvider` no .NET 8?
<details><summary>Resposta</summary>Para abstrair o acesso ao tempo (relógio, timers) e tornar testável a lógica dependente de tempo, substituindo chamadas diretas como DateTime.UtcNow.</details>

4. O que são keyed services e quando usar?
<details><summary>Resposta</summary>Registro e resolução de serviços por chave no contêiner de DI, úteis quando há várias implementações da mesma interface e se escolhe uma por nome.</details>

5. Qual o tradeoff de `FrozenDictionary`?
<details><summary>Resposta</summary>Construção mais cara em troca de leituras muito mais rápidas; vale para coleções imutáveis montadas uma vez e lidas muitas.</details>

6. Que restrição importante o Native AOT impõe?
<details><summary>Resposta</summary>Reflexão limitada e ausência de geração de código em runtime, exigindo que as dependências sejam compatíveis com AOT.</details>

## Referências

- [What's new in C# 12](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12)
- [What's new in .NET 8](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-8/overview)
- [Keyed services in dependency injection](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#keyed-services)
