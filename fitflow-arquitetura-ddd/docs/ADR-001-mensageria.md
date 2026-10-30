# ADR-001 - Mensageria do FitFlow (proposta)

**Status:** proposta; nao implementado na N9.

## Contexto
Publicar um treino deve continuar rapido mesmo quando futuras notificacoes ou integracoes estao lentas.

## Decisao
Usar RabbitMQ como broker, exchange do tipo `topic` (`fitflow.events`) e a routing key `workout.published.v1`, com fila duravel `notifications.workout-published`. O modulo de treinos publica um evento apos a confirmacao da transacao; um worker isolado consome e envia a notificacao.

## Integridade e operacao
Em evolucao de producao, aplicar **Transactional Outbox**: gravar evento na mesma transacao do treino e publicar de modo confiavel a partir da outbox; so emitir evento apos commit. Consumidores idempotentes por `eventId`, confirmacao manual (ACK), retentativas com limite, dead-letter queue e logs com correlacao. Eventos minimizam dados pessoais. Monitorar backlog e falhas de entrega. Se a publicacao falhar, preservar o treino e reprocessar o evento da outbox.

## Alternativas
HTTP sincronico: mais simples, mas acopla disponibilidade e latencia. Kafka: poderoso para streaming e replay, mas desproporcional ao escopo didatico inicial. RabbitMQ tem filas e roteamento adequados ao caso de uso.

## Consequencias
Aumenta desacoplamento e resiliencia, mas adiciona infraestrutura, operacao e consistencia eventual nas notificacoes.
