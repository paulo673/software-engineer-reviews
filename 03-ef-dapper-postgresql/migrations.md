---
id: migrations
title: "Migrations no EF Core"
area: ef-dapper-postgresql
difficulty: intermediario
prerequisites: [change-tracking]
related: [dapper-vs-ef, ci-cd]
tags: [ef-core, migrations, schema, versionamento, banco]
sources:
  - "https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/"
  - "https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying"
status: revisado
last_updated: 2026-06-20
---

## Resumo

Migrations são o mecanismo do EF Core para versionar o schema do banco em código, em sincronia com o modelo de entidades. Cada migration descreve as mudanças incrementais (criar tabela, adicionar coluna, índice) e pode ser aplicada para frente ou revertida. Importam porque mantêm o schema sob controle de versão, reprodutível entre ambientes e integrável ao pipeline de CI/CD, em vez de alterações manuais e divergentes no banco.

## Explicação detalhada

O EF Core compara o modelo atual (suas entidades e configurações) com um snapshot do modelo da última migration e gera o diff como uma nova migration. Cada migration tem dois métodos: `Up`, que aplica as mudanças, e `Down`, que as reverte.

Fluxo típico com a CLI:

- `dotnet ef migrations add NomeDaMigration`: gera a migration a partir das mudanças no modelo.
- `dotnet ef database update`: aplica as migrations pendentes ao banco.
- `dotnet ef migrations remove`: remove a última migration ainda não aplicada.
- `dotnet ef migrations script`: gera um script SQL idempotente para aplicar em produção.

O EF mantém no banco uma tabela `__EFMigrationsHistory` com as migrations já aplicadas, para saber o que falta. É assim que ele sabe quais migrations rodar em cada ambiente.

**Aplicação em produção**: há três caminhos principais.

- **Script SQL** gerado e revisado, aplicado pelo pipeline. É o mais seguro e auditável, recomendado para produção.
- **`dotnet ef database update`** ou o bundle de migrations executado no deploy.
- **`context.Database.Migrate()`** na inicialização da aplicação. Cômodo, mas arriscado com múltiplas instâncias subindo ao mesmo tempo (corrida) e mistura responsabilidades de schema com runtime da aplicação.

## Por baixo dos panos

Cada migration carrega um snapshot do modelo (`ModelSnapshot`), uma representação completa do schema esperado após aquela migration. O diff é calculado entre o modelo das entidades e esse snapshot, não lendo o banco. Por isso o snapshot precisa estar versionado e correto: se ele divergir do banco real, o diff sai errado.

O EF traduz as operações de migration (criar tabela, coluna, índice) para o DDL do provedor (no PostgreSQL, via Npgsql). Nem toda mudança é detectável ou segura automaticamente: renomear uma coluna pode ser interpretado como drop e add (perdendo dados) se você não ajustar a migration manualmente. Migrations de dados (mover ou transformar valores) frequentemente exigem SQL escrito à mão dentro do `Up`.

O script idempotente gerado verifica a `__EFMigrationsHistory` para aplicar só o que falta, podendo ser rodado com segurança em qualquer ambiente independentemente do estado atual.

## Exemplos em C#

Configuração que uma migration materializa:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>(entity =>
    {
        entity.HasKey(p => p.Id);
        entity.Property(p => p.Name).HasMaxLength(200).IsRequired();
        entity.HasIndex(p => p.Sku).IsUnique();
    });
}
```

Migration com transformação de dados manual no `Up`:

```csharp
public partial class BackfillStatus : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "status", table: "orders", nullable: false, defaultValue: "pending");

        migrationBuilder.Sql(
            "UPDATE orders SET status = 'completed' WHERE shipped_at IS NOT NULL;");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(name: "status", table: "orders");
    }
}
```

Gerar script idempotente para o pipeline:

```bash
dotnet ef migrations script --idempotent --output migrations.sql
```

## Tradeoffs

- Migrations dão schema versionado, reprodutível e revisável, integrável ao CI/CD. O custo é disciplina: revisar cada migration gerada, especialmente quando há risco de perda de dados.
- Aplicar via script revisado é seguro e auditável, mas exige um passo no pipeline. `Database.Migrate()` na inicialização é cômodo, porém arriscado em escala (corrida entre instâncias, acoplamento de schema ao boot).
- Migrations automáticas economizam escrita, mas não capturam toda intenção (renomeações, transformações de dados), exigindo ajuste manual.

## Pegadinhas e erros comuns

- Editar o banco manualmente e deixar o snapshot divergir: o próximo diff fica errado. Toda mudança de schema deve passar por migration.
- Renomear coluna ou tabela esperando que o EF preserve dados: ele pode gerar drop e add. Ajuste a migration para usar rename.
- Rodar `Database.Migrate()` em várias instâncias simultâneas: corrida ao aplicar a mesma migration. Aplique no pipeline ou serialize a aplicação.
- Não revisar a migration gerada antes de aplicar em produção: operações destrutivas podem passar despercebidas.
- Apagar ou editar uma migration já aplicada em outro ambiente: quebra o histórico. Crie uma nova migration corretiva.
- Esquecer o `Down` ou deixá-lo inconsistente, impossibilitando reverter.

## Quando usar e quando evitar

Use migrations do EF Core sempre que o EF for o dono do schema, mantendo cada mudança versionada e aplicando em produção via script idempotente revisado no pipeline. Para transformações de dados, escreva SQL explícito na migration. Evite alterações manuais no banco fora das migrations, e em equipes onde DBAs controlam o schema com outra ferramenta, alinhe o processo para não haver duas fontes da verdade. Evite `Database.Migrate()` no boot em produção com múltiplas instâncias.

## Perguntas de auto-teste

1. O que o EF Core compara para gerar uma nova migration?
<details><summary>Resposta</summary>O modelo atual das entidades com o snapshot do modelo da última migration (não lê o banco). O diff vira a nova migration.</details>

2. Para que serve a tabela `__EFMigrationsHistory`?
<details><summary>Resposta</summary>Registra quais migrations já foram aplicadas no banco, permitindo ao EF saber o que ainda falta aplicar em cada ambiente.</details>

3. Qual a forma mais segura de aplicar migrations em produção?
<details><summary>Resposta</summary>Gerar um script SQL idempotente, revisá-lo e aplicá-lo pelo pipeline, em vez de migrar automaticamente na inicialização da aplicação.</details>

4. Por que renomear uma coluna pode causar perda de dados?
<details><summary>Resposta</summary>Porque o EF pode interpretar a mudança como drop da coluna antiga e add de uma nova, descartando os dados. É preciso ajustar a migration para usar rename.</details>

5. Qual o risco de `Database.Migrate()` na inicialização com várias instâncias?
<details><summary>Resposta</summary>Corrida: múltiplas instâncias tentando aplicar a mesma migration ao mesmo tempo, com risco de erro ou inconsistência. Além de acoplar schema ao boot da aplicação.</details>

6. O que fazer ao precisar transformar dados existentes numa migration?
<details><summary>Resposta</summary>Escrever SQL explícito no método Up (via migrationBuilder.Sql), pois o diff automático não captura transformações de dados.</details>

## Diagrama

```mermaid
flowchart LR
    M[Modelo de entidades] -->|diff com snapshot| MIG[Nova migration: Up / Down]
    MIG -->|migrations script idempotente| SQL[Script revisado]
    SQL -->|pipeline CI/CD| DB[(Banco)]
    DB --> H[__EFMigrationsHistory]
```

## Referências

- [Migrations Overview (EF Core)](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/)
- [Applying Migrations (EF Core)](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying)
