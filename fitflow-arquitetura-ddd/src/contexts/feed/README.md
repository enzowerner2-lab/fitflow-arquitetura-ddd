# Feed e Busca

**Status:** arquitetura proposta; nao e implementacao do N9.

Consultas paginadas e pesquisa; um modelo de leitura, inicialmente dependente dos dados dos treinos. Busca global atual e um placeholder; evolucao proposta.

- Entidade/projecao principal: ProjecaoTreino
- Agregado: Modelo de leitura
- Eventos (proposta): Consome eventos para atualizar projecoes (futuro)

## Contratos sugeridos

As portas de entrada expressam casos de uso; as portas de saida permitem trocar Sequelize, sistema de arquivos e broker sem alterar regras do dominio.
