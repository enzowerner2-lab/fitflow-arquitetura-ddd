# Evidencias SOLID - FitFlow N9

Metodo: inspecao estatica, sem testes em execucao. Os recortes preservam linhas da versao fornecida.

## 1. Atualização de perfil acoplada ao sistema de arquivos

`backend/modules/user/userService.js` (linhas 38–52)

```javascript
 38 | async function updateUserProfile(userId,payload,newFile){
 39 |   const user=await User.findByPk(userId);
 40 |   if(!user){const error=new Error('Usuário não encontrado.');error.status=404;throw error;}
 41 |   if(payload.fullName!==undefined) user.fullName=payload.fullName;
 42 |   if(payload.bio!==undefined) user.bio=payload.bio;
 43 |   if(newFile){
 44 |     const oldPicture=user.profilePicture;
 45 |     user.profilePicture=newFile.filename;
 46 |     if(oldPicture && oldPicture!=='default-profile.png'){
 47 |       const oldPath=path.join(__dirname,'..','..','public','uploads','profiles',oldPicture);
 48 |       fs.promises.unlink(oldPath).catch(()=>{});
 49 |     }
 50 |   }
 51 |   await user.save();
 52 |   return getUserProfile(userId);
```

**Princípios:** S — Responsabilidade Única; D — Inversão de Dependência
**Evidência:** O serviço modifica dados de perfil, manipula nome de arquivo e exclui imagens com fs/path. A regra de perfil fica ligada à implementação de armazenamento local, elevando a quantidade de motivos para mudança.
**Proposta:** Separar o caso de uso AtualizarPerfil de um FileStoragePort; mover exclusão de arquivo para adaptador de armazenamento e coordenar a ação após persistência confirmada.

## 2. Publicação de treino dependente de Sequelize e modelos ORM

`backend/modules/workout/workoutService.js` (linhas 13–24)

```javascript
 13 | async function createWorkout(userId, { title, description }, image) {
 14 |   const transaction = await sequelize.transaction();
 15 |   try {
 16 |     const workout = await Workout.create({ title, description, image: image.filename, userId }, { transaction });
 17 |     await User.increment('workoutsCount', { by: 1, where: { id: userId }, transaction });
 18 |     await transaction.commit();
 19 |     return workout;
 20 |   } catch (err) {
 21 |     await transaction.rollback();
 22 |     throw err;
 23 |   }
 24 | }
```

**Princípios:** D — Inversão de Dependência
**Evidência:** O caso de uso chama sequelize.transaction(), Workout.create() e User.increment() diretamente. O processo de negócio depende de contratos concretos da persistência.
**Proposta:** Criar PublicarTreino com WorkoutRepository e UnitOfWork; implementar essas portas em adaptadores Sequelize. Manter a contagem consistente na transação, preservando a regra de negócio.

## 3. Consulta de detalhes que também grava visualizações

`backend/modules/workout/workoutService.js` (linhas 26–42)

```javascript
 26 | async function getDetails(id, viewerId = null) {
 27 |   const workout = await Workout.findByPk(id, {
 28 |     include: [{ model: User, as: 'author', attributes: authorAttributes }]
 29 |   });
 30 |   if (!workout) {
 31 |     const error = new Error('Treino não encontrado.');
 32 |     error.status = 404;
 33 |     throw error;
 34 |   }
 35 | 
 36 |   await workout.increment('viewsCount');
 37 |   await workout.reload({ include: [{ model: User, as: 'author', attributes: authorAttributes }] });
 38 | 
 39 |   return {
 40 |     ...workout.toJSON(),
 41 |     isOwner: Boolean(viewerId && Number(viewerId) === Number(workout.userId))
 42 |   };
```

**Princípios:** S — Responsabilidade Única (sinal de mistura de leitura e comando)
**Evidência:** getDetails() busca o treino e também incrementa viewsCount antes de retorná-lo. A leitura passa a carregar uma política de contabilização, dificultando evolução e testes isolados.
**Proposta:** Extrair uma operação RegistrarVisualizacao e uma consulta BuscarDetalhesTreino. A aplicação pode orquestrá-las, de forma explícita, sem misturar consultas e atualização.

## 4. Edição de treino com remoção física de imagem

`backend/modules/workout/workoutService.js` (linhas 76–86)

```javascript
 76 | async function updateWorkout(id, userId, { title, description }, newImage) {
 77 |   const workout = await Workout.findByPk(id);
 78 |   if (!workout) throw notFound();
 79 |   if (Number(workout.userId) !== Number(userId)) throw forbidden();
 80 |   const oldImage = workout.image;
 81 |   workout.title = title.trim();
 82 |   workout.description = description.trim();
 83 |   if (newImage) workout.image = newImage.filename;
 84 |   await workout.save();
 85 |   if (newImage && oldImage !== workout.image) fs.promises.unlink(storedFile(oldImage)).catch(() => {});
 86 |   return workout;
```

**Princípios:** S — Responsabilidade Única; D — Inversão de Dependência
**Evidência:** A função faz validação de autoria, edição, persistência e remoção da imagem antiga via fs. Regras de edição e detalhes do disco coexistem no mesmo serviço.
**Proposta:** Introduzir UpdateWorkout use case, WorkoutRepository e ImageStorage; tratar troca/limpeza de arquivos por serviço dedicado com política de falha explícita.

## 5. Exclusão de treino e decremento fora de transação conjunta

`backend/modules/workout/workoutService.js` (linhas 89–97)

```javascript
 89 | async function deleteWorkout(id, userId) {
 90 |   const workout = await Workout.findByPk(id);
 91 |   if (!workout) throw notFound();
 92 |   if (Number(workout.userId) !== Number(userId)) throw forbidden();
 93 |   const image = workout.image;
 94 |   await workout.destroy();
 95 |   await User.decrement('workoutsCount', { by: 1, where: { id: userId } });
 96 |   await fs.promises.unlink(storedFile(image)).catch(() => {});
 97 |   return { id: Number(id) };
```

**Princípios:** S — Responsabilidade Única; D — Inversão de Dependência
**Evidência:** A mesma função elimina registro, altera contador de outro agregado e remove arquivo no disco. Há dependência direta de Sequelize e fs, e não se observa transação englobando as duas alterações de banco.
**Proposta:** Extrair ExcluirTreino e UoW, persistindo remoção e atualização do contador numa transação; solicitar limpeza do arquivo por uma porta de storage após commit, com retry/auditoria.

## 6. Interceptor HTTP controla navegação e sessão

`frontend/src/services/api.js` (linhas 37–47)

```javascript
 37 |       if (error.response.status === 401) {
 38 |         const { useAuthStore } = await import('../stores/auth')
 39 |         const authStore = useAuthStore()
 40 |         authStore.clearSession()
 41 | 
 42 |         if (window.location.pathname !== '/login') {
 43 |           const redirect = encodeURIComponent(
 44 |             window.location.pathname + window.location.search
 45 |           )
 46 |           window.location.href = `/login?redirect=${redirect}`
 47 |         }
```

**Princípios:** S — Responsabilidade Única; D — Inversão de Dependência
**Evidência:** O adaptador HTTP recebe erro 401, altera store de autenticação e executa redirecionamento via window.location. A camada de transporte passa a decidir comportamento da interface.
**Proposta:** Delegar o evento de sessão expirada a um serviço/handler de autenticação; a camada de apresentação ou router decide o redirecionamento. O interceptor apenas normaliza erros e notifica a política.

## 7. Formulário de publicação mistura validação, upload e estado da UI

`frontend/src/views/UploadView.vue` (linhas 27–50)

```javascript
 27 | function validateForm() {
 28 |   const errors = {}
 29 |   if (!title.value.trim()) errors.title = 'Informe o título do treino.'
 30 |   if (!description.value.trim()) errors.description = 'Informe a descrição do treino.'
 31 |   if (!image.value) errors.image = 'Escolha uma imagem para o treino.'
 32 |   fieldErrors.value = errors
 33 |   return Object.keys(errors).length === 0
 34 | }
 35 | 
 36 | async function submit() {
 37 |   successMessage.value = ''
 38 |   errorMessage.value = ''
 39 |   progress.value = 0
 40 |   if (!validateForm()) return
 41 | 
 42 |   const formData = new FormData()
 43 |   formData.append('title', title.value.trim())
 44 |   formData.append('description', description.value.trim())
 45 |   formData.append('image', image.value)
 46 | 
 47 |   isSubmitting.value = true
 48 |   try {
 49 |     const response = await createWorkout(formData, (event) => {
 50 |       if (event.total) progress.value = Math.round((event.loaded * 100) / event.total)
```

**Princípios:** S — Responsabilidade Única
**Evidência:** A view inclui validação de domínio, preparação do FormData, operação de rede e atualização de progresso; concentra política de formulário e infraestrutura de upload.
**Proposta:** Extrair um composable useWorkoutForm para estado e validação; centralizar serialização/upload em workoutService; deixar à view a renderização e interação.

## 8. Regras de tipo de arquivo repetidas em adaptadores

`backend/middlewares/workoutMulter.js` (linhas 16–20)

```javascript
 16 | const fileFilter = (_req, file, cb) => {
 17 |   const allowed = ['image/jpeg', 'image/png', 'image/webp'];
 18 |   if (!allowed.includes(file.mimetype)) {
 19 |     return cb(new Error('A imagem do treino deve ser JPG, PNG ou WEBP.'));
 20 |   }
```

**Princípios:** O — Aberto-Fechado (ponto de extensão limitado)
**Evidência:** A whitelist de MIME types está codificada no middleware de treino e aparece também em profileMulter.js, linhas 15–19. Acrescentar um tipo permitido demanda editar múltiplas implementações em vez de configurar política compartilhada.
**Proposta:** Criar uma factory createImageUpload({destination, maxSize, allowedMimeTypes, namingPolicy}) e injetar políticas por caso de uso; extensões seriam feitas por configuração/estratégia.
