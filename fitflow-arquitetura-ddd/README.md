# FitFlow - Proposta de Arquitetura DDD / Hexagonal e Mensageria C4

**Estudante:** Enzo Gonçalves Werner  
**Disciplina:** Projeto e Arquitetura de Software  
**Professor:** Luiz C. Camargo, PhD

Material de **Projeto e Arquitetura de Software**, baseado na versao N9 do projeto FitFlow (backend/ frontend analisados). **Esta pasta contem organizacao arquitetural e documentacao; nao constitui migracao funcional do codigo de Programacao Web.**

## Onde comecar
- [`docs/visao-ddd.md`](docs/visao-ddd.md): domain model, bounded contexts e hexagonal.
- [`docs/C4.md`](docs/C4.md): C1, C2 e C3 em imagens.
- [`docs/ADR-001-mensageria.md`](docs/ADR-001-mensageria.md): motivos e escolhas do RabbitMQ.
- [`docs/metodologia.md`](docs/metodologia.md): protocolo de inspecao SOLID.
- [`docs/evento-treino-publicado.json`](docs/evento-treino-publicado.json): exemplo de contrato de evento.
- [`src/contexts`](src/contexts): proposta de diretorios por contextos delimitados e camadas hexagonais.

## Estrutura de arquitetura
```text
src/contexts/
  identity/
  workouts/
  feed/
  notifications/          # novo, proposto
    domain/{entities,value-objects,events,ports}
    application/{use-cases,dto}
    adapters/in/{http,events}
    adapters/out/{persistence,messaging,storage}
```
Todos os contextos seguem a mesma organizacao `domain`, `application` e `adapters`. Pastas vazias sao intencionais e possuem `.gitkeep`; o professor **nao exige codigo-fonte refatorado na questao 1**.

## Publicar no GitHub
1. Criar repositorio publico **fitflow-arquitetura-ddd** em sua conta GitHub.
2. Descompactar este pacote, abrir o terminal dentro da pasta e executar:
```bash
git init
git add .
git commit -m "Documenta arquitetura DDD, hexagonal e mensageria C4"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/fitflow-arquitetura-ddd.git
git push -u origin main
```
3. Trocar `SEU-USUARIO` e inserir o link real do repositorio no PDF antes da entrega.

**Seguranca:** nao publicar `.env`, senhas, tokens ou outras credenciais. Nenhum `.env` do projeto original esta incluido neste pacote.
