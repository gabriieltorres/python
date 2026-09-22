# Base de Remediação de CWEs — Plano de Execução

## Objetivo

Criar uma base de remediação para os desenvolvedores consultarem quando uma CWE for identificada pelo Veracode.

A documentação não deve tentar cobrir todas as CWEs para todas as tecnologias existentes no ambiente. O objetivo é documentar as combinações que realmente aparecem no cenário real:

**Repositório → Stack/Tecnologia → CWE encontrada pelo Veracode → Orientação de remediação**

Exemplo:

- Repositório A utiliza Java + Spring Boot.
- O Veracode identificou CWE-117 nesse repositório.
- Portanto, deve existir documentação de remediação para **CWE-117 em Java/Spring Boot**.
- Se a CWE-117 nunca apareceu nas outras tecnologias do ambiente, inicialmente não é necessário produzir documentação para essas combinações.

---

## Fase 1 — Consolidar o inventário tecnológico

### Objetivo

Transformar o inventário já produzido em um catálogo confiável das tecnologias realmente utilizadas nos repositórios.

### Dados esperados

Para cada repositório, consolidar quando disponível:

- Linguagem.
- Framework.
- Versão da linguagem/JDK/runtime.
- Versão do framework.
- Build/package manager.
- Dependências ou componentes relevantes.
- Evidências utilizadas para identificar a tecnologia.
- Nível de confiança já existente no inventário: alto, médio ou baixo.

### Regra importante

Não inventar tecnologia ou versão ausente. Manter explicitamente como `não identificado` quando não houver evidência suficiente.

---

## Fase 2 — Consolidar os achados do Veracode

### Objetivo

Criar uma visão estruturada das CWEs que realmente existem no ambiente.

### Dados esperados

Sempre que possível, associar:

- Repositório/aplicação.
- CWE.
- Categoria ou nome da vulnerabilidade.
- Severidade/prioridade existente no resultado.
- Informações adicionais úteis fornecidas pelo Veracode.

O ponto principal desta fase é saber **qual CWE apareceu em qual repositório**.

---

## Fase 3 — Cruzar Veracode + inventário por repositório

### Objetivo

Juntar os dois conjuntos de informações.

Exemplo:

**Inventário**

`repo-a → Java → Spring Boot → JDK 17`

**Veracode**

`repo-a → CWE-117`

**Resultado do cruzamento**

`CWE-117 → Java → Spring Boot → JDK 17`

A IA não deve decidir sozinha se uma CWE é aplicável a uma tecnologia apenas por conhecimento geral. A evidência principal da combinação vem do ambiente real: **o Veracode encontrou a CWE naquele repositório e o inventário identificou a stack daquele repositório.**

---

## Fase 4 — Consolidar combinações únicas de CWE + stack

### Objetivo

Remover duplicações e descobrir quais documentos precisam ser produzidos.

Exemplo: se 30 repositórios Java/Spring Boot apresentam CWE-117, não é necessário criar 30 documentos iguais.

Pode ser consolidado como:

`CWE-117 + Java + Spring Boot`

Quando versões diferentes alterarem de forma relevante a forma de correção, a documentação poderá ser separada por versão.

### Saída esperada

Uma matriz semelhante a:

| CWE | Linguagem | Framework | Versão relevante | Repositórios afetados |
|---|---|---|---|---|
| CWE-117 | Java | Spring Boot | JDK 17 | repo-a, repo-b |
| CWE-80 | Java | Spring Boot | JDK 17 | repo-c |
| CWE-XXX | JavaScript | React | versão X | repo-d |

Essa matriz será a fila de geração da documentação.

---

## Fase 5 — Gerar a documentação de remediação

### Estrutura no Confluence

A estrutura pode seguir:

**Base de Remediação**
- **CWE-117**
  - Visão geral
  - Java / Spring Boot
  - Outras stacks, somente quando existirem no ambiente
- **CWE-80**
  - Visão geral
  - Java / Spring Boot
  - Outras stacks, somente quando existirem no ambiente

### Estrutura sugerida para cada combinação CWE + stack

1. **Identificação**
   - CWE.
   - Nome.
   - Linguagem/framework.
   - Versões relevantes.

2. **Resumo**
   - O que é a vulnerabilidade.
   - Qual é o risco.

3. **Como ela costuma ocorrer nessa stack**
   - Padrões de implementação que podem provocar o problema.
   - APIs/componentes relevantes.

4. **Orientação de remediação**
   - Abordagem recomendada.
   - O que deve ser evitado.
   - Pontos que o desenvolvedor precisa adaptar ao contexto da aplicação.

5. **Exemplo vulnerável**
   - Exemplo pequeno e diretamente relacionado ao problema.

6. **Exemplo corrigido**
   - Exemplo equivalente utilizando a abordagem recomendada.

7. **Como verificar**
   - O que revisar no código.
   - Testes recomendados.
   - Novo scan do Veracode quando aplicável.

8. **Referências**
   - MITRE/CWE.
   - Veracode, quando houver documentação aplicável.
   - Documentação oficial da linguagem/framework/biblioteca.
   - OWASP ou outra referência técnica reconhecida quando pertinente.

9. **Status de validação**
   - `Validado documentalmente`
   - `Validado em implementação`

---

## Fase 6 — Validação técnica documental

### Objetivo

Não será necessário clonar todos os repositórios, implementar cada correção e executar scans em massa antes de publicar a base.

A primeira validação será **documental/técnica**.

Verificar se:

- A remediação realmente corresponde à CWE.
- As APIs e mecanismos sugeridos existem na stack/versão indicada.
- A recomendação está fundamentada em fontes confiáveis.
- Os exemplos de código são coerentes com a recomendação.
- Não existem afirmações importantes sem fundamento.
- A IA não inventou métodos, bibliotecas, configurações ou comportamentos.

Uma orientação que passar por essa etapa recebe o status:

**Validado documentalmente**

Esse status significa que a recomendação possui fundamento técnico, mas **não garante que o Veracode deixará de apontar o finding em toda implementação possível**.

---

## Fase 7 — Validação em implementação e melhoria contínua

A validação mais forte acontece quando a documentação for utilizada em um caso real.

Fluxo:

1. O desenvolvedor encontra uma CWE.
2. Consulta a base.
3. Aplica a orientação ao contexto da aplicação.
4. Executa os testes necessários.
5. O Veracode executa um novo scan.
6. Se o finding for eliminado, a solução real pode ser registrada como evidência.
7. Se não funcionar, Segurança + Desenvolvimento investigam o caso.
8. O aprendizado é incorporado à documentação.

Quando houver evidência real de implementação/rescan, a orientação pode receber:

**Validado em implementação**

Isso transforma a base em documentação viva: ela começa tecnicamente fundamentada e fica progressivamente mais aderente ao ambiente conforme os times a utilizam.

---

# Estratégia para uso de IA

Não executar todo o processo em um único prompt.

Trabalhar por fases e por lotes pequenos permite:

- Revisar as saídas.
- Identificar alucinações.
- Corrigir o processo antes de escalar.
- Manter rastreabilidade.
- Evitar geração desnecessária de documentação.

A IA deve receber dados estruturados sempre que possível e deve ser instruída a **não inferir informações ausentes**.

## Princípio central

**Não gerar uma biblioteca teórica de todas as combinações possíveis entre CWE e tecnologia. Gerar uma base orientada pelas ocorrências reais do ambiente.**

Assim, se existem 20 tecnologias no inventário, mas a CWE-117 só apareceu em Java/Spring Boot e em outra stack, inicialmente serão produzidas somente essas duas remediações.

---

# Resultado esperado

Ao final do processo haverá:

1. Um catálogo tecnológico consolidado.
2. Uma base consolidada de CWEs encontradas pelo Veracode.
3. Um mapa `repositório → stack → CWE`.
4. Uma lista única de combinações `CWE + stack`.
5. Documentações de remediação específicas para essas combinações.
6. Validação documental das recomendações.
7. Evolução da base através de implementações e rescans reais.
8. Conteúdo preparado para publicação no Confluence e posterior integração ao Golden Path.

---

## Contexto para outra IA

Este documento resume uma discussão sobre a criação de uma base interna de remediação de vulnerabilidades para desenvolvedores.

Ao continuar o trabalho a partir deste arquivo, **não reinicie o desenho da solução do zero**. Considere as sete fases acima como o fluxo acordado. Ajude a executar uma fase de cada vez, preferencialmente por lotes pequenos, mantendo evidências, níveis de confiança e referências técnicas e evitando inferências sem suporte.
