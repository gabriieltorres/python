# Handoff de Contexto — Projeto AppSec de Padronização de Remediações

> **Finalidade deste arquivo**
>
> Este documento é um **handoff de contexto para o Copilot conversacional** no ambiente corporativo.
> Ele registra o problema, o raciocínio, as decisões já tomadas, a arquitetura conceitual do projeto, exemplos e o ponto atual do trabalho.
>
> **Não é uma skill, não é um prompt de execução para Gemini CLI e não é uma documentação final de segurança.**
>
> Ao continuar este trabalho, preserve as decisões abaixo e não reinicie a discussão do zero.

---

## 1. Contexto atual

O ambiente possui **milhares de aplicações** hospedadas em repositórios GitHub.

Existe pipeline de CI e análise **SAST com Veracode**.

Hoje o fluxo é, de forma simplificada:

```text
Desenvolvedor faz alteração
        ↓
Pipeline CI
        ↓
Veracode SAST
        ↓
Finding / CWE
        ↓
Desenvolvedor precisa corrigir
```

Exemplo:

```text
Aplicação Java / Spring Boot
        ↓
Veracode identifica CWE-89
        ↓
Pipeline aponta/bloqueia conforme política atual
        ↓
Dev procura uma solução
```

O problema é que atualmente o desenvolvedor pode corrigir de diferentes maneiras:

- Veracode Fix;
- documentação externa;
- pesquisa na internet;
- IA;
- experiência própria;
- solução encontrada em outro projeto;
- qualquer outra implementação que faça o finding desaparecer.

Se dois projetos possuem a mesma stack e a mesma CWE, um desenvolvedor pode utilizar o método A e outro o método B.

O AppSec **não possui hoje um catálogo normativo dizendo qual padrão de remediação deve ser utilizado naquela stack e naquele contexto**.

Também não existe necessariamente visibilidade central sobre qual método de correção foi adotado em cada aplicação.

---

# 2. Problema que queremos resolver

O objetivo **não é criar documentação genérica sobre CWE**.

Não queremos apenas algo como:

```text
CWE-89 — SQL Injection

Recomendação:
Utilize queries parametrizadas.
```

Isso é genérico demais.

Queremos responder:

> **Dentro das tecnologias e padrões do nosso ambiente, qual é a forma aprovada de corrigir esta CWE?**

Portanto, a unidade principal de conhecimento será aproximadamente:

```text
STACK + CWE + CONTEXTO
        ↓
PADRÃO DE REMEDIAÇÃO APROVADO
```

Exemplo conceitual:

```text
CWE: CWE-89
Linguagem: Java 17
Framework: Spring Boot
Componente: Spring Data JPA
Contexto: acesso a banco via JPA
        ↓
Padrão de remediação aprovado
```

Outro exemplo fictício:

```text
CWE: CWE-XXX
Linguagem: Java 17
Framework: Micronaut
Componente/Contexto: gRPC
        ↓
Padrão aprovado correspondente
```

Os exemplos acima são ilustrativos. Não significam que essas tecnologias estejam confirmadas no ambiente.

---

# 3. Por que não usar a documentação genérica antiga como base normativa

Existe material genérico sobre algumas CWEs produzido anteriormente com auxílio de IA.

Esse material:

- pode servir eventualmente como referência;
- não deve ser tratado automaticamente como padrão corporativo;
- não representa necessariamente as stacks reais;
- não considera obrigatoriamente os frameworks e componentes utilizados internamente;
- não passou pelo processo de validação que este projeto pretende estabelecer.

A nova base deverá nascer da combinação entre:

```text
stack real
+
CWE
+
contexto técnico
+
análise de código
+
remediação proposta
+
validação
+
aprovação AppSec
```

A IA poderá produzir o **candidato**, mas não declarar unilateralmente que algo é padrão corporativo.

---

# 4. Visão geral das fases

```text
FASE 1
Inventário das stacks reais
        ↓
FASE 2
Inventário/priorização das CWEs do Veracode
        ↓
FASE 3
Matriz STACK × CWE
        ↓
FASE 4
Encontrar caso real ou construir laboratório
        ↓
FASE 5
IA analisa e propõe remediação
        ↓
FASE 6
Aplicar + build + testes + novo scan Veracode
        ↓
FASE 7
Revisão AppSec / técnica
        ↓
FASE 8
Publicar padrão aprovado
        ↓
FASE FUTURA
Golden Path / enforcement
```

**Ponto atual do projeto: FASE 1.**

Não avançar prematuramente para enforcement.

---

# 5. FASE 1 — Inventário tecnológico

## Objetivo

Descobrir quais linguagens, frameworks e componentes relevantes para remediação SAST realmente existem nos milhares de repositórios.

Não partir de uma planilha/documentação possivelmente desatualizada se os próprios repositórios podem fornecer evidências mais confiáveis.

A ideia é percorrer os repositórios do GitHub e consultar principalmente arquivos que descrevem:

- linguagem;
- versão;
- framework;
- dependências;
- mecanismo de build;
- componentes que podem alterar a forma de corrigir uma vulnerabilidade.

## Não é necessário ler todo o código-fonte

Priorizar arquivos determinísticos de manifesto, dependência, build e configuração.

Exemplos:

### Java / JVM

```text
pom.xml
build.gradle
build.gradle.kts
settings.gradle
gradle.properties
```

Esses arquivos podem revelar, por exemplo:

```text
Java
Spring Boot
Spring Data JPA
Hibernate
Spring Security
JDBC
Kafka
drivers de banco
etc.
```

### JavaScript / TypeScript

```text
package.json
package-lock.json
yarn.lock
pnpm-lock.yaml
```

Podem revelar:

```text
TypeScript
Angular
React
Node.js
Express
NestJS
Prisma
etc.
```

### .NET

```text
*.csproj
*.fsproj
*.sln
packages.lock.json
```

Podem revelar:

```text
C#
ASP.NET Core
Entity Framework Core
Dapper
etc.
```

### Python

```text
pyproject.toml
requirements.txt
Pipfile
poetry.lock
```

### Go

```text
go.mod
go.sum
```

### Rust

```text
Cargo.toml
```

### PHP

```text
composer.json
```

### Ruby

```text
Gemfile
Gemfile.lock
```

### Evidências complementares

```text
Dockerfile
arquivos de configuração de framework
manifests
arquivos de build
arquivos de gerenciamento de dependências
```

A lista não é exaustiva.

---

# 6. O que registrar por repositório

O inventário deve preservar a relação entre **repositório e stack**.

Exemplo:

```text
Repository:
pagamentos-api

Language:
Java 17

Framework:
Spring Boot 3.x

Data Access:
Spring Data JPA
Hibernate

Security:
Spring Security

Database/Driver:
Oracle

Build:
Gradle

Evidence:
build.gradle
```

Outro exemplo:

```text
Repository:
portal-cliente

Language:
TypeScript

Framework:
Angular

Build:
npm

Evidence:
package.json
```

Sempre que possível registrar:

- repositório;
- linguagem;
- versão da linguagem;
- framework principal;
- versão do framework;
- componentes/frameworks secundários relevantes;
- mecanismo de acesso a dados/ORM;
- bibliotecas que possam alterar uma futura estratégia de remediação;
- build;
- arquivo/evidência que sustentou a identificação.

**Não inferir tecnologia apenas pelo nome do repositório.**

---

# 7. Resultado consolidado esperado da Fase 1

Depois da coleta por repositório, produzir uma visão consolidada.

Exemplo:

```text
Java
├── Spring Boot
│   ├── Spring Data JPA
│   ├── Hibernate
│   └── JDBC
└── Quarkus

C#
└── ASP.NET Core
    ├── Entity Framework Core
    └── Dapper

TypeScript
├── Angular
├── React
└── NestJS

Python
├── FastAPI
└── Django
```

A finalidade não é criar um inventário de absolutamente qualquer ferramenta existente.

O foco são tecnologias/componentes que podem **influenciar como uma CWE deve ser remediada no código**.

Exemplo:

```text
Java
Spring Boot
JPA
JDBC
```

podem ser altamente relevantes.

Já:

```text
Jenkins
Docker
Helm
```

podem aparecer no inventário geral, mas não necessariamente devem virar dimensão da matriz de remediação de uma CWE de código.

A relevância deve ser avaliada conforme o tipo de vulnerabilidade.

---

# 8. Persistência do inventário

Como existem milhares de aplicações, não depender da memória/janela de contexto de uma IA.

Persistir os resultados incrementalmente.

Possíveis formatos:

```text
CSV
JSON
```

Exemplo:

```csv
repository,language,language_version,framework,framework_version,data_access,evidence
pagamentos-api,Java,17,Spring Boot,3.x,Spring Data JPA,pom.xml
portal-cliente,TypeScript,5.x,Angular,18,,package.json
cliente-api,C#,8,ASP.NET Core,8,Entity Framework Core,cliente-api.csproj
```

O processo ideal deve permitir retomada caso seja interrompido.

---

# 9. FASE 2 — CWEs do Veracode

Após o inventário tecnológico, levantar as CWEs relevantes ao SAST/Veracode.

Há duas dimensões diferentes:

## A. CWEs dentro do escopo/capacidade relevante do Veracode

Identificar quais CWEs fazem sentido dentro do universo que será tratado.

## B. CWEs que efetivamente aparecem no ambiente

Consultar findings/histórico do Veracode para descobrir quais CWEs realmente ocorreram nas aplicações.

Exemplo fictício:

| CWE | Ocorrências | Aplicações afetadas |
|---|---:|---:|
| CWE-89 | 142 | 31 |
| CWE-79 | 97 | 22 |
| CWE-77 | 40 | 12 |
| CWE-117 | 55 | 18 |

Não assumir esses números; são apenas exemplos.

Mesmo uma CWE que ainda não apareceu poderá futuramente precisar de padrão, caso seja considerada relevante.

---

# 10. FASE 3 — Matriz Stack × CWE

Depois de conhecer:

```text
STacks existentes
+
CWEs relevantes
```

construir a matriz.

Exemplo conceitual:

| Stack | CWE-89 | CWE-77 | CWE-79 | CWE-117 |
|---|---|---|---|---|
| Java + Spring Boot + JPA | X | X | - | X |
| Java + Spring Boot + JDBC | X | X | - | X |
| TypeScript + Angular | - | - | X | X |
| .NET + ASP.NET + EF Core | X | X | - | X |

O `X` representa uma combinação para a qual pode ser necessário construir um padrão.

Não é necessário produzir artificialmente documentação para combinações sem sentido técnico.

A matriz deve refletir contexto real.

---

# 11. A stack pode precisar de subcontextos

Não assumir:

```text
Spring Boot + CWE-89 = exatamente uma única implementação
```

Pode existir:

```text
Spring Boot + CWE-89
├── Contexto A: Spring Data JPA
│       → padrão aprovado A
│
├── Contexto B: JdbcTemplate
│       → padrão aprovado B
│
└── Contexto C: Native Query
        → padrão aprovado C
```

Isso continua sendo padronização.

A diferença é que existe um **conjunto controlado de abordagens aprovadas**, aplicáveis a contextos específicos.

A decisão sobre permitir um ou vários padrões deve ser técnica e deliberada.

---

# 12. FASE 4 — Encontrar código real vulnerável

Para cada combinação priorizada:

```text
Stack + CWE
```

procurar uma aplicação real que tenha apresentado aquela CWE.

Exemplo:

```text
Java 17
Spring Boot
Spring Data JPA
CWE-89
```

Fluxo:

```text
Veracode mostra ocorrência
        ↓
identificar aplicação
        ↓
clonar repositório
        ↓
localizar finding
        ↓
entender contexto real
```

O objetivo é trabalhar em cima de código representativo do ambiente, e não apenas produzir uma resposta teórica.

---

# 13. FASE 5 — IA como construtora do candidato

A IA deverá analisar:

- finding;
- CWE;
- arquivo/linha;
- fluxo relevante;
- linguagem;
- versão;
- framework;
- bibliotecas;
- componentes disponíveis;
- padrões já utilizados na aplicação;
- contexto arquitetural necessário.

Então deverá propor uma remediação.

Exemplo:

```text
CWE-89
+
Java 17
+
Spring Boot
+
Spring Data JPA
+
código vulnerável real
        ↓
IA analisa
        ↓
candidato a remediação
```

IMPORTANTE:

> Uma prática encontrada no repositório não é automaticamente segura só porque já é utilizada internamente.

A IA deve avaliar segurança e adequação técnica.

---

# 14. FASE 6 — Validação da correção

A correção não vira padrão apenas porque parece correta.

Fluxo desejado:

```text
Código vulnerável
        ↓
IA propõe correção
        ↓
aplicar em branch/laboratório
        ↓
build
        ↓
testes
        ↓
Veracode SAST novamente
        ↓
finding eliminado?
        ↓
revisão técnica
```

Critérios podem incluir:

```text
✓ compila/builda
✓ testes existentes continuam passando
✓ comportamento funcional preservado
✓ finding deixa de ser detectado
✓ não introduz vulnerabilidade óbvia alternativa
✓ solução é compatível com a arquitetura
✓ AppSec revisou
```

O desaparecimento do finding no Veracode é evidência importante, mas não deve ser o único critério de segurança.

---

# 15. Quando não existir código real com aquela CWE

Pode acontecer:

```text
Stack existe
+
CWE é relevante
+
nenhuma aplicação atual possui finding conhecido
```

Nesse caso, considerar um **projeto mínimo/laboratório representativo da stack**.

Exemplo:

```text
Java 17
Spring Boot 3.x
Spring Data JPA
banco compatível
```

Criar deliberadamente um cenário vulnerável:

```text
laboratório vulnerável
        ↓
Veracode detecta CWE
        ↓
IA propõe correção
        ↓
aplicar correção
        ↓
novo scan
        ↓
finding eliminado
        ↓
revisão AppSec
        ↓
candidato aprovado
```

Isso permite preparar padrões antes que a vulnerabilidade apareça em produção.

---

# 16. FASE 7 — Aprovação

A IA **não aprova** o padrão.

Modelo:

```text
IA
↓
candidato técnico
↓
validação automatizada
↓
Veracode
↓
revisão técnica / especialista quando necessário
↓
AppSec
↓
PADRÃO APROVADO
```

O padrão aprovado passa a fazer parte da base corporativa.

---

# 17. FASE 8 — Modelo da documentação de um padrão

Um padrão poderá ter estrutura semelhante a:

```text
ID:
ARP-JAVA-SPRING-CWE89-001

Status:
Approved

CWE:
CWE-89 — SQL Injection

Linguagem:
Java 17

Framework:
Spring Boot 3.x

Componente:
Spring Data JPA

Contexto aplicável:
[descrever quando utilizar]

Padrão aprovado:
[descrever a implementação]

Alternativas aprovadas:
[se existirem]

Não permitido:
[abordagens conhecidamente inadequadas]

Exemplo vulnerável:
[código]

Exemplo corrigido:
[código]

Justificativa de segurança:
[por que a solução mitiga a CWE]

Validação:
- Build: sucesso
- Testes: sucesso
- Veracode SAST antes: CWE detectada
- Veracode SAST depois: finding eliminado
- Revisão AppSec: aprovada

Evidências:
[referências]

Versão do padrão:
1.0

Histórico:
[data / alteração / responsável ou processo de aprovação]
```

O nome/ID acima é apenas exemplo e poderá ser redefinido.

---

# 18. Exemplo visual completo

Exemplo puramente ilustrativo:

```text
CWE-89
SQL Injection

STACK
Java 17
Spring Boot 3.x
Spring Data JPA

CONTEXTO A
Uso de Repository/JPA
        ↓
Padrão aprovado A

CONTEXTO B
Uso de JdbcTemplate
        ↓
Padrão aprovado B

CONTEXTO C
Uso inevitável de Native Query
        ↓
Padrão aprovado C
```

Cada padrão deverá possuir:

```text
contexto
solução
exemplo
restrições
validação
evidência
status
versão
histórico
```

---

# 19. Base resultante

Ao final das fases de construção, poderá existir uma estrutura lógica semelhante a:

```text
security-remediation-patterns/

CWE-089/
├── java-spring-jpa.md
├── java-spring-jdbc.md
└── dotnet-efcore.md

CWE-079/
├── angular.md
└── react.md

CWE-117/
├── java-logback.md
└── dotnet-serilog.md
```

A estrutura definitiva ainda deverá ser desenhada.

O ponto importante é que a documentação seja **específica o suficiente para orientar uma implementação real**, mas organizada para manutenção e automação futura.

---

# 20. Golden Path — etapa futura

**Não é o foco atual.**

Primeiro:

```text
descobrir stacks
↓
descobrir/priorizar CWEs
↓
construir matriz
↓
criar padrões
↓
validar
↓
documentar
```

Somente depois pensar em enforcement.

Visão futura:

```text
Aplicação:
Java + Spring Boot + JPA

Finding:
CWE-89

Padrão corporativo:
ARP-JAVA-SPRING-CWE89-001

Código utiliza padrão aprovado?
        ↓
SIM → segue pipeline
NÃO → possível bloqueio/enforcement
```

A intenção futura é transformar a orientação AppSec em um **Golden Path verificável**, evitando que cada desenvolvedor implemente uma solução arbitrária para a mesma classe de problema.

A política exata de bloqueio, exceções e enforcement ainda não deve ser definida nesta fase.

---

# 21. Princípios já decididos

## 21.1 Não criar catálogo genérico de CWE

O valor está em:

```text
CWE + stack + contexto
```

e não somente na CWE.

## 21.2 IA propõe; AppSec aprova

Nenhuma resposta de IA vira automaticamente padrão.

## 21.3 Usar evidência real

Sempre que possível:

```text
finding real
+
código real
+
stack real
+
novo scan
```

## 21.4 Não analisar milhares de códigos integralmente sem necessidade

Na Fase 1, utilizar primeiro manifests/build/dependencies/configuração.

## 21.5 Preservar rastreabilidade

Registrar:

```text
qual stack
qual CWE
qual contexto
qual solução
como foi validada
qual versão
qual histórico
```

## 21.6 Não confundir ferramenta com padrão

Veracode Fix, IA, documentação externa etc. podem ajudar a chegar à solução.

Nenhuma dessas fontes, isoladamente, define o padrão corporativo.

---

# 22. O que NÃO fazer agora

Não começar por:

- escrever correções para centenas de CWEs;
- criar Golden Path;
- criar bloqueio na pipeline;
- obrigar um método antes de validar a matriz;
- considerar material antigo gerado por IA como padrão aprovado;
- pedir para IA inventar todas as stacks;
- analisar todo o código de milhares de aplicações quando manifests são suficientes;
- criar combinações `CWE × tecnologia` que não façam sentido.

---

# 23. Próximo passo imediato

Estamos atualmente aqui:

```text
┌─────────────────────────────────────┐
│ FASE 1 — INVENTÁRIO DAS STACKS      │
└─────────────────────────────────────┘
```

Próxima discussão/trabalho:

> **Definir e executar a estratégia para percorrer os repositórios GitHub e produzir um inventário confiável de linguagens, versões, frameworks e componentes relevantes para remediação SAST.**

A saída deve ser estruturada e persistente.

Depois disso:

```text
FASE 2 → CWEs
FASE 3 → matriz
FASE 4+ → remediações
```

---

# 24. Instrução para continuidade no Copilot

Ao receber este documento:

1. Considere todo o conteúdo acima como contexto já discutido.
2. Não peça para reconstruir a motivação do projeto.
3. Não reinicie propondo documentação genérica de CWE.
4. Não avance diretamente para Golden Path/enforcement.
5. Preserve a separação das fases.
6. Estamos atualmente na **Fase 1 — Inventário Tecnológico**.
7. Ajude a transformar a próxima etapa em um processo executável e controlado.
8. Quando uma decisão nova for tomada, considere atualizar este documento para preservar o histórico do projeto.

---

## Resumo em uma linha

> **Construir, a partir das stacks reais e das CWEs relevantes do ambiente, padrões corporativos de remediação validados tecnicamente, para futuramente transformá-los em Golden Paths verificáveis no pipeline.**
