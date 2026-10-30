# Treinos

**Status:** arquitetura proposta; nao e implementacao do N9.

Publicacao, edicao, exclusao e contagem; agregado Treino, ID do autor como referencia externa.

- Entidade/projecao principal: Treino
- Agregado: Treino
- Eventos (proposta): TreinoPublicado, TreinoAtualizado, TreinoRemovido

## Contratos sugeridos

As portas de entrada expressam casos de uso; as portas de saida permitem trocar Sequelize, sistema de arquivos e broker sem alterar regras do dominio.
