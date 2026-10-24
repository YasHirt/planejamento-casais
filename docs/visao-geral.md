# Visão Geral do Projeto

## 1. Apresentação

O projeto consiste no desenvolvimento de uma aplicação web voltada ao planejamento e à organização de atividades, experiências e objetivos compartilhados por casais.

Inspirada na organização visual de ferramentas Kanban, a plataforma permitirá estruturar planejamentos em quadros, colunas e atividades, oferecendo uma experiência personalizada por meio da integração com inteligência artificial generativa.

A proposta é combinar organização, criatividade e personalização em um ambiente que ajude casais a transformar ideias em experiências concretas.

## 2. Problema e motivação

Ferramentas tradicionais de gerenciamento de tarefas oferecem recursos para organizar atividades de diferentes naturezas. Entretanto, por serem soluções generalistas, normalmente exigem que o próprio usuário defina a estrutura do planejamento e identifique quais atividades deseja realizar.

No contexto de casais, o planejamento pode envolver preferências individuais, interesses compartilhados, orçamento, disponibilidade e objetivos específicos.

A ausência de uma experiência direcionada a esse público motivou a concepção de uma plataforma que não apenas organize atividades, mas também auxilie na descoberta e na elaboração de experiências personalizadas.

## 3. Proposta de solução

A aplicação oferecerá um processo guiado de criação de planejamentos.

Inicialmente, o usuário informará o objetivo do casal e responderá a um questionário contextualizado, cujas perguntas poderão variar conforme a categoria selecionada, como viagens ou encontros.

As respostas serão utilizadas como contexto para um modelo de inteligência artificial generativa, responsável por sugerir atividades e estruturas de organização adequadas aos interesses apresentados.

Antes da persistência, as sugestões poderão ser revisadas, modificadas ou descartadas pelo usuário.

Após a criação, o planejamento poderá ser gerenciado manualmente por meio de colunas e atividades, com recursos de organização visual, personalização de capas e armazenamento de arquivos.

A inteligência artificial atuará como ferramenta de apoio à personalização, enquanto a aplicação será responsável pela validação, pelas regras de negócio e pela persistência dos dados.

## 4. Público-alvo

A plataforma é direcionada a casais que desejam organizar atividades e experiências compartilhadas de maneira prática, visual e personalizada.

Entre os possíveis contextos de utilização estão:

- Planejamento de viagens e passeios.
- Organização de encontros e atividades a dois.
- Preparação de experiências especiais.
- Registro de fotografias, ingressos e recordações.
- Organização de objetivos compartilhados.

O desenvolvimento inicial será concentrado em viagens e encontros, com possibilidade de expansão para outras categorias.

## 5. Diferenciais do projeto

### 5.1. Personalização contextualizada por IA

A geração de sugestões considerará informações específicas do casal, incluindo interesses individuais, preferências compartilhadas, objetivos e restrições informadas no questionário.

A proposta não é apenas gerar tarefas genéricas, mas oferecer sugestões relevantes para o contexto apresentado.

### 5.2. Experiência de planejamento guiada

A criação de um quadro será acompanhada de um questionário contextualizado, reduzindo a necessidade de o usuário estruturar manualmente todo o planejamento.

### 5.3. Identidade visual temática

A aplicação adotará uma identidade visual baseada em imagens de gatos adaptadas aos temas das atividades.

Uma biblioteca interna de imagens categorizadas permitirá a seleção automática de capas relacionadas às sugestões geradas pela IA.

O usuário também poderá substituir as capas escolhidas automaticamente.

### 5.4. Organização de arquivos e recordações

As atividades poderão conter coleções personalizadas de anexos, permitindo organizar fotografias, ingressos, passagens e outros documentos.

Dessa forma, a plataforma poderá servir tanto ao planejamento de experiências futuras quanto à preservação de recordações.

### 5.5. Controle do usuário

As sugestões produzidas pela inteligência artificial não serão obrigatoriamente incorporadas ao planejamento.

O usuário poderá revisar o conteúdo sugerido e manter o controle sobre a organização e a personalização das atividades.

## 6. Tecnologias previstas

| Camada | Tecnologia | Finalidade |
|---|---|---|
| Frontend | Angular | Construção da interface web |
| Backend | Java e Spring Boot | API REST e regras de negócio |
| Persistência | PostgreSQL | Armazenamento dos dados |
| Migrations | Flyway | Versionamento do esquema do banco |
| Inteligência artificial | Gemini API | Geração de sugestões contextualizadas |
| Persistência de arquivos | Armazenamento local inicialmente | Armazenamento dos anexos e imagens |
| Versionamento | Git e GitHub | Controle de versões e documentação |

As escolhas tecnológicas poderão ser revisadas conforme as necessidades identificadas durante o desenvolvimento.

## 7. Diretrizes de desenvolvimento

O projeto será desenvolvido incrementalmente, com foco na qualidade do código, na compreensão das decisões técnicas e na aplicação de boas práticas de engenharia de software.

Serão priorizados:

- Separação de responsabilidades e baixo acoplamento.
- Modelagem de dados orientada às necessidades do domínio.
- Validação das informações produzidas pela inteligência artificial.
- Testes automatizados para regras de negócio e funcionalidades relevantes.
- Documentação das decisões arquiteturais por meio de ADRs (*Architecture Decision Records*).
- Implementação incremental, evitando complexidade desnecessária.

## 8. Escopo inicial e evolução

A primeira versão será desenvolvida como um MVP (*Minimum Viable Product*), priorizando um único quadro de planejamento, utilização administrativa sem autenticação e sugestões personalizadas por IA.

O MVP incluirá gerenciamento de colunas e atividades, questionário contextualizado, biblioteca temática de imagens, personalização de capas e organização de anexos.

Funcionalidades como autenticação, colaboração em tempo real, múltiplos quadros simultâneos, notificações e integração com serviços externos de localização poderão ser consideradas em versões futuras.

O escopo detalhado será mantido no documento [MVP](mvp.md).

## 9. Documentação complementar

A documentação técnica será organizada em arquivos específicos:

- [Escopo do MVP](mvp.md)
- [Modelagem de dados](modelagem-dados.md)
- [Registros de decisões arquiteturais](adr/)

Esses documentos serão atualizados conforme a evolução do projeto e poderão registrar alterações nas decisões inicialmente propostas.
