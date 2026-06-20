---
id: change-tracking
title: "Change Tracking in EF Core"
area: ef-dapper-postgresql
difficulty: intermediate
prerequisites: [essential-sql]
related: [n-plus-one-problem, dapper-vs-ef, iqueryable-vs-ienumerable]
tags: [ef-core, change-tracking, no-tracking, performance, orm]
sources:
  - "https://learn.microsoft.com/en-us/ef/core/change-tracking/"
  - "https://learn.microsoft.com/en-us/ef/core/querying/tracking"
status: reviewed
last_updated: 2026-06-20
---

## Resumo

Change tracking é o mecanismo do EF Core que monitora as entidades carregadas por um `DbContext` para descobrir, no `SaveChanges`, o que mudou e gerar os comandos SQL de `INSERT`, `UPDATE` e `DELETE` apropriados. Ele dá a conveniência de "alterar o objeto e salvar", mas tem custo de memória e CPU. Saber quando desligá-lo (consultas só de leitura) é uma das otimizações mais comuns em EF Core.

## Explicação detalhada

Quando você consulta entidades com tracking (o padrão), o `DbContext` guarda uma referência a cada entidade e um snapshot dos seus valores originais. Cada entidade tem um estado no tracker:

- **Added**: nova, será inserida.
- **Unchanged**: carregada e não modificada.
- **Modified**: teve propriedades alteradas, será atualizada.
- **Deleted**: marcada para remoção.
- **Detached**: não rastreada pelo contexto.

No `SaveChanges`, o EF compara os valores atuais com o snapshot original (snapshot change tracking) para detectar o que mudou e gera apenas os comandos necessários. É isso que permite carregar uma entidade, mudar uma propriedade e chamar `SaveChanges` sem escrever SQL.

**No-tracking**: para consultas que só leem e exibem dados, rastrear é desperdício. `AsNoTracking()` diz ao EF para não guardar snapshot nem referência, o que reduz memória e acelera a materialização. A entidade retornada não é rastreada, então alterá-la não afeta o database.

**Identity resolution**: com tracking, se a mesma linha (mesma chave) aparece duas vezes na consulta, o EF retorna a mesma instância (resolve a identidade). Com `AsNoTracking` simples, pode retornar instâncias distintas para a mesma linha; existe `AsNoTrackingWithIdentityResolution` quando você precisa de no-tracking mas com instâncias unificadas.

## Por baixo dos panos

O `ChangeTracker` mantém entradas (`EntityEntry`) para cada entidade rastreada, cada uma com os valores originais. A detecção de mudanças (`DetectChanges`) percorre as entidades comparando atuais com originais; ela é chamada automaticamente antes de `SaveChanges` e de algumas operações de consulta. Em grafos grandes, esse percurso custa, o que explica parte do overhead.

`AsNoTracking` pula a criação de `EntityEntry` e do snapshot, então cada linha vira só o objeto materializado. Por isso consultas de leitura em listas grandes ganham bastante com no-tracking.

O EF também faz **fix-up** de navegação: ao rastrear entidades relacionadas, ele conecta as referências de navegação entre elas com base nas chaves. Isso só acontece com tracking, e é por isso que entidades no-tracking podem ter navegações não preenchidas a menos que carregadas explicitamente.

## Exemplos em C#

Atualização baseada em tracking, sem escrever UPDATE:

```csharp
public async Task RenameAsync(int id, string newName, CancellationToken ct)
{
    var product = await _db.Products.FirstAsync(p => p.Id == id, ct);
    product.Name = newName;
    await _db.SaveChangesAsync(ct);
}
```

O EF detecta que só `Name` mudou e gera um `UPDATE` apenas dessa coluna.

Leitura otimizada com no-tracking:

```csharp
public async Task<List<ProductDto>> ListAsync(CancellationToken ct)
{
    return await _db.Products
        .AsNoTracking()
        .Where(p => p.IsActive)
        .Select(p => new ProductDto(p.Id, p.Name, p.Price))
        .ToListAsync(ct);
}
```

No-tracking por padrão em todo o contexto, ligando tracking só onde precisa:

```csharp
_db.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;
```

## Tradeoffs

- Tracking dá produtividade: o modelo de "carregar, alterar, salvar" elimina SQL manual e detecta o delta. O custo é memória (snapshots) e CPU (detecção de mudanças), proporcional ao número e tamanho das entidades.
- No-tracking acelera leituras e reduz memória, ideal para consultas de exibição, mas a entidade não é atualizável pelo contexto e navegações podem não ser conectadas.
- Projeções com `Select` para DTOs evitam carregar entidades inteiras e já dispensam tracking do agregado, sendo muitas vezes a melhor escolha para leitura.

## Pegadinhas e erros comuns

- Usar tracking em consultas de leitura de listas grandes: desperdício de memória e CPU. Adicione `AsNoTracking`.
- Esperar que alterar uma entidade `AsNoTracking` persista no `SaveChanges`: não persiste, pois ela não é rastreada.
- `DbContext` de vida longa acumulando entidades rastreadas: o tracker cresce, consome memória e degrada `DetectChanges`. Use contexto de vida curta (escopo de requisição).
- Reutilizar o mesmo contexto entre requisições concorrentes: `DbContext` não é thread-safe.
- Esquecer que `SaveChanges` chama `DetectChanges`: em loops com muitos `SaveChanges`, o custo se acumula; agrupe as alterações.
- Contar com identity resolution em no-tracking simples: pode haver instâncias duplicadas da mesma linha.

## Quando usar e quando evitar

Use tracking quando vai modificar e salvar entidades, aproveitando a detecção automática de mudanças. Use `AsNoTracking` (ou projeções para DTO) em toda consulta puramente de leitura, especialmente listas e relatórios. Mantenha o `DbContext` de vida curta. Evite tracking em caminhos de leitura quentes e evite contextos longos que acumulam entidades.

## Perguntas de auto-teste

1. O que o change tracking permite fazer sem escrever SQL?
<details><summary>Resposta</summary>Carregar uma entidade, alterar suas propriedades e chamar SaveChanges; o EF detecta o que mudou comparando com o snapshot e gera o UPDATE adequado.</details>

2. Quando usar `AsNoTracking`?
<details><summary>Resposta</summary>Em consultas puramente de leitura (exibição, listas, relatórios), onde rastrear é desperdício de memória e CPU e a entidade não será modificada e salva.</details>

3. Quais são os estados de uma entidade no change tracker?
<details><summary>Resposta</summary>Added, Unchanged, Modified, Deleted e Detached.</details>

4. Por que um `DbContext` de vida longa é problemático?
<details><summary>Resposta</summary>Porque acumula entidades rastreadas, crescendo o uso de memória e tornando a detecção de mudanças mais lenta. Além disso o DbContext não é thread-safe. Prefira contexto de vida curta.</details>

5. Alterar uma entidade obtida com `AsNoTracking` e chamar `SaveChanges` persiste a mudança?
<details><summary>Resposta</summary>Não, porque a entidade não está sendo rastreada pelo contexto, então a alteração não é detectada nem persistida.</details>

6. O que é identity resolution no tracking?
<details><summary>Resposta</summary>Quando a mesma linha (mesma chave) aparece mais de uma vez na consulta, o tracker retorna a mesma instância. Em no-tracking simples isso não acontece por padrão.</details>

## Diagrama

```mermaid
flowchart LR
    Q[Consulta com tracking] --> SNAP[Snapshot dos valores originais]
    M[Alteração na entidade] --> DC[DetectChanges compara atual x original]
    SNAP --> DC
    DC --> SC[SaveChanges gera INSERT/UPDATE/DELETE do delta]
```

## Referências

- [Change Tracking in EF Core](https://learn.microsoft.com/en-us/ef/core/change-tracking/)
- [Tracking vs. No-Tracking Queries](https://learn.microsoft.com/en-us/ef/core/querying/tracking)
