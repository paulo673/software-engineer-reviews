# 03 - EF Core, Dapper e PostgreSQL

Acesso a dados em .NET: do change tracking do EF Core ao controle fino de SQL com Dapper, com PostgreSQL como banco de referência.

## Ordem de leitura sugerida

| # | Tópico | Dificuldade | Descrição |
| --- | --- | --- | --- |
| 1 | [SQL essencial](sql-essencial.md) | iniciante | JOINs, agregação, índices e plano de execução |
| 2 | [Change tracking](change-tracking.md) | intermediario | Como o EF Core rastreia mudanças e impacto em performance |
| 3 | [Problema N+1](problema-n-mais-1.md) | intermediario | Causa, detecção e correção com eager e split queries |
| 4 | [Migrations](migrations.md) | intermediario | Versionamento de schema com EF Core |
| 5 | [Dapper vs EF Core](dapper-vs-ef.md) | intermediario | Micro ORM versus ORM completo, quando usar cada um |
| 6 | [Índices e JSONB](indices-jsonb.md) | intermediario | Tipos de índice no PostgreSQL e consultas em JSONB |
| 7 | [Isolamento de transação](isolamento-transacao.md) | avancado | Anomalias, níveis de isolamento e MVCC no PostgreSQL |
