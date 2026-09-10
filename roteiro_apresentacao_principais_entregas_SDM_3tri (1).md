# Roteiro de Apresentação --- Principais Entregas SDM --- 3º TRI

## 1. Abertura

Boa tarde, pessoal.

Falando um pouco das principais entregas do SDM nesse terceiro tri, a
primeira delas foi o **novo fluxo de modelagem**.

------------------------------------------------------------------------

## 2. Novo Fluxo de Modelagem

-   O novo fluxo já foi apresentado para a **Auditoria**.
-   Tivemos o **OK da Auditoria** e estamos na fase de implementação.
-   A partir desse novo fluxo:
    -   a **modelagem passa a ser feita pelo time de Arquitetura**;
    -   nosso time fica concentrado na **validação dos requisitos
        referentes a AppSec**;
    -   a Engenharia já recebe os requisitos e fazemos a validação, sem
        executar toda a etapa de modelagem anterior.

### Frameworks e requisitos

Dentro desse trabalho, também fizemos uma revisão dos frameworks e
requisitos de segurança:

-   **9 frameworks**, de um total de **X**;
-   **102 requisitos de segurança**;
-   desses 102, **X serão inicialmente bloqueantes**;
-   os demais entram como **informativos**, podendo evoluir para
    bloqueio nas próximas ondas.

------------------------------------------------------------------------

## 3. Escopo de Tecnologias

Outro ponto foi o levantamento do nosso escopo de tecnologias, para
passar essa expansão para Arquitetura de forma mais estruturada.

-   **192 tecnologias** identificadas;
-   **58 cobertas pelo SDE**;
-   aproximadamente **30% de cobertura**.

Para as tecnologias ainda não cobertas, fiz uma análise separando:

-   tecnologias que possuem **esteira de CI**;
-   tecnologias que **não possuem esteira de CI**.

Isso nos dá um norte melhor para priorizar as próximas expansões.

------------------------------------------------------------------------

## 4. Expansão de Cobertura

Dentro dessa expansão, já estamos trabalhando com **APIGEE e CAAPI**.

Hoje, o bloqueio acontece apenas no **Portal Tech**. A ideia agora é
levar essa validação também para as esteiras.

Assim, quando uma API ou CAAPI gerar uma nova release, a **modelagem no
SDE passa a ser cobrada dentro desse fluxo**.

------------------------------------------------------------------------

## 5. Validador de Requisitos do SDE

Em paralelo, estamos avaliando um **validador automatizado de
requisitos**.

A ideia é automatizar parte dessa validação e permitir que a gente
consiga **ampliar a quantidade de requisitos avaliados sem aumentar
proporcionalmente o esforço operacional**.

Também existe todo o trabalho de preparação do SDE para o novo fluxo:

-   organização de torre e sigla;
-   automações;
-   integrações com ferramentas de validação;
-   demais ajustes necessários para iniciar o fluxo de forma
    estruturada.

------------------------------------------------------------------------

## 6. Skill de Modelagem SDLC --- Jornada Tech

Outra entrega importante foi o desenvolvimento da **Skill de Modelagem
SDLC da Jornada Tech**.

Antes, quando chegava um card da Jornada Tech, era necessário realizar
manualmente uma série de consultas para chegar à modelagem no SD
Elements.

Hoje, a skill automatiza praticamente todo esse caminho.

### Fluxo da skill

1.  Passo para a skill o **card do Jira**.
2.  Ela consulta o **fórum de Arquitetura**.
3.  Busca a **ArqCore**.
4.  Identifica a **DAP**.
5.  Verifica os **componentes envolvidos**.
6.  Consulta o **SD Elements**.
7.  Para componentes já modelados, retorna o **link das
    Countermeasures**.
8.  Para componentes ainda não modelados, prepara o processo e entrega
    as informações organizadas no Jira.

### Quando o componente não possui profile

A skill gera um arquivo chamado **Modelagem.txt**.

O desenvolvedor copia esse conteúdo para o **Copilot** e responde às
perguntas. Dessa forma, em vez de marcar uma call e realizar manualmente
a entrevista de modelagem, o próprio fluxo conduz essa etapa.

Por trás, as respostas são vinculadas às **IDs do questionário do SD
Elements**.

Ao final:

1.  é gerado um **JSON**;
2.  o desenvolvedor anexa o JSON na Jornada Tech;
3.  o card é passado novamente para a skill;
4.  ela identifica o JSON e recupera as respostas;
5.  a modelagem é criada.

------------------------------------------------------------------------

## 7. Resultados da Skill

No período de **X a X**:

-   **474 cards analisados**;
-   **6.680 componentes verificados**;
-   aproximadamente **31,2 segundos de tempo médio por análise**;
-   cerca de **67 mil cliques economizados**;
-   aproximadamente **351 horas economizadas**;
-   equivalente a aproximadamente **44 dias de trabalho**, considerando
    8 horas por dia;
-   aproximadamente **98,8% de redução de esforço operacional**.

### Comparação de esforço

O processamento dos **6.680 componentes** foi realizado pela skill em
aproximadamente **4 horas**.

Manual: aproximadamente **355 horas**\
Skill: aproximadamente **4 horas**\
Economia: aproximadamente **351 horas**

------------------------------------------------------------------------

## 8. Fechamento

A iniciativa da skill já foi compartilhada com o pessoal de Arquitetura
para avaliação de possível utilização em outros fluxos.

Hoje, continuo utilizando a solução nas minhas Jornadas Tech.

**De forma geral, essas foram as principais entregas do terceiro tri.**

------------------------------------------------------------------------

## Pontos para confirmar antes da apresentação

-   [ ] Total de frameworks: **X**
-   [ ] Quantidade de requisitos inicialmente bloqueantes: **X**
-   [ ] Período utilizado para medir os 474 cards: **X a X**
-   [ ] Confirmar nomenclatura oficial: **APIGEE / CAAPI**
-   [ ] Confirmar nomenclatura oficial: **SDM / SDE / SD Elements**
