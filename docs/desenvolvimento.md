# DOCUMENTO DE ESPECIFICAÇÃO ARQUITETURAL E DE DESENVOLVIMENTO
Projeto: Gia Horóscopo (Horóscopo Inteligente) 

1. ESCOPO DA STACK E FERRAMENTAS

* Front-end: React 19 (latest), Vite (latest), TypeScript, Tailwind CSS, shadcn/ui.
* Gerenciamento de Formulários & Validação: React Hook Form + Zod.
* Roteamento & DOM: React Router DOM (latest).
* Gerenciador de Pacotes: npm.
* Back-end de Orquestração: Node.js (Servindo como BFF/Proxy se necessário, ou consumindo o n8n diretamente).

1. ARQUITETURA DO PROJETO E DESIGN PATTERNS
O projeto adota uma arquitetura limpa adaptada para o ecossistema React, garantindo desacoplamento entre a interface de usuário (UI), as regras de negócio e os serviços externos (n8n/API).

Estrutura de Pastas Sugerida:
src/
├── assets/         # Imagens, ícones, fontes
├── components/     # UI Components Globais (Design System / shadcn)
│   └── ui/         # Componentes primitivos do shadcn/ui
├── features/       # Módulos baseados em domínio (Screaming Architecture)
│   └── horoscope/
│       ├── components/ # Componentes específicos da feature (ex: HoroscopeResult)
│       ├── hooks/      # Custom hooks (Regras de negócio e estado: useHoroscope)
│       ├── services/   # Gateways de API (Tratamento de dados do n8n)
│       ├── validators/ # Schemas do Zod para esta feature
│       └── views/      # Páginas/Telas da feature (ex: HoroscopePage)
├── routes/         # Configuração de rotas (React Router DOM)
├── shared/         # Utilitários, contextos e tipos genéricos
│   ├── design-patterns/ # Implementações de patterns globais
│   └── utils/
└── main.tsx 

Práticas e Princípios Aplicados: 

* S.O.L.I.D.: 

  * Single Responsibility Principle (SRP): Componentes renderizam apenas UI. A lógica de busca e transformação fica em Custom Hooks e Services.
  * Dependency Inversion Principle (DIP): O front-end depende de abstrações (interfaces/tipos de dados) e não do formato bruto que a API do n8n entrega.
* DRY & Clean Code: Funções utilitárias puras para conversão de datas e cálculo de signos baseados em Ranges imutáveis.
* Design Patterns: 

  * Repository Pattern: Abstrai as chamadas HTTP para o n8n em uma camada de serviço isolada.
  * Adapter Pattern: Converte a estrutura de dados (JSON) vinda do n8n para o formato exato que a UI do React espera.
  * Factory Pattern: Utilizado para instanciar payloads de requisição de maneira padronizada.

1. ESTILIZAÇÃO E DESIGN SYSTEM (TAILWIND + SHADCN/UI)
O design deve refletir misticismo e modernidade através de uma paleta de cores profunda (Dark Mode predominante).

Configuração de Cores (Tailwind CSS): 

* Background: Escuro Profundo (ex: slate-950 ou zinc-950).
* Primary/Accents: Tons de roxo cósmico, dourado estelar e azul neon (ex: violet-600, amber-400, indigo-400).

Componentes shadcn/ui a serem instalados de imediato: 

* Form, Input (para coleta de nome, e-mail).
* Select, Popover + Calendar (para seleção otimizada da data de nascimento).
* Card, Skeleton (para exibição do resultado e estado de carregamento/loading do n8n).

1. FLUXO DE DADOS, VALIDAÇÃO E FORMULÁRIOS
A captura de dados utilizará o React Hook Form integrado ao Zod para garantir tipagem estática e validação em tempo de execução de forma unificada.

Schema de Validação (Zod):
O schema deve validar se a data de nascimento é válida, se o horário está no formato correto (HH:MM) e se a cidade foi preenchida. Caso o usuário opte por buscar diretamente pelo signo, o Zod aplicará uma validação via Enum. 

Fluxo do Formulário: 

1. O usuário preenche os dados de nascimento no formulário validado pelo Zod.
2. Ao submeter, o React Hook Form dispara a função de submit que invoca o Custom Hook de busca.
3. O hook ativa o estado de loading (disparando o Skeleton na tela) e envia o payload limpo para o serviço HTTP.
4. METODOLOGIA DE DESENVOLVIMENTO (TDD & BDD)
O ciclo de desenvolvimento será guiado por testes e comportamentos esperados pelo usuário antes mesmo da escrita do código final.

Abordagem BDD (Behavior-Driven Development):
Os critérios de aceitação serão definidos usando a sintaxe Gherkin para alinhar o comportamento do negócio com o desenvolvimento: 

* Cenário: Usuário consulta horóscopo com sucesso por data de nascimento. 

  * Dado que o usuário está na página inicial do portal.
  * Quando ele insere a data "15/04/1995" no campo de nascimento.
  * E clica no botão "Gerar Mapa e Horóscopo".
  * Então o sistema deve fazer uma requisição para o fluxo do n8n.
  * E deve exibir o card de resultado contendo a previsão astrológica rica baseada no acervo do Pinecone.

Abordagem TDD (Test-Driven Development):
Utilizando Vitest e React Testing Library, o desenvolvimento seguirá o ciclo Red-Green-Refactor: 

1. Red: Escrever um teste que simula o preenchimento do formulário com uma data inválida e espera encontrar uma mensagem de erro na tela. O teste falhará.
2. Green: Escrever o schema do Zod e ligá-lo ao componente do formulário para fazer o erro aparecer. O teste passará.
3. Refactor: Limpar a estrutura do componente, mover o schema para um arquivo de validação isolado e garantir que o código esteja legível sem quebrar o teste.
4. ESTRATÉGIA DE METODOLOGIA ÁGIL
Para gerenciar o desenvolvimento do produto de forma incremental e adaptativa:

* Sprints Semanais: Dado que o escopo envolve integração com IA e banco vetorial, sprints curtas de 1 semana ajudam a mitigar riscos de chamadas de API e latência.
* MVP (Minimum Viable Product): 

  * Sprint 1: Setup do repositório (Vite, TS, Tailwind, shadcn) + Criação do formulário com Zod e validações locais + Testes unitários do form.
  * Sprint 2: Criação do serviço de API + Integração com o Webhook de testes do n8n + Exibição do estado de loading e erro.
  * Sprint 3: Polimento visual com animações + Criação dos cards de exibição do resultado detalhado (RAG) + Testes de integração de ponta a ponta.
* Critério de Pronto (Definition of Done - DoD): Uma funcionalidade só é considerada concluída se possuir cobertura de testes unitários, passar na validação estática do TypeScript e rodar sem erros de renderização em ambiente de staging.