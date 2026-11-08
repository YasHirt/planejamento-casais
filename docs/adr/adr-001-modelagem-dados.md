# ADR-0001: Modelagem e organização do banco de dados

**Status:** Proposto  
**Data:** 08/11/2026  
**Área:** Arquitetura de dados e persistência  
**Tecnologia:** PostgreSQL

## 1. Contexto

O projeto consiste em uma plataforma de planejamento para casais, utilizando inteligência artificial generativa para sugerir atividades personalizadas a partir dos objetivos e interesses dos usuários.

A aplicação permitirá organizar atividades em colunas, armazenar respostas de questionários e personalizar cards com imagens temáticas.

Também será possível criar coleções de anexos dentro das atividades, permitindo o armazenamento de fotografias, ingressos, passagens e outros documentos.

A modelagem deve atender às funcionalidades do MVP, priorizando:

- Separação de responsabilidades entre entidades.
- Baixo acoplamento.
- Integridade referencial.
- Reutilização dos recursos de armazenamento.
- Flexibilidade para evolução.
- Controle sobre a persistência e a exclusão de arquivos.

## 2. Decisão arquitetural

Será utilizado um banco de dados relacional PostgreSQL, com modelagem inicial composta por oito tabelas:

1. `planejamento`
2. `resposta_questionario`
3. `coluna`
4. `atividade`
5. `colecao_anexo`
6. `anexo`
7. `colecao_anexo_item`
8. `imagem_biblioteca`

As entidades serão organizadas de acordo com suas responsabilidades no domínio.

### 2.1. Planejamento e questionário

A entidade `Planejamento` representa o quadro principal, armazenando seu nome, categoria, objetivo e informações gerais.

A entidade `RespostaQuestionario` armazena as respostas utilizadas para contextualizar as sugestões da IA.

O relacionamento será de um para muitos (1:N), permitindo que um planejamento possua várias respostas.

No MVP, será permitido apenas um planejamento ativo por instalação.

As perguntas serão definidas inicialmente na aplicação, sem necessidade de uma tabela específica para cadastro de perguntas.

A categoria será representada por um enum Java, persistido como string.

### 2.2. Colunas e atividades

Um planejamento poderá possuir várias colunas, e cada coluna poderá conter várias atividades.

Os relacionamentos serão:

- `Planejamento 1:N Coluna`
- `Coluna 1:N Atividade`

Colunas e atividades possuirão um atributo `ordem`, responsável por determinar sua posição na interface.

A movimentação de uma atividade entre colunas será representada pela alteração de sua referência à coluna de destino e de sua posição.

### 2.3. Coleções de anexos

Uma atividade poderá possuir várias coleções de anexos.

Cada coleção terá um nome definido pelo usuário, como "Ingressos", "Passagens" ou "Fotos da viagem".

O relacionamento será:

`Atividade 1:N ColecaoAnexo`

A coleção representa um agrupamento lógico de arquivos, não sendo responsável pelo armazenamento físico dos documentos.

### 2.4. Entidade Anexo genérica

A entidade `Anexo` armazenará exclusivamente os metadados necessários para identificar e recuperar um arquivo.

Entre esses metadados estão:

- Nome original.
- Chave de armazenamento.
- Tipo MIME.
- Tamanho em bytes.
- Data de criação.
- Data de exclusão lógica.

O arquivo físico será armazenado separadamente, inicialmente em um diretório local.

A entidade não possuirá uma chave estrangeira direta para `ColecaoAnexo`, permitindo sua reutilização em outros contextos, como imagens de capa e futuras fotos de perfil.

### 2.5. Relacionamento N:N entre coleções e anexos

Será criada a tabela associativa `ColecaoAnexoItem`.

Essa tabela representará o relacionamento muitos-para-muitos (N:N) entre `ColecaoAnexo` e `Anexo`.

Um anexo poderá pertencer a várias coleções, enquanto cada coleção poderá conter vários anexos.

Será aplicada a restrição:

`UNIQUE(colecao_id, anexo_id)`

Isso impedirá que o mesmo anexo seja adicionado duas vezes à mesma coleção, sem impedir sua utilização em outras coleções.

O relacionamento será modelado como entidade associativa própria, permitindo armazenar atributos adicionais, como a ordem de exibição.

### 2.6. Identificação da capa da atividade

A capa será representada por uma chave estrangeira opcional `capa_id` na tabela `atividade`, referenciando `anexo.id`.

Essa abordagem foi escolhida em vez de uma flag `eh_capa` na tabela de anexos.

A decisão oferece as seguintes vantagens:

- Cada atividade aponta para, no máximo, uma capa.
- A troca da capa exige atualizar apenas uma referência.
- O anexo não precisa conhecer sua utilização como capa.
- O mesmo arquivo pode ser reutilizado em diferentes atividades.
- A origem da imagem não interfere no relacionamento.

A aplicação validará se o anexo utilizado como capa é uma imagem compatível.

Também será registrada a origem da seleção da capa, distinguindo escolhas automáticas de escolhas manuais, para evitar sobrescrever uma imagem selecionada pelo usuário.

### 2.7. Biblioteca de imagens temáticas

Será criada a entidade `ImagemBiblioteca`, responsável por catalogar imagens disponibilizadas pela própria aplicação.

Cada registro terá uma referência para um `Anexo` e armazenará informações de classificação, como título e tags.

O relacionamento será:

`Anexo 1:0..1 ImagemBiblioteca`

Um registro da biblioteca estará associado obrigatoriamente a um anexo, enquanto um anexo poderá ou não possuir uma entrada na biblioteca.

A restrição `UNIQUE(anexo_id)` impedirá que o mesmo anexo seja cadastrado mais de uma vez no catálogo.

Quando a IA sugerir um tema visual, o backend pesquisará imagens compatíveis na biblioteca.

Ao selecionar uma imagem, a atividade referenciará diretamente `Anexo.id` por meio de `capa_id`.

Nenhum novo registro de anexo ou cópia física será necessário.

### 2.8. Exclusão lógica de anexos

A entidade `Anexo` possuirá o atributo `excluido_em`, inicialmente nulo.

O preenchimento desse campo representará a exclusão lógica do arquivo.

Como um mesmo anexo poderá ser compartilhado por diferentes coleções, capas e registros da biblioteca, a remoção de uma associação não implicará automaticamente a exclusão do anexo.

Serão diferenciadas as operações:

- **Remoção do vínculo:** desvincula um anexo de uma coleção ou atividade.
- **Exclusão lógica:** marca o anexo como excluído quando ele não possuir referências que exijam sua preservação.
- **Exclusão física:** remove o conteúdo do armazenamento quando o arquivo estiver elegível à limpeza.

Será prevista uma rotina agendada para exclusão física de anexos logicamente excluídos após um período de retenção.

A implementação do job ficará para uma etapa posterior ao desenvolvimento das funcionalidades principais.

A rotina deverá ser tolerante a falhas, permitir novas tentativas e considerar que o banco de dados e o armazenamento de arquivos não compartilham necessariamente a mesma transação.

## 3. Alternativas consideradas

### 3.1. Armazenar `colecao_id` diretamente em Anexo

**Não adotada.**

Embora simplifique a modelagem, introduz uma referência específica de coleção na entidade genérica de arquivo e impede múltiplos vínculos por meio dessa única FK.

### 3.2. Utilizar `@ManyToMany` diretamente entre coleção e anexo

**Não adotada.**

Foi escolhida uma entidade associativa explícita para permitir atributos próprios do vínculo, como `ordem`, e facilitar futuras regras de negócio.

### 3.3. Identificar capas por uma flag `eh_capa`

**Não adotada.**

Essa abordagem exigiria controles adicionais para garantir que cada atividade possua somente uma capa e para coordenar mudanças entre diferentes registros.

A referência direta por `capa_id` representa melhor a regra de negócio escolhida.

### 3.4. Armazenar imagens da biblioteca em tabela independente de arquivos

**Não adotada.**

Criaria duas estruturas distintas para representar metadados de arquivos semelhantes.

A entidade `ImagemBiblioteca` será responsável apenas pela catalogação temática, reutilizando os registros de `Anexo`.

### 3.5. Exclusão física imediata

**Não adotada.**

A remoção física imediata dificultaria a recuperação de arquivos e exigiria maior coordenação entre operações do banco de dados e do sistema de armazenamento.

## 4. Consequências da decisão

### Positivas

- Separação clara de responsabilidades.
- Reutilização de arquivos em diferentes funcionalidades.
- Redução de duplicação de arquivos físicos.
- Suporte a múltiplas coleções de anexos.
- Biblioteca temática independente das atividades.
- Possibilidade de substituir o armazenamento local por armazenamento em nuvem.
- Maior controle sobre exclusões e integridade dos vínculos.
- Modelo extensível sem exigir mudanças constantes na entidade `Anexo`.

### Negativas e trade-offs

- Maior quantidade de tabelas e relacionamentos.
- Necessidade de JOINs adicionais para recuperar anexos das coleções.
- Complexidade adicional na verificação de referências antes da exclusão.
- Necessidade de coordenar a persistência dos metadados com o armazenamento físico.
- Algumas regras de integridade precisarão de validação na aplicação ou de restrições adicionais no banco.

## 5. Decisões pendentes

Antes da implementação definitiva das migrations, deverão ser revisados:

- Estratégia de exclusão das entidades relacionadas a um planejamento.
- Regras de unicidade e ordenação de colunas e atividades.
- Comportamento ao excluir uma imagem utilizada como capa.
- Controle de concorrência na associação e exclusão de anexos.
- Política de retenção dos arquivos excluídos.
- Estratégia de armazenamento persistente para o ambiente de produção.
- Necessidade de versionamento das respostas do questionário.

## 6. Status de implementação

Esta decisão representa a modelagem lógica inicial do projeto.

As entidades, constraints e regras descritas deverão ser validadas durante a criação das migrations e dos testes de integração.

O job de exclusão física está previsto, mas não faz parte das primeiras entregas de implementação.

## 7. Documentos relacionados

- [Visão geral do projeto](../visao-geral.md)
- [Escopo do MVP](../mvp.md)
- [Modelagem de dados](../modelagem-dados.md)