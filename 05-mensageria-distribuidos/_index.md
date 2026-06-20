# 05 - Mensageria e Sistemas Distribuídos

Comunicação assíncrona e os limites fundamentais de sistemas distribuídos: garantias de entrega, brokers e teoremas que regem o tradeoff entre consistência e disponibilidade.

## Ordem de leitura sugerida

| # | Tópico | Dificuldade | Descrição |
| --- | --- | --- | --- |
| 1 | [Dead letter queue](dead-letter-queue.md) | intermediario | Tratamento de mensagens que falham repetidamente |
| 2 | [RabbitMQ vs Kafka](rabbitmq-vs-kafka.md) | intermediario | Broker de filas versus log distribuído |
| 3 | [Semântica de entrega](semantica-entrega.md) | avancado | At-most-once, at-least-once, exactly-once e idempotência |
| 4 | [Teorema CAP](teorema-cap.md) | avancado | Consistência, disponibilidade e tolerância a partição |
