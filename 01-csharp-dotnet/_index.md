# 01 - C# e .NET

Fundamentos da linguagem C# e do runtime .NET, do modelo async à coleta de lixo. Base para todas as outras áreas.

## Ordem de leitura sugerida

| # | Tópico | Dificuldade | Descrição |
| --- | --- | --- | --- |
| 1 | [record, class and struct](record-class-struct.md) | beginner | Reference types versus value types, equality e immutability |
| 2 | [Generics and Variance](generics-variance.md) | intermediate | Type parameters, constraints, covariance e contravariance |
| 3 | [Pipeline and Middleware](pipeline-middleware.md) | intermediate | Request pipeline do ASP.NET Core e ordem dos middlewares |
| 4 | [.NET 8 and C# 12 New Features](dotnet8-csharp12-new-features.md) | intermediate | Principais new features da linguagem e da plataforma |
| 5 | [Garbage Collector](garbage-collector.md) | intermediate | Generations, heaps, modos do GC e impacto em performance |
| 6 | [Threads, ThreadPool, Task and async/await](threads-threadpool.md) | advanced | Funcionamento de threads, ThreadPool e relação com Task, ValueTask e await |
| 7 | [Concurrency, Parallelism and Race Conditions](concurrency-parallelism.md) | advanced | Concurrency vs parallelism, sync vs async, race conditions e synchronization |
| 8 | [async and await](async-await.md) | advanced | Modelo async, SynchronizationContext, ConfigureAwait, deadlock |
| 9 | [Task vs ValueTask](task-vs-valuetask.md) | advanced | Custo de allocation e quando a ValueTask compensa |
| 10 | [IQueryable vs IEnumerable](iqueryable-vs-ienumerable.md) | advanced | Expression tree, deferred execution e onde o filtro roda |
