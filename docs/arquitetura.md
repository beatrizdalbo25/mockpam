# Documento de Arquitetura do Aplicativo Mobile

## A. Features escolhidas

Para o aplicativo de gestão de treinamentos, a arquitetura será organizada por features, seguindo o princípio de que o app deve refletir experiências de usuário e não apenas recursos da API. A proposta inicial considera as seguintes features:

### 1. Auth
- Objetivo: permitir que o usuário faça login e mantenha sessão ativa.
- Por que é relevante: a autenticação é o ponto de entrada do sistema e depende de dados como e-mail, senha e associação com funcionário.
- Dados envolvidos: `usuarios`, `funcionarios`, `perfis`, `permissoes`, `usuarioPerfis`.
- Telas esperadas: login, recuperação de senha, splash e tela inicial após autenticação.

### 2. Dashboard
- Objetivo: apresentar uma visão geral do usuário e do status dos treinamentos.
- Por que é relevante: é a área de entrada mais útil após login, mostrando pendências, certificados e progresso.
- Dados envolvidos: `treinamentos`, `treinamentoParticipantes`, `certificados`, `evidencias`, `auditorias`.
- Telas esperadas: resumo do usuário, treinamentos em andamento, próximos vencimentos e certificados recentes.

### 3. Treinamentos
- Objetivo: listar, consultar e acompanhar treinamentos.
- Por que é relevante: é a funcionalidade principal do sistema, com grande volume de dados e diferentes relações (instrutores, responsáveis, participantes e evidências).
- Dados envolvidos: `treinamentos`, `treinamentoInstrutores`, `treinamentoParticipantes`, `treinamentoResponsaveis`, `instrutores`, `funcionarios`.
- Telas esperadas: lista de treinamentos, detalhes do treinamento, participantes, evidências e status.

### 4. Certificados
- Objetivo: disponibilizar certificados emitidos e o estado de validade dos treinamentos concluídos.
- Por que é relevante: representa um resultado tangível para o usuário e é uma funcionalidade de alto valor para o negócio.
- Dados envolvidos: `certificados`, `treinamentoParticipantes`, `treinamentos`, `auditorias`.
- Telas esperadas: lista de certificados, detalhe do certificado e informações de validade.

### 5. Perfil
- Objetivo: exibir dados do usuário logado, informações pessoais e contexto do colaborador.
- Por que é relevante: o perfil centraliza a identificação do usuário e a personalização da experiência.
- Dados envolvidos: `usuarios`, `funcionarios`, `perfis`.
- Telas esperadas: dados pessoais, cargo, setor e configurações do perfil.

### Justificativa para não criar feature para cada recurso da API

Os endpoints da API representam entidades de negócio, mas nem todos se traduzem em features independentes no app. Por exemplo:
- `funcionarios`, `instrutores`, `perfis` e `permissoes` podem ser consumidos como dados auxiliares de outras features;
- `perfilPermissoes` e `usuarioPerfis` não são telas em si, mas regras de acesso e relacionamento;
- `assinaturas` e `auditorias` são dados complementares e devem aparecer em contexto específico, não como feature principal;
- `evidencias` devem ser exibidas dentro da feature de treinamentos ou dashboard, e não como aplicação independente.

A separação por feature reduz acoplamento e facilita crescimento futuro sem misturar regras de negócio e visualização.

---

## B. Estrutura de pastas

A estrutura inicial proposta para o projeto segue uma organização em camadas e features:

```text
src/
├── app/
│   ├── navigation/
│   ├── routes/
│   ├── screens/
│   └── app.tsx
├── core/
│   ├── api/
│   ├── config/
│   ├── hooks/
│   └── utils/
├── shared/
│   ├── components/
│   ├── theme/
│   ├── types/
│   └── constants/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── screens/
│   ├── dashboard/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── screens/
│   ├── treinamentos/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── screens/
│   ├── certificados/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── screens/
│   └── perfil/
│       ├── components/
│       ├── hooks/
│       ├── services/
│       └── screens/
└── index.ts
```

### Responsabilidade de cada camada

#### app/
- Centraliza a navegação e a composição visual da aplicação.
- Define o fluxo principal do app e as rotas acessíveis.
- Mantém as telas e as rotas na camada de apresentação.

#### core/
- Contém infraestruturas transversais.
- Guarda acesso à API, configurações globais, hooks reutilizáveis e utilitários.
- Não deve depender diretamente de features específicas.

#### shared/
- Guarda elementos reutilizados pelos módulos.
- Fornece temas, componentes comuns, tipos globais e constantes.
- Evita duplicação de código entre features.

#### features/
- Organiza o negócio por funcionalidade.
- Cada feature encapsula telas, serviços, hooks e componentes relacionados.
- Facilita manutenção, testes e evolução incremental do app.

---

## C. Navegação

A navegação inicial do aplicativo será estruturada em um fluxo simples e objetivo:

```text
Splash / Login
    ↓
Autenticação
    ↓
Dashboard
    ├── Treinamentos
    │   ├── Lista de treinamentos
    │   └── Detalhes do treinamento
    ├── Certificados
    │   ├── Listagem
    │   └── Detalhe do certificado
    └── Perfil
        └── Dados do usuário
```

### Fluxo sugerido
1. O usuário abre o app.
2. Se não estiver autenticado, é direcionado para a tela de login.
3. Após login, entra no dashboard.
4. A partir do dashboard, o usuário navega para treinamentos, certificados ou perfil.
5. Cada feature acessa os dados por meio de hooks e serviços específicos.

### Vantagens desse modelo
- reduz a complexidade inicial;
- facilita a inclusão de novas telas;
- permite que cada feature tenha uma responsabilidade clara;
- mantém a navegação coerente com a experiência do cliente.

---

## D. Comunicação com a API

A comunicação com a API seguirá o padrão de separação entre apresentação e dados:

```text
Screen
↓
Hook
↓
Service
↓
API
```

### 1. Screen
- A tela somente renderiza dados e dispara ações do usuário.
- Não conhece detalhes da API.
- Exemplo: ao abrir a tela de treinamentos, a screen chama um hook responsável por buscar os dados.

### 2. Hook
- Centraliza a lógica de carregamento, estado e sincronização da tela.
- Encapsula carregamento, erro, sucesso e filtros.
- Exemplo: `useTrainings()` pode encapsular `isLoading`, `error` e `trainings`.

### 3. Service
- É responsável por acessar a API e converter respostas para o formato que a aplicação espera.
- Mantém a infraestrutura de networking separada da interface.
- Exemplo: `trainingService.getAll()` chama o endpoint `/api/treinamentos`.

### 4. API
- Responde em formato JSON e fornece os dados da regra de negócio.
- O app consome apenas o necessário para a tela atual.

### Exemplo de fluxo real
- Tela: `TreinamentosScreen`
- Hook: `useTrainings()`
- Service: `trainingService.list()`
- API: `GET /api/treinamentos`

Esse padrão facilita manutenção, testes e troca da implementação do serviço sem impactar a interface.

---

## E. Decisões arquiteturais

A seguir estão as principais decisões de arquitetura adotadas para o projeto.

### 1. Organização por features
- Decisão: o projeto será estruturado por funcionalidade e não por tipo de arquivo.
- Justificativa: reduz o acoplamento entre funcionalidades e facilita a escala do app quando novas áreas forem adicionadas.

### 2. Separação de camadas
- Decisão: apresentação, regras de negócio e comunicação com a API ficarão separadas.
- Justificativa: melhora a manutenção, permite testes mais focados e evita que a tela dependa diretamente de detalhes técnicos.

### 3. Uso de hooks para acesso aos dados
- Decisão: a lógica de estado e carregamento ficará em hooks.
- Justificativa: os hooks encapsulam comportamento reutilizável e deixam a interface mais limpa e legível.

### 4. Serviços específicos por feature
- Decisão: cada feature possuirá seus próprios serviços.
- Justificativa: cada funcionalidade acessa endpoints distintos e tem contratos de dados diferentes. Isso reduz mistura de responsabilidades.

### 5. Shared para elementos reutilizáveis
- Decisão: componentes e temas comuns serão compartilhados em `shared`.
- Justificativa: evita duplicação de UI e mantém a identidade visual consistente em todas as telas.

### 6. Navegação centralizada em `app`
- Decisão: todas as rotas e fluxos principais ficarão na camada de aplicação.
- Justificativa: facilita a organização da experiência do usuário e permite evoluir a navegação sem atrapalhar a lógica das features.

### 7. Consumo orientado à experiência do usuário
- Decisão: o app não será uma cópia fiel da estrutura da API, mas uma visão funcional do sistema.
- Justificativa: a API expõe recursos e dados; o app deve juntar esses dados em experiências úteis para o usuário final.

---

## Conclusão

A arquitetura proposta busca equilibrar clareza, manutenção e escalabilidade. Ao organizar o app por features, manter a camada de apresentação separada da comunicação com a API e centralizar a navegação, o projeto estará preparado para receber novas funcionalidades sem perder organização.

Essa abordagem é adequada para um ambiente mobile com crescimento previsto, pois reduz o risco de acoplamento e facilita a evolução do sistema ao longo do tempo.
