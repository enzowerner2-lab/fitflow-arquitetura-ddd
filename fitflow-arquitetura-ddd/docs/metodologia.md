# Protocolo de inspecao SOLID (replicavel)

1. Obter os ZIPs N9 `EnzoWesleyFitFlow-backend(2).zip` e `EnzoWesleyFitFlow-frontend(2).zip`.
2. Extrair ambos conservando pastas `backend/` e `frontend/` e ignorar `node_modules/`, `dist/`, arquivos `.env` e imagens.
3. Localizar as rotas HTTP nos arquivos `backend/modules/*/*Routes.js`; rastrear controllers, services, models, middlewares e o fluxo Vue por imports.
4. Numerar linhas dos arquivos JavaScript/Vue (por exemplo, `nl -ba backend/modules/workout/workoutService.js`).
5. Inspecionar chamadas diretas a banco, arquivo, token, navegador, validação, apresentação e tratamento de eventos. Aplicar a interpretação do enunciado: módulos ~ classes; funções ~ métodos.
6. Em cada caso, registrar caminho, linhas, recorte literal, principio SOLID envolvido, motivo e sugestão de refatoracao. Distinguir evidencia de hipotese e propostas de funcionalidades existentes.
7. Validar o recorte contra o ZIP e revisar os achados. Nao inferir testes de execucao: a tecnica e leitura estatica, nao validacao funcional.

A ausencia de heranca explicita e de interfaces polimorficas adequadas impede afirmar violacoes de LSP e ISP a partir destes recortes; nao se forcam exemplos artificiais.
