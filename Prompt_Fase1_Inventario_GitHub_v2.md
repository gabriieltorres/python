# PROMPT — Fase 1: Inventário Tecnológico dos Repositórios GitHub

## 1. Contexto

Estamos executando a **Fase 1 de um projeto de AppSec para padronização corporativa de remediações de vulnerabilidades SAST**.

O ambiente possui um número elevado de aplicações armazenadas em repositórios dentro de um grupo/organização no GitHub.

Existe uma pipeline de CI com análise SAST utilizando Veracode, porém **Veracode, CWEs, findings e remediações NÃO fazem parte desta fase**.

O objetivo atual é exclusivamente construir um inventário confiável das stacks tecnológicas realmente utilizadas nos repositórios.

Este inventário será utilizado posteriormente para cruzamento com as CWEs relevantes do ambiente e construção de padrões de remediação específicos por stack.

## 2. Objetivo desta execução

Percorrer todos os repositórios acessíveis dentro do grupo/organização GitHub informado e identificar as stacks tecnológicas utilizadas.

A análise deve utilizar evidências encontradas nos próprios arquivos dos repositórios.

O resultado final deverá permitir responder:
- Quais linguagens existem no ambiente?
- Quais versões são utilizadas?
- Quais frameworks e versões existem?
- Quais mecanismos de acesso a dados/ORM são utilizados?
- Quais componentes relevantes para desenvolvimento e futura remediação SAST existem?
- Em quais repositórios cada stack foi encontrada?
- Quantas aplicações/componentes utilizam cada stack?

## 3. Restrição crítica — NÃO clonar repositórios

Esta análise deverá ser realizada em modo **READ-ONLY**, utilizando os recursos de consulta disponíveis no GitHub/API/ferramentas autorizadas no ambiente.

**NÃO CLONE REPOSITÓRIOS.**

Clone será utilizado somente em fase posterior, quando uma combinação específica **Stack + CWE + repositório** for selecionada para análise detalhada.

Nesta fase:
- não executar `git clone`;
- não criar cópias locais completas;
- não modificar código;
- não criar branches ou commits;
- não abrir Pull Requests;
- não modificar configurações.

## 4. Descoberta do escopo

Antes da análise:
1. Identifique o grupo/organização GitHub alvo.
2. Liste todos os repositórios acessíveis.
3. Determine a quantidade total.
4. Identifique, quando disponível, ativos, arquivados, forks, templates, bibliotecas, aplicações, infraestrutura/configuração e documentação.
5. Não exclua silenciosamente nenhum repositório.
6. Caso algum repositório não seja analisado, registre o motivo.
7. Trabalhe incrementalmente e permita retomada.

## 5. Estratégia

Não analisar todo o código-fonte quando isso não for necessário.

Priorize arquivos capazes de fornecer evidências determinísticas sobre linguagem, versão, framework, dependências, build, acesso a dados, ORM e componentes relevantes.

Consulte arquivos adicionais apenas quando manifests/build/configurações não forem suficientes.

## 6. Arquivos prioritários

### Java / JVM
- `pom.xml`
- `build.gradle`
- `build.gradle.kts`
- `settings.gradle`
- `gradle.properties`

Buscar Java/versão, Spring Boot, Spring Framework, Quarkus, Micronaut, Jakarta EE, Spring Data, JPA, Hibernate, JDBC, Spring Security, drivers e componentes relevantes.

### JavaScript / TypeScript
- `package.json`
- `package-lock.json`
- `yarn.lock`
- `pnpm-lock.yaml`

Buscar JavaScript, TypeScript, Node.js, Angular, React, Vue, Express, NestJS, Next.js, Prisma, Sequelize e componentes relevantes.

### .NET
- `*.csproj`
- `*.fsproj`
- `*.sln`
- `packages.lock.json`

Buscar C#, F#, runtime/versão, ASP.NET Core, Entity Framework Core, Dapper e componentes relevantes.

### Python
- `pyproject.toml`
- `requirements.txt`
- `Pipfile`
- `poetry.lock`

Buscar Python/versão, Django, Flask, FastAPI, SQLAlchemy e componentes relevantes.

### Outros
- Go: `go.mod`, `go.sum`
- Rust: `Cargo.toml`
- PHP: `composer.json`
- Ruby: `Gemfile`, `Gemfile.lock`

Quando necessário, considerar `Dockerfile`, configurações específicas de frameworks, manifests e outros arquivos que forneçam evidência confiável.

A lista NÃO é exaustiva. Tecnologias não previstas devem ser identificadas normalmente.

## 7. Informações por repositório/componente

Identificar, quando possível:
- Repository
- Component_Path
- Repository_Type
- Language
- Language_Version
- Framework
- Framework_Version
- Data_Access
- ORM
- Relevant_Components
- Build_Tool
- Database_Driver
- Evidence_Files
- Confidence
- Scan_Status
- Notes

Dê atenção especial a componentes que possam futuramente alterar a forma adequada de remediar vulnerabilidades SAST.

## 8. Não transformar qualquer ferramenta em dimensão de stack

Diferencie tecnologias que podem alterar a implementação da correção de ferramentas auxiliares.

Java, Spring Boot, JPA, Hibernate e JDBC podem ser relevantes para remediação. Jenkins, Docker, Helm e Kubernetes podem ser registrados, mas não devem automaticamente ser tratados como dimensões principais da stack de remediação.

## 9. Evidência e confiança

Não invente informações ausentes. Toda identificação relevante deve possuir evidência.

Quando não for possível determinar algo, use `Not Identified`.

Não adivinhe versões.

Confiança:
- **HIGH** — explicitamente encontrada em manifesto/build/configuração confiável.
- **MEDIUM** — sustentada por dependências ou múltiplas evidências indiretas fortes.
- **LOW** — inferência indireta que precisa de revisão.

Resultados LOW devem ir também para revisão manual.

## 10. Monorepositórios

Um repositório pode conter múltiplas aplicações/stacks. Não classifique o repositório inteiro pela primeira tecnologia encontrada.

Exemplo:
- `repo-x | /frontend | TypeScript | Angular`
- `repo-x | /backend | Java | Spring Boot`
- `repo-x | /worker | Python | FastAPI`

Preserve `repository + component_path + stack`.

## 11. Classificação dos repositórios

Quando possível:
- Application
- Library
- SDK
- Infrastructure
- Configuration
- Documentation
- Template
- Unknown

Não descarte silenciosamente outros tipos.

## 12. Processamento incremental

Como existem muitos repositórios:
- processe incrementalmente;
- preserve resultados;
- mantenha controle de processados e falhas;
- permita retomada sem repetir tudo.

JSON/CSV temporários ou checkpoints são permitidos internamente, mas NÃO são a entrega final.

## 13. Entrega final — UM ÚNICO EXCEL

A entrega humana final deverá ser **um único arquivo `.xlsx`**, com:

### `01_Inventario`
Inventário detalhado, uma linha por repositório/componente.

### `02_Stacks`
Visão consolidada das combinações tecnológicas e quantidade de aplicações/componentes.

### `03_Tecnologias`
Catálogo único de tecnologias, categoria, versões e ocorrências.

Categorias possíveis: Language, Framework, Data Access, ORM, Library/Component, Build Tool, Database/Driver, Other.

### `04_Nao_Analisados`
Repositórios/componentes não analisados, motivo, erro/status e notas.

### `05_Revisao_Manual`
Baixa confiança, ambiguidades, versões conflitantes, evidências contraditórias, estruturas incomuns e tecnologias não classificadas.

### `06_Resumo`
Totais: repositórios encontrados/processados/não analisados, componentes identificados, monorepos, linguagens, frameworks, principais stacks, itens para revisão e percentual concluído.

## 14. Controle de qualidade

Antes de finalizar:
1. Remova duplicidades indevidas.
2. Preserve componentes diferentes de monorepos.
3. Normalize nomes equivalentes.
4. Não misture linguagem, framework, ORM, build e infraestrutura.
5. Preserve versões relevantes.
6. Mantenha evidências.
7. Identifique baixa confiança.
8. Informe erros de acesso.
9. Informe repositórios sem stack identificável.
10. Não esconda resultados inconclusivos.

## 15. Fora do escopo

Nesta fase, NÃO:
- executar Veracode;
- consultar findings;
- pesquisar CWE por aplicação;
- correlacionar CWE com stack;
- corrigir vulnerabilidades;
- analisar profundamente código vulnerável;
- utilizar Veracode Fix;
- criar padrão de remediação;
- criar Golden Path;
- implementar enforcement;
- modificar pipeline;
- bloquear aplicações;
- clonar repositórios.

## 16. Próximas fases — apenas contexto

`Fase 1: Inventário tecnológico → Fase 2: CWEs/ocorrências Veracode → Fase 3: Matriz Stack × CWE → selecionar combinação específica → identificar repositório real → somente então clonar o repositório específico → analisar código → IA propor remediação → validar → novo scan Veracode → AppSec aprovar → padrão corporativo.`

Não antecipe essas etapas.

## 17. Critério de sucesso

A Fase 1 estará concluída quando existir um único Excel confiável que responda:

> **Quais stacks de desenvolvimento realmente existem nos repositórios do ambiente, onde elas estão e quais evidências sustentam essa identificação?**

## 18. Antes de iniciar

1. Confirme quais recursos disponíveis permitem consultar o grupo/organização GitHub em read-only.
2. Confirme que não será necessário clonar.
3. Determine a quantidade de repositórios acessíveis.
4. Defina estratégia incremental/checkpoint.
5. Informe resumidamente a estratégia.
6. Somente então inicie o inventário.

Não solicite acesso de escrita.
