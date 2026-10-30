# Identidade e Acesso

**Status:** arquitetura proposta; nao e implementacao do N9.

Registro, autenticacao, perfil; entidade Usuario e regras de bloqueio e unicidade.

- Entidade/projecao principal: Usuario
- Agregado: Usuario
- Eventos (proposta): UsuarioRegistrado (opcional)

## Contratos sugeridos

As portas de entrada expressam casos de uso; as portas de saida permitem trocar Sequelize, sistema de arquivos e broker sem alterar regras do dominio.
