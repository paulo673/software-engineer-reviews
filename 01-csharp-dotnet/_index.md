# 01 - C# e .NET

Fundamentos da linguagem C# e do runtime .NET, do modelo assíncrono à coleta de lixo. Base para todas as outras áreas.

## Ordem de leitura sugerida

| # | Tópico | Dificuldade | Descrição |
| --- | --- | --- | --- |
| 1 | [record, class e struct](record-class-struct.md) | iniciante | Tipos de referência versus valor, igualdade e imutabilidade |
| 2 | [Generics e variância](generics-variancia.md) | intermediario | Parâmetros de tipo, restrições, covariância e contravariância |
| 3 | [Pipeline e middleware](pipeline-middleware.md) | intermediario | Pipeline de requisição do ASP.NET Core e ordem dos middlewares |
| 4 | [Novidades do .NET 8 e C# 12](novidades-dotnet8-csharp12.md) | intermediario | Principais recursos novos da linguagem e da plataforma |
| 5 | [Garbage Collector](garbage-collector.md) | intermediario | Gerações, heaps, modos do GC e impacto em performance |
| 6 | [async e await](async-await.md) | avancado | Modelo assíncrono, SynchronizationContext, ConfigureAwait, deadlock |
| 7 | [Task vs ValueTask](task-vs-valuetask.md) | avancado | Custo de alocação e quando ValueTask compensa |
| 8 | [IQueryable vs IEnumerable](iqueryable-vs-ienumerable.md) | avancado | Árvore de expressão, deferred execution e onde o filtro roda |
