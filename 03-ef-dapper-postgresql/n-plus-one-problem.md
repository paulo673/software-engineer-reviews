---
id: n-plus-one-problem
title: "N+1 Problem in EF Core"
area: ef-dapper-postgresql
difficulty: intermediate
prerequisites: [change-tracking]
related: [change-tracking, iqueryable-vs-ienumerable, dapper-vs-ef]
tags: [ef-core, n-plus-one, performance, eager-loading, lazy-loading]
sources:
  - "https://learn.microsoft.com/en-us/ef/core/querying/related-data/"
  - "https://learn.microsoft.com/en-us/ef/core/querying/single-split-queries"
status: reviewed
last_updated: 2026-06-20
---

## Resumo

O problema N+1 acontece quando, para carregar uma lista de N entidades e seus relacionamentos, o código dispara 1 consulta para a lista mais N consultas adicionais, uma por entidade, para buscar os dados relacionados. O resultado é dezenas ou centenas de idas ao database onde bastariam uma ou duas. É uma das causas mais comuns de lentidão com ORMs, e a correção é carregar os relacionamentos de forma antecipada.

## Explicação detalhada

Imagine listar 100 pedidos e, para cada um, acessar o cliente. Se o cliente é carregado sob demanda, ocorre: 1 consulta para os 100 pedidos, depois 1 consulta para o cliente de cada pedido, totalizando 1 + 100 = 101 consultas. Daí o nome N+1.

As causas no EF Core:

- **Lazy loading**: se proxies de lazy loading estão habilitados, acessar uma navegação (`order.Customer`) dispara uma consulta naquele instante. Em um loop, vira N+1 silenciosamente.
- **Carregar e iterar acessando navegações** sem ter incluído os relacionados: mesmo sem lazy loading, padrões de código que consultam dentro do loop reproduzem o problema.

As correções:

- **Eager loading com `Include`**: pede ao EF para trazer os relacionados na mesma consulta, via `JOIN`. Uma consulta em vez de N+1.
- **Projeção com `Select`**: trazer só os campos necessários de pedido e cliente em um DTO, também em uma consulta, frequentemente a opção mais eficiente.
- **Split queries**: quando o `Include` traz muitas coleções e o `JOIN` causa explosão cartesiana (linhas duplicadas), `AsSplitQuery` divide em uma consulta por coleção, evitando o produto cartesiano ao custo de mais idas ao database (mas poucas, não N+1).

## Por baixo dos panos

Com um `Include` de uma navegação de referência (muitos para um, como pedido para cliente), o EF gera um único `LEFT JOIN`. Com `Include` de coleções (um para muitos), um único `JOIN` multiplica as linhas: se cada pedido tem vários itens e você inclui itens e pagamentos, o resultado é o produto cartesiano das duas coleções, inflando o volume transferido. Isso é a "explosão cartesiana".

`AsSplitQuery` resolve emitindo uma consulta para os pedidos e uma consulta separada para cada coleção relacionada, usando os ids carregados. Troca o produto cartesiano por algumas consultas adicionais. O EF então faz o fix-up das navegações em memória (ver [change tracking](change-tracking.md)).

Projeções com `Select` geram exatamente o SQL para os campos pedidos, sem materializar entidades inteiras nem rastreá-las, sendo geralmente o caminho mais enxuto para leitura.

## Exemplos em C#

Código que causa N+1 (acessando navegação no loop):

```csharp
var orders = await _db.Orders.ToListAsync(ct);
foreach (var order in orders)
{
    Console.WriteLine(order.Customer.Name);
}
```

Se lazy loading estiver ligado, cada `order.Customer` dispara uma consulta.

Correção com eager loading:

```csharp
var orders = await _db.Orders
    .Include(o => o.Customer)
    .ToListAsync(ct);
```

Correção com projeção, trazendo só o necessário:

```csharp
var summaries = await _db.Orders
    .Select(o => new OrderSummary(o.Id, o.Customer.Name, o.Total))
    .ToListAsync(ct);
```

Múltiplas coleções com split query para evitar explosão cartesiana:

```csharp
var orders = await _db.Orders
    .Include(o => o.Items)
    .Include(o => o.Payments)
    .AsSplitQuery()
    .ToListAsync(ct);
```

## Tradeoffs

- `Include` resolve N+1 com uma consulta, mas incluir várias coleções num único join causa explosão cartesiana; aí `AsSplitQuery` é melhor.
- Split query evita o produto cartesiano com poucas consultas extras, ao custo de não ser uma única ida ao database e de possível inconsistência se os dados mudarem entre as consultas (sem transação).
- Projeção com `Select` costuma ser a mais eficiente para leitura, mas exige definir DTOs e não retorna entidades rastreadas para edição.
- Lazy loading é conveniente para escrever, mas é a fonte clássica de N+1 acidental; muitos times o mantêm desligado.

## Pegadinhas e erros comuns

- Lazy loading habilitado e navegação acessada em loop: N+1 silencioso, difícil de notar sem olhar os logs de SQL.
- Resolver N+1 com `Include` de várias coleções e criar explosão cartesiana, transferindo um volume enorme de linhas duplicadas.
- Filtrar ou paginar depois de materializar (`ToList` cedo): além de N+1, traz tudo para a memória (ver [IQueryable vs IEnumerable](../01-csharp-dotnet/iqueryable-vs-ienumerable.md)).
- Não olhar o SQL gerado: a única forma confiável de detectar N+1 é inspecionar as consultas (logging do EF, tools de profiling).
- Usar `Include` quando só precisa de alguns campos: projeção seria mais leve.
- Esperar consistência entre as consultas de um split query sem envolvê-las em transação.

## Quando usar e quando evitar

Use `Include` para carregar relacionamentos necessários em uma consulta quando há uma ou poucas navegações. Use `AsSplitQuery` quando incluir múltiplas coleções que causariam explosão cartesiana. Use projeção com `Select` para leitura quando não precisa da entidade inteira, que é o caminho mais eficiente. Mantenha o logging de SQL ativo em desenvolvimento para flagrar N+1. Evite lazy loading em caminhos de listagem e evite acessar navegações dentro de loops sem ter carregado os dados.

## Perguntas de auto-teste

1. Por que o problema se chama N+1?
<details><summary>Resposta</summary>Porque carrega-se a lista com 1 consulta e, para cada uma das N entidades, dispara-se mais 1 consulta para os dados relacionados, totalizando N+1 idas ao database.</details>

2. Qual a causa mais comum de N+1 acidental no EF Core?
<details><summary>Resposta</summary>Lazy loading habilitado, com acesso a uma navegação dentro de um loop, disparando uma consulta por iteração.</details>

3. Como `Include` resolve o N+1?
<details><summary>Resposta</summary>Trazendo os dados relacionados na mesma consulta via JOIN, reduzindo de N+1 para 1 ida ao database.</details>

4. O que é explosão cartesiana e quando ocorre?
<details><summary>Resposta</summary>Quando se incluem múltiplas coleções num único JOIN, o resultado é o produto cartesiano delas, multiplicando e duplicando linhas transferidas. Ocorre com vários Include de coleções.</details>

5. O que `AsSplitQuery` faz e qual o tradeoff?
<details><summary>Resposta</summary>Divide a consulta em uma por coleção, evitando o produto cartesiano, ao custo de algumas consultas extras e possível inconsistência se os dados mudarem entre elas sem transação.</details>

6. Como detectar N+1 com confiança?
<details><summary>Resposta</summary>Inspecionando o SQL gerado via logging do EF ou tools de profiling; é a única forma segura de ver as consultas repetidas.</details>

## Diagrama

```mermaid
flowchart TD
    subgraph Problema
    A[1 consulta: lista de N pedidos] --> B[N consultas: cliente de cada pedido]
    end
    subgraph Solução
    C[1 consulta com Include ou Select] --> D[Pedidos + relacionados juntos]
    end
```

## Referências

- [Loading Related Data (EF Core)](https://learn.microsoft.com/en-us/ef/core/querying/related-data/)
- [Single vs. Split Queries (EF Core)](https://learn.microsoft.com/en-us/ef/core/querying/single-split-queries)
