# Notificacoes (proposto)

**Status:** arquitetura proposta; nao e implementacao do N9.

Consome TreinoPublicado e envia mensagens sem bloquear a publicacao; nao existe no N9.

- Entidade/projecao principal: Notificacao
- Agregado: Notificacao
- Eventos (proposta): Consome TreinoPublicado

## Contratos sugeridos

As portas de entrada expressam casos de uso; as portas de saida permitem trocar Sequelize, sistema de arquivos e broker sem alterar regras do dominio.
