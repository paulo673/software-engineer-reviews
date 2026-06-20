# 03 - EF Core, Dapper e PostgreSQL

Acesso a dados em .NET: do change tracking do EF Core ao controle fino de SQL com Dapper, com PostgreSQL como database de referência.

## Ordem de leitura sugerida

| # | Tópico | Dificuldade | Descrição |
| --- | --- | --- | --- |
| 1 | [Essential SQL](essential-sql.md) | beginner | JOINs, aggregation, indexes e execution plan |
| 2 | [Change tracking](change-tracking.md) | intermediate | Como o EF Core rastreia mudanças e impacto em performance |
| 3 | [N+1 Problem](n-plus-one-problem.md) | intermediate | Causa, detecção e correção com eager e split queries |
| 4 | [Migrations](migrations.md) | intermediate | Schema versioning com EF Core |
| 5 | [Dapper vs EF Core](dapper-vs-ef.md) | intermediate | Micro ORM versus ORM completo, quando usar cada um |
| 6 | [JSONB Indexes](jsonb-indexes.md) | intermediate | Index types no PostgreSQL e queries em JSONB |
| 7 | [Transaction Isolation](transaction-isolation.md) | advanced | Anomalias, isolation levels e MVCC no PostgreSQL |
