# Runbook — SD Elements + Jira privado via Remote Integration Agent (RIA)

> Objetivo: criar uma integração nova em DEV, do zero, para exportar/sincronizar Countermeasures do SD Elements para um projeto Jira que não é acessível diretamente pelo SD Elements.

---

## 0. Como saber se este runbook é o seu cenário

Use este fluxo quando:

- o SD Elements não consegue acessar diretamente o Jira;
- o Jira está em rede interna/privada;
- existe uma máquina/VM que consegue acessar:
  - o SD Elements por HTTPS;
  - o Jira por HTTPS.

Arquitetura:

```text
SD Elements
    |
    | HTTPS / 443
    v
Remote Integration Agent (RIA)
    |
    | HTTPS / 443
    v
Jira DEV
```

O RIA busca os jobs no SD Elements, acessa o Jira internamente e devolve o resultado ao SD Elements.

---

# PARTE 1 — Antes de começar

Tenha em mãos:

- URL do SD Elements;
- acesso administrativo suficiente no SD Elements;
- URL do Jira DEV;
- Project Key do projeto Jira de destino;
- usuário/conta de serviço do Jira;
- senha/token/credencial dessa conta;
- acesso administrativo à máquina/VM onde o RIA será instalado.

A máquina do RIA precisa conseguir abrir:

```text
https://<seu-sd-elements>
https://<seu-jira-dev>
```

Normalmente isso significa saída HTTPS/TCP 443 para os dois destinos.

---

# PARTE 2 — Criar o Remote Agent no SD Elements

## Passo 1 — Entrar na área de integração

1. Acesse o SD Elements pelo navegador.
2. Clique no ícone de engrenagem / Settings.
3. Entre em **Integration**.
4. Abra a aba **Remote Agents**.

## Passo 2 — Criar o agente

1. Clique no botão `+` / **New Remote Agent**.
2. Escolha o sistema operacional da máquina onde o agente será instalado:
   - Windows; ou
   - Linux.
3. Dê um nome descritivo.

Exemplo:

```text
ria-jira-dev
```

O nome é apenas uma identificação administrativa.

4. Salve.

Após a criação, deve aparecer uma opção semelhante a:

```text
Download and install agent
```

Guarde também o identificador do RIA, caso a instalação Linux peça o `ria_id`.

---

# PARTE 3 — Instalar o RIA

Escolha somente o bloco correspondente ao sistema operacional da sua máquina.

---

## OPÇÃO A — Windows

### Passo 3A.1 — Baixar

No SD Elements:

```text
Settings
  -> Integration
     -> Remote Agents
```

Abra o agente criado e clique em:

```text
Download and install agent
```

Baixe o instalador.

### Passo 3A.2 — Instalar

1. Entre no Windows com uma conta que tenha permissão de Administrador.
2. Execute o instalador.
3. Se for a primeira instalação nessa máquina, deixe marcada a opção:

```text
Generate Quickstart Configuration
```

4. Continue o wizard.

O instalador pode solicitar a credencial da conta que será utilizada para criar/executar tarefas no **Windows Task Scheduler**.

O RIA usa tarefas agendadas para execuções:

- hourly;
- daily;
- weekly;
- monthly.

### Passo 3A.3 — Onde fica a configuração

Se precisar corrigir a configuração depois:

```text
config.txt
```

fica no diretório de instalação do Remote Integration Agent.

### Passo 3A.4 — Teste manual

Depois de termos configurado o conector Jira no SD Elements, o teste manual pode ser feito executando:

```text
integrate.bat
```

no diretório do RIA.

Se ocorrer erro, verificar:

```text
messages.log
```

no mesmo diretório.

**Não execute ainda se o conector Jira ainda não tiver sido criado.**

---

## OPÇÃO B — Linux

A documentação atual do SD Elements indica Python 3.12 para o instalador Linux atual.

### Passo 3B.1 — Confirmar Python

```bash
python3.12 --version
```

Esperado:

```text
Python 3.12.x
```

### Passo 3B.2 — Criar virtual environment

```bash
python3.12 -m venv ric-env
```

Ativar:

```bash
source ric-env/bin/activate
```

### Passo 3B.3 — Atualizar pip

```bash
pip install 'pip~=23.3.1'
```

### Passo 3B.4 — Baixar o pacote

O pacote deve ser obtido pela tela:

```text
Settings
  -> Integration
     -> Remote Agents
```

O nome será semelhante a:

```text
remote-integration-<VERSAO>.tar.gz
```

### Passo 3B.5 — Extrair

Exemplo:

```bash
mkdir -p ./sdetools
tar -xzvf remote-integration-<VERSAO>.tar.gz -C ./sdetools
cd sdetools
```

Confira:

```bash
ls
```

Deve existir algo semelhante a:

```text
dist/
sdetools-<VERSAO>-py3-none-any.whl
```

### Passo 3B.6 — Instalar

```bash
pip install --no-index --find-links=dist/ sdetools-<VERSAO>-py3-none-any.whl
```

### Passo 3B.7 — Validar

```bash
sderic help command_driver
```

Em releases mais antigas o executável pode aparecer como:

```bash
sderic.py
```

### Passo 3B.8 — Arquivo de configuração

O caminho padrão documentado é:

```text
~/.sdetools.cnf
```

Modelo:

```ini
[global]

# APIv2 connection string
sde_api_token=<TOKEN>@<HOST_SD_ELEMENTS>

# Remote Integration Agent ID
ria_id=<RIA_ID>

log_level=default

sde_validate_cert=True
command_params={"issue_tracker_validate_cert":"True","analysis_validate_cert":"True","ldap_validate_cert":"True"}
```

**Não invente TOKEN ou RIA_ID. Pegue os valores fornecidos pelo seu ambiente SD Elements.**

---

# PARTE 4 — Validar rede antes de configurar Jira

Na máquina do RIA, confirme que ela alcança os dois sistemas.

## Windows

PowerShell:

```powershell
Test-NetConnection <HOST_SD_ELEMENTS> -Port 443
Test-NetConnection <HOST_JIRA> -Port 443
```

Procure:

```text
TcpTestSucceeded : True
```

## Linux

```bash
curl -I https://<HOST_SD_ELEMENTS>
curl -I https://<HOST_JIRA>
```

ou:

```bash
nc -vz <HOST_SD_ELEMENTS> 443
nc -vz <HOST_JIRA> 443
```

Se a máquina não alcança um dos dois destinos, não adianta continuar para a sincronização: primeiro é necessário corrigir rede, firewall, proxy, DNS ou certificado.

---

# PARTE 5 — Preparar a conta do Jira

A conta que o SD Elements/RIA usa deve:

- ser membro/ter acesso ao projeto Jira;
- conseguir criar issues;
- conseguir aplicar labels/tags;
- conseguir atribuir issues;
- conseguir fazer transições;
- conseguir resolver/fechar issues.

Permissões Jira citadas pela documentação:

```text
CREATE_ISSUES
CLOSE_ISSUE
ASSIGN_ISSUES
TRANSITION_ISSUES
RESOLVE_ISSUES
```

Se houver sincronização de comentários/notas, podem ser necessárias também:

```text
CREATE_COMMENTS
EDIT_ALL_COMMENTS
ADMINISTER_PROJECTS
```

Observação: a documentação informa restrições para sincronização de comentários/notas quando a conexão usa RIA. Valide essa necessidade separadamente; não use isso como requisito para o primeiro teste.

---

# PARTE 6 — Criar a conexão Jira no SD Elements

Agora voltamos ao SD Elements.

## Passo 6.1 — Abrir integrações

Procure:

```text
Settings / Manage
  -> Integration
     -> Issue Tracker
```

A nomenclatura pode variar por release, mas a integração atual é chamada de **Issue Tracker**.

## Passo 6.2 — Nova conexão

1. Clique para adicionar uma nova conexão.
2. Escolha:

```text
Atlassian Jira
```

3. Defina um nome.

Exemplo:

```text
jira-dev
```

## Passo 6.3 — Informar endereço do Jira

Preencha os dados de conexão.

Normalmente:

```text
Protocol: HTTPS
Server: jira-dev.empresa.local
Context Root: <somente se o Jira usar um>
```

Exemplo sem context root:

```text
https://jira-dev.empresa.local
```

Exemplo com context root:

```text
https://servidor.empresa.local/jira
```

## Passo 6.4 — Marcar que o Jira é privado

Marque a opção equivalente a:

```text
This Issue Tracker server is hosted within a private network
and cannot be reached directly by SD Elements
```

Essa opção é o que faz a integração usar o Remote Integration Agent.

## Passo 6.5 — Credencial

Informe a conta Jira destinada à integração.

Dependendo da versão/configuração do Jira, o formulário pode pedir:

- username;
- password;
- token;
- OAuth 2.0.

Use o método efetivamente habilitado no Jira DEV.

---

# PARTE 7 — Associar o projeto SD Elements ao projeto Jira

Dentro da configuração do projeto/conector, informe:

```text
JIRA Project Key
```

Exemplo:

```text
APPSECDEV
```

O Project Key é a sigla usada nas issues.

Exemplo:

```text
APPSECDEV-123
```

Nesse caso:

```text
Project Key = APPSECDEV
```

---

# PARTE 8 — Definir o tipo de issue

O padrão documentado pelo SD Elements é:

```text
Bug
```

Mas você pode selecionar outro Issue Type suportado pelo projeto Jira.

Exemplos:

```text
Bug
Task
Story
```

Para o primeiro teste, use um tipo de issue que você sabe que pode ser criado manualmente pela conta de integração.

---

# PARTE 9 — Escolher quais Countermeasures serão exportadas

O SD Elements permite escolher entre opções como:

```text
Sync all Countermeasures
```

ou:

```text
Sync Risk Policy Countermeasures
```

Para um DEV controlado, evite começar exportando tudo se o projeto possuir muitos requisitos.

Você também pode filtrar por:

- prioridade mínima;
- status;
- fase;
- tags;
- verification status.

Exemplo de primeiro teste:

```text
Status: TODO
Phase: Requirements
```

ou utilize uma tag específica de teste, se disponível no projeto.

---

# PARTE 10 — Campos enviados ao Jira

Por padrão, a integração do SD Elements com Jira trabalha com:

```text
Summary
Description
Labels
Priority
```

A integração também suporta configuração avançada de campos.

Para o primeiro teste:

**não configure campos customizados se não forem obrigatórios no Jira.**

Primeiro faça o caminho mínimo funcionar.

---

# PARTE 11 — Status

A integração suporta sincronização de status nos dois sentidos.

Exemplo:

```text
SD Elements Countermeasure
        |
        v
Jira Issue
        |
   issue resolvida
        |
        v
próxima sincronização
        |
        v
SD Elements Countermeasure atualizada
```

Status Jira que não estiverem mapeados podem cair no significado de status `Incomplete` no SD Elements.

Para o primeiro teste, não complique o mapeamento. Use o fluxo padrão do projeto Jira e depois refine.

---

# PARTE 12 — Frequência

Configure o Sync Frequency.

Possibilidades usuais:

```text
Manual
Hourly
Daily
Weekly
Monthly
```

No Windows, o RIA utiliza o Windows Task Scheduler.

No Linux, é possível programar a execução via cron.

Para DEV, a melhor estratégia inicial é configurar/testar de forma controlada antes de deixar execução recorrente.

---

# PARTE 13 — Primeiro teste

Use **um projeto SD Elements de DEV e poucos Countermeasures**.

Checklist:

```text
[ ] RIA criado no SD Elements
[ ] RIA instalado
[ ] máquina alcança SD Elements:443
[ ] máquina alcança Jira:443
[ ] conta Jira válida
[ ] conta Jira possui permissões
[ ] conexão Jira criada
[ ] opção de private network marcada
[ ] Project Key correto
[ ] Issue Type válido
[ ] filtro de Countermeasures definido
```

---

## Windows — disparar teste

No diretório do RIA:

```bat
integrate.bat
```

Depois abra:

```text
messages.log
```

---

## Linux — disparar teste

Com o virtualenv ativo:

```bash
source ric-env/bin/activate
```

Execute o `command_driver` usando o arquivo de configuração definido para o ambiente.

Exemplo estrutural:

```bash
sderic command_driver -c ~/.sdetools.cnf
```

---

# PARTE 14 — Validar no Jira

Abra o projeto Jira DEV.

Procure as issues recém-criadas.

Confirme:

```text
Summary      -> preenchido
Description  -> preenchido
Labels       -> preenchido
Priority     -> preenchida/mapeada
Issue Type   -> correto
Project      -> correto
```

Se apareceu uma issue correspondente ao Countermeasure:

```text
SD Elements -> RIA -> Jira
```

está funcionando.

---

# PARTE 15 — Erros mais prováveis

## 401 / Unauthorized

Verificar:

- usuário;
- senha/token;
- método de autenticação;
- Basic Auth/OAuth configurado no Jira;
- conta bloqueada/expirada.

## 403 / Forbidden

Autenticação funcionou, mas a conta não possui alguma permissão necessária.

Verifique principalmente:

```text
CREATE_ISSUES
ASSIGN_ISSUES
TRANSITION_ISSUES
RESOLVE_ISSUES
CLOSE_ISSUE
```

## Timeout

Verificar:

```text
DNS
Firewall
Proxy
Rota
Porta 443
```

## SSL / Certificate

Se o Jira usa CA/certificado corporativo, a máquina do RIA precisa confiar nessa cadeia.

Evite desabilitar validação TLS como solução permanente.

## Issue não é criada, mas autenticação funciona

Verifique:

- Project Key;
- Issue Type;
- campos obrigatórios do Jira;
- filtros de Countermeasures;
- status selecionado;
- fase;
- prioridade mínima;
- tags.

Um campo obrigatório customizado no Jira é uma causa comum de falha ao criar issues por API.

## RIA executa, mas não há job

Verifique:

- conexão marcada como private network;
- projeto associado à conexão;
- frequência;
- filtros;
- existência de Countermeasures elegíveis.

---

# PARTE 16 — Ordem recomendada para fazer hoje

Faça exatamente nesta ordem:

```text
1. Criar RIA no SD Elements
2. Baixar RIA
3. Instalar na máquina
4. Validar acesso da máquina ao SD Elements
5. Validar acesso da máquina ao Jira
6. Confirmar conta Jira
7. Criar conexão Jira no SD Elements
8. Marcar Jira como private network
9. Informar credencial
10. Informar Project Key
11. Escolher Issue Type
12. Selecionar poucos Countermeasures
13. Configurar teste/sincronização
14. Executar RIA
15. Ler log
16. Abrir Jira e confirmar issue criada
17. Só depois refinar filtros, campos e frequência
```

---

## Regra principal

Não tente configurar tudo de uma vez.

O primeiro objetivo é somente provar:

```text
SD Elements
     |
     v
RIA
     |
     v
Jira DEV
     |
     v
1 issue criada com sucesso
```

Depois disso, refine:

- filtros;
- Risk Policy;
- fases;
- prioridades;
- status;
- campos customizados;
- periodicidade;
- mapeamento de workflow.
