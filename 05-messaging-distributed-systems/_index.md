# 05 - Messaging and Distributed Systems

Comunicação async e os limites fundamentais de distributed systems: delivery guarantees, brokers e teoremas que regem o tradeoff entre consistency e availability.

## Ordem de leitura sugerida

| # | Tópico | Dificuldade | Descrição |
| --- | --- | --- | --- |
| 1 | [Dead letter queue](dead-letter-queue.md) | intermediate | Tratamento de mensagens que falham repetidamente |
| 2 | [RabbitMQ vs Kafka](rabbitmq-vs-kafka.md) | intermediate | Broker de filas versus log distribuído |
| 3 | [Delivery Semantics](delivery-semantics.md) | advanced | At-most-once, at-least-once, exactly-once e idempotency |
| 4 | [CAP Theorem](cap-theorem.md) | advanced | Consistency, availability e partition tolerance |
