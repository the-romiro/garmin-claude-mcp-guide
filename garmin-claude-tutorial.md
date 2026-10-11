# Conecte seu Garmin no Claude (Windows) — versão revisada

> Baseado no tutorial original do [@jp.runningclub](https://www.instagram.com/jp.runningclub/)
> (publicado no Notion e divulgado no Instagram), revisado e atualizado em **11/10/2026** contra a
> documentação atual do [garmin_mcp](https://github.com/Taxuspt/garmin_mcp), da
> [python-garminconnect](https://github.com/cyberjunky/python-garminconnect) e do
> [MCP / Claude Desktop](https://modelcontextprotocol.io/docs/develop/connect-local-servers).
> O resumo das mudanças está em [O que mudou em relação ao original](#o-que-mudou-em-relação-ao-original).

**Aviso antes de começar:** o `garmin_mcp` é um projeto da comunidade, **não oficial**. Ele usa uma
biblioteca que imita o login do app Garmin Connect. A Garmin não tem integração oficial com o
Claude, e uma mudança no login dela pode quebrar o conector de uma hora pra outra (isso já
aconteceu em 2025/2026).

---

## 1. Instala o Claude Desktop

Baixa em: **https://claude.ai/download**

Escolhe a versão pra Windows. Faz login com sua conta Claude (cria uma grátis se não tiver).

## 2. Cria seu projeto "Treinador" no Claude

Dentro do Claude Desktop:

- Clica em "**Projetos**" no menu lateral
- **"Criar projeto"**
- Nome: `Treinador` (ou outro que preferir)

Dica: Escreve as instruções do projeto pra ele se comportar como treinador. Exemplo (tem uma versão
mais completa no [Bônus](#bônus-configurando-o-projeto-treinador)):

> *"Você é meu treinador pessoal de corrida. Use meus dados do Garmin pra responder com base no
> que de fato fiz, não no que é genérico. Foque em volume semanal, intensidade, recuperação e
> progressão. Seja direto e prático."*

Pronto! Agora bora pro setup técnico.

---

## PARTE 1: Instalando Python e Git (8 min)

> Não precisa abrir o Prompt de Comando como administrador em nenhum passo. Um Prompt de Comando
> normal resolve, e é o mais seguro.

### Passo 1.1 — Instalar o Python 3.13

> **Mudou:** o original pedia Python 3.11. O README atual do `garmin_mcp` pede **Python 3.12 ou
> mais novo**, e é com essa versão que o projeto é testado e documentado. Hoje o 3.11 ainda
> instala (o projeto fixa a `garminconnect` 0.3.2, que aceita 3.10+), mas está fora do suportado e
> pode parar de funcionar quando o projeto atualizar a biblioteca: as versões 0.3.5+ da
> `garminconnect` já exigem 3.12.

1. Vai no site oficial: **https://www.python.org/downloads/windows/**
2. Procura por **Python 3.13.x** (qualquer versão 3.12 ou mais nova serve)
3. Baixa o **"Windows installer (64-bit)"**
4. Abre o instalador

**MUITO IMPORTANTE:** Na primeira tela do instalador, **MARCA a caixa "Add python.exe to PATH"**
(no rodapé).

5. Clica em "**Install Now**"
6. Espera terminar e clica em "**Close**"

> **Alternativa:** a partir do Python 3.14, o python.org oferece o novo **Python Install Manager**
> (comando `py`), e o instalador tradicional deixa de ser publicado no Python 3.16. Se você
> preferir esse caminho, siga as instruções da página de download. O resto do tutorial funciona
> igual.

### Passo 1.2 — Verificar a instalação do Python

1. Aperta `Tecla Windows`
2. Digita `cmd` e abre o **Prompt de Comando**
3. Vai abrir uma janela preta. Cola esse comando:

```shell
python --version
```

Se aparecer `Python 3.12.x` ou `Python 3.13.x`, deu certo!

Se aparecer erro tipo "não é reconhecido como comando", você esqueceu de marcar "Add python.exe to
PATH". Desinstala o Python e instala de novo marcando essa caixa.

Se aparecer uma versão **mais antiga** (ex.: `Python 3.11.x`), você tem outro Python instalado
antes no PATH. Desinstala o antigo pelo Painel de Controle, ou use `py -3.13` no lugar de `python`
no Passo 2.3.

### Passo 1.3 — Instalar o Git

1. Vai em: **https://git-scm.com/install/windows** (o link antigo `git-scm.com/download/win`
   redireciona pra cá)
2. Baixa o instalador 64-bit
3. Abre o instalador
4. **Aceita todas as opções padrão** (só vai clicando "Next")
5. Termina a instalação

> Atalho: quem tem `winget` pode instalar com `winget install --id Git.Git -e --source winget`.

**Fecha e abre o Prompt de Comando de novo** e verifica se funcionou:

```shell
git --version
```

Se aparecer `git version 2.x.x`, tá certo!

---

## PARTE 2: Instalando o conector Garmin (5 min)

### Passo 2.1 — Criar pasta do projeto

No Prompt de Comando, cola um comando de cada vez:

```shell
cd %USERPROFILE%
```

```shell
mkdir garmin_mcp_server
```

```shell
cd garmin_mcp_server
```

### Passo 2.2 — Baixar o programa

```shell
git clone https://github.com/Taxuspt/garmin_mcp.git .
```

(Não esquece o ponto `.` no final: ele faz baixar na pasta atual)

### Passo 2.3 — Criar ambiente Python isolado

```shell
python -m venv venv
```

```shell
venv\Scripts\activate
```

Você vai notar que o terminal agora começa com `(venv)`. Isso é normal e indica que tá tudo certo.

### Passo 2.4 — Instalar dependências

```shell
python -m pip install --upgrade pip
```

```shell
pip install -e .
```

Espera terminar (3-5 min). Quando aparecer `Successfully installed ... garmin-mcp...`, tá pronto!

Se der erro dizendo que nenhuma versão é compatível com o seu Python ("requires a different
Python" / "No matching distribution"), seu Python é antigo demais. Volta pro Passo 1.1.

---

## PARTE 3: Conectando com sua conta Garmin (2 min)

### Passo 3.1 — Autenticar

**No mesmo Prompt de Comando** (com `(venv)` no início), cola:

```shell
garmin-mcp-auth
```

**O que vai acontecer:**

1. Vai pedir seu **email do Garmin Connect**
2. Vai pedir sua **senha do Garmin** (não aparece nada quando digita, é normal!)
3. Se você tem **autenticação de 2 fatores (MFA)** ativada no Garmin, vai pedir o código. Esse
   passo **tem que** ser feito aqui no terminal: o Claude Desktop não consegue pedir o código de
   MFA sozinho.

Quando terminar com sucesso, aparece uma mensagem de autenticação bem-sucedida com o seu nome.

Os tokens de acesso ficam salvos em `C:\Users\SEUNOME\.garminconnect\garmin_tokens.json`. Segundo
o README do projeto, eles valem **cerca de 6 meses**.

Pra conferir se os tokens estão funcionando a qualquer momento:

```shell
garmin-mcp-auth --verify
```

> **Já tinha instalado uma versão antiga?** Os tokens antigos (`oauth1_token.json` /
> `oauth2_token.json`) não são compatíveis com o formato novo. Atualize (ver
> [Como atualizar](#como-atualizar-o-conector)) e rode `garmin-mcp-auth --force-reauth`.

---

## PARTE 4: Configurando o Claude Desktop (5 min)

### Passo 4.1 — Descobrir seu nome de usuário do Windows

No Prompt de Comando, cola:

```shell
echo %USERNAME%
```

Vai aparecer seu nome de usuário (exemplo: `joaopaulo`). **Anota porque você vai usar agora.**

### Passo 4.2 — Descobrir o caminho exato do garmin-mcp

No mesmo Prompt de Comando (com `(venv)` ativo), cola:

```shell
where garmin-mcp
```

Vai aparecer um caminho tipo:

```text
C:\Users\joaopaulo\garmin_mcp_server\venv\Scripts\garmin-mcp.exe
```

**Anota esse caminho inteiro**. Você vai usar no próximo passo.

### Passo 4.3 — Abrir o arquivo de configuração do Claude

Jeito recomendado (cria o arquivo se ele não existir e abre o certo, mesmo que o Claude tenha sido
instalado pela Microsoft Store):

1. No Claude Desktop, abre **Configurações** (Settings)
2. Vai na aba **Desenvolvedor** (Developer)
3. Clica em **"Editar configuração"** (Edit Config)
4. Vai abrir uma pasta com o arquivo `claude_desktop_config.json` selecionado
5. **Clica com botão direito → Abrir com → Bloco de Notas** (ou Notepad)

Jeito manual (igual ao original): `Tecla Windows + R`, cola `%APPDATA%\Claude` e aperta Enter.

> Se o arquivo não existir, **cria um novo arquivo** chamado exatamente
> `claude_desktop_config.json` (cuidado pra não salvar como `.json.txt`)

### Passo 4.4 — Substituir o conteúdo

> **Mudou:** o original colocava uma variável `GARMIN_COOKIES_FILE` no bloco `env`. Ela **não
> existe** no `garmin_mcp` atual e era ignorada, então saiu. O servidor já procura os tokens em
> `C:\Users\SEUNOME\.garminconnect` sozinho.

**Se o arquivo estiver vazio ou só tiver `{}`**, apaga tudo e cola exatamente isso:

```json
{
  "mcpServers": {
    "garmin-local": {
      "command": "C:\\Users\\SEUNOME\\garmin_mcp_server\\venv\\Scripts\\garmin-mcp.exe"
    }
  }
}
```

**Se o arquivo já tiver outras configurações** (outros servidores MCP ou `preferences`), **não
apague**: só adicione o bloco `"garmin-local": { ... }` dentro do `"mcpServers"` que já existe (ou
crie o `"mcpServers"` ao lado das outras chaves, separando com vírgula).

**MUITO IMPORTANTE — Atenção redobrada aqui:**

1. **Substitua `SEUNOME` pelo nome de usuário** que apareceu no Passo 4.1. O caminho tem que bater
   com o que apareceu no Passo 4.2.
2. **Use barras duplas `\\`** em todos os caminhos. Isso é exigência do formato JSON: com barra
   simples `\`, não funciona.

Exemplo: se seu usuário é `joaopaulo`, fica:

```json
{
  "mcpServers": {
    "garmin-local": {
      "command": "C:\\Users\\joaopaulo\\garmin_mcp_server\\venv\\Scripts\\garmin-mcp.exe"
    }
  }
}
```

> **Opcional:** se você salvou os tokens em outra pasta (`garmin-mcp-auth --token-path ...`),
> informe essa pasta com a variável `GARMINTOKENS`:
>
> ```json
> "garmin-local": {
>   "command": "C:\\Users\\joaopaulo\\garmin_mcp_server\\venv\\Scripts\\garmin-mcp.exe",
>   "env": {
>     "GARMINTOKENS": "C:\\Users\\joaopaulo\\.garminconnect"
>   }
> }
> ```

### Passo 4.5 — Salvar

Aperta `Ctrl + S` para salvar.

**IMPORTANTE:** Se aparecer caixa de diálogo perguntando o tipo de arquivo:

- **Salvar como tipo:** "Todos os Arquivos (\*.\*)"
- **Codificação:** UTF-8
- Mantém o nome `claude_desktop_config.json` (não vira `.json.txt`)

### Passo 4.6 — Reiniciar Claude

Fechar a janela no X **não basta**: no Windows o Claude continua rodando em segundo plano.

1. **Sai do Claude Desktop completamente:**
   - Vai na bandeja do sistema (canto direito da barra de tarefas, perto do relógio)
   - Acha o ícone do Claude
   - Clica com botão direito → "**Quit**" ou "**Sair**"
2. **Abre o Claude Desktop de novo** pelo menu Iniciar

### Passo 4.7 — Conferir se o servidor subiu

- Em **Configurações → Desenvolvedor**, o servidor `garmin-local` deve aparecer como **running**.
  Se aparecer **failed**, vai pra [Solução de problemas](#solução-de-problemas-windows).
- Numa conversa, clica no botão **"+"** no canto inferior esquerdo da caixa de mensagem →
  **Conectores** e confere se `garmin-local` está listado e ativado.

---

## PARTE 5: Testando

Abre seu projeto "**Treinador**" e pergunta:

> *"Quais foram meus últimos 5 treinos?"*

Na primeira vez, o Claude pede **permissão** pra usar a ferramenta do Garmin. Aprova (pode marcar
pra sempre permitir). Se ele responder com seus dados reais do Garmin, **DEU CERTO!**

---

## Perguntas matadoras pra começar

Agora que tá conectado, experimenta perguntar coisas como:

**Análise de performance:**

- *"Como tá meu volume semanal nos últimos 30 dias?"*
- *"Compara meu pace médio desse mês com o mês passado"*
- *"Quais treinos foram mais intensos essa semana?"*

**Recuperação e bem-estar:**

- *"Como tá meu sono em relação aos meus treinos?"*
- *"Tô em risco de overtraining? Analisa meu Body Battery e HRV"*
- *"Meu VO2 max tá evoluindo?"*

**Planejamento:**

- *"Monta um plano de 12 semanas pra uma maratona, considerando meu pace atual"*
- *"Qual deveria ser meu treino longo essa semana baseado no meu volume?"*
- *"Sugere ajustes na carga de treino pra próxima semana"*

**Provas:**

- *"Baseado no meu histórico, qual ritmo realista pra correr 10k em 45min?"*
- *"Estratégia de pace pra meia maratona"*

---

## Solução de problemas (Windows)

### Onde ficam os logs

- **Configurações → Desenvolvedor** → seleciona `garmin-local` → **"Abrir pasta de logs"**
  (Open Logs Folder)
- Ou direto em `%APPDATA%\Claude\logs`:
  - `mcp.log`: conexões e falhas em geral
  - `mcp-server-garmin-local.log`: saída do servidor do Garmin (é aqui que aparece o erro real)

### "Server disconnected" / servidor "failed" no Claude

**1.** Verifica se você usou **barras duplas** `\\` em TODOS os caminhos do JSON. Esse é o erro
mais comum no Windows.

**2.** Verifica se substituiu `SEUNOME` pelo seu usuário e se o caminho é idêntico ao do Passo 4.2.

**3.** Confirma que rodou o `garmin-mcp-auth` com sucesso (Parte 3). Teste com
`garmin-mcp-auth --verify`.

**4.** Se o log mostrar **"MFA authentication required but no interactive terminal available"**,
rode `garmin-mcp-auth` no Prompt de Comando (com `(venv)` ativo), digite o código de MFA e
reinicie o Claude.

**5.** Teste o servidor fora do Claude. No Prompt de Comando, cole o caminho do Passo 4.2 e aperte
Enter. Se ele mostrar um erro, é esse erro que precisa ser resolvido. Se ficar parado sem mostrar
nada, está funcionando (feche com `Ctrl + C`).

**6.** Abra os [logs](#onde-ficam-os-logs) e procure a última mensagem de erro.

### Erro de login (401, 403, "login failed") no `garmin-mcp-auth`

- Confere se o Python é 3.12+ e se `pip show garminconnect` mostra 0.3.x. As versões 0.2.x usam o
  login antigo (biblioteca `garth`, descontinuada), que não funciona mais. Isso acontece com
  instalações antigas do conector: atualize (abaixo).
- Atualiza o conector ([Como atualizar](#como-atualizar-o-conector)) e tenta
  `garmin-mcp-auth --force-reauth`.
- Se ainda falhar, olhe as [issues do garmin_mcp](https://github.com/Taxuspt/garmin_mcp/issues):
  quando a Garmin muda o login, costuma aparecer lá primeiro.

### "Python não é reconhecido como comando"

Você esqueceu de marcar **"Add python.exe to PATH"** na instalação. Solução:

1. Desinstala o Python pelo Painel de Controle
2. Reinstala marcando a caixa "Add python.exe to PATH" na primeira tela
3. Fecha e abre o Prompt de Comando

### "Git não é reconhecido"

Reinstala o Git mantendo as opções padrão. Depois **fecha e abre o Prompt de Comando** novamente.

### "garmin-mcp não encontrado"

Provavelmente você não ativou o ambiente virtual. No Prompt de Comando:

```shell
cd %USERPROFILE%\garmin_mcp_server
venv\Scripts\activate
```

Você deve ver `(venv)` no início da linha.

### Erro de permissão / "Acesso negado"

- Confere se você está dentro da sua pasta de usuário (`cd %USERPROFILE%\garmin_mcp_server`), e não
  em `C:\Windows\System32` (pasta padrão de quem abre o Prompt como administrador).
- Se o antivírus ou o **Smart App Control** do Windows 11 bloquear o `garmin-mcp.exe`, veja em
  **Segurança do Windows → Controle de aplicativos e navegador → Smart App Control**. O README do
  projeto recomenda, nesse caso, o caminho com `uv` instalado via `winget` (ver
  [Caminho alternativo](#caminho-alternativo-com-uv-menos-passos)) ou Docker.
- Só como último recurso, rode o Prompt de Comando como administrador.

### O arquivo de config foi salvo como `.json.txt`

No Windows, o Bloco de Notas adiciona `.txt` automaticamente às vezes. Solução:

1. Abre o Explorador de Arquivos
2. Vai em `%APPDATA%\Claude`
3. Acha o arquivo errado (vai aparecer como `claude_desktop_config.json.txt`)
4. **Renomeia** removendo o `.txt` do final

Se não aparecer a extensão `.txt`, ativa a visualização de extensões:

- **Windows 11:** Explorador de Arquivos → **Exibir** → **Mostrar** → marca **"Extensões de nomes
  de arquivos"**
- **Windows 10:** Explorador de Arquivos → aba **Exibir** → marca **"Extensões de nomes de
  arquivos"**

### Tokens expiraram (depois de ~6 meses)

Roda de novo:

```shell
cd %USERPROFILE%\garmin_mcp_server
venv\Scripts\activate
garmin-mcp-auth --force-reauth
```

Depois reinicia o Claude Desktop (Passo 4.6).

### Outro erro? Tenta isso ANTES de pedir ajuda

**Dica de ouro:** o Claude pode te ajudar a resolver erros de instalação dele mesmo. Funciona muito bem!

**Como fazer:**

1. No Claude Desktop, abre uma **nova conversa** (fora do projeto Treinador, pra não confundir contexto)
2. Cola algo assim:

> *"Tô seguindo um tutorial pra conectar meu Garmin ao Claude Desktop via MCP no Windows, usando o
> projeto github.com/Taxuspt/garmin_mcp. Tô travado no passo X e aparece esse erro: [COLA O ERRO
> AQUI]. Me ajuda a resolver passo a passo, considerando que sou leigo em tecnologia."*

3. Se o erro for no log do Claude Desktop, abre o arquivo `mcp-server-garmin-local.log` (ver
   [Onde ficam os logs](#onde-ficam-os-logs)), copia as últimas linhas e cola na conversa.
   **Antes de colar, apaga qualquer email, senha ou token que aparecer.**

O Claude consegue diagnosticar a maioria dos problemas e te dar o comando exato pra resolver.
Foi assim que o tutorial original foi montado: o autor usou o próprio Claude pra debugar tudo
enquanto instalava.

### Ainda não conseguiu?

O autor do tutorial original atende no Instagram: @jp.runningclub (Direct).

Manda:

- Print do erro
- Print dos logs (Configurações → Desenvolvedor → Abrir pasta de logs)
- Em qual passo travou
- Versão do Windows (10 ou 11)

**Nunca mande sua senha do Garmin nem o arquivo `garmin_tokens.json` pra ninguém.**

---

## Como atualizar o conector

De tempos em tempos (e sempre que parar de funcionar):

```shell
cd %USERPROFILE%\garmin_mcp_server
venv\Scripts\activate
git pull
pip install -e . --upgrade
```

Depois reinicia o Claude Desktop (Passo 4.6).

---

## Caminho alternativo com `uv` (menos passos)

O README atual do `garmin_mcp` recomenda usar o [`uv`](https://docs.astral.sh/uv/) em vez de
Python + Git + venv. O `uv` baixa sozinho o Python certo e o conector. Se você ainda não fez as
Partes 1 a 4, pode seguir só isto:

1. Instala o `uv` (Prompt de Comando):

   ```shell
   winget install --id=astral-sh.uv -e
   ```

   Fecha e abre o Prompt de Comando. Confere com `uvx --version`.

2. Autentica no Garmin (pede email, senha e MFA):

   ```shell
   uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth
   ```

3. Descobre o caminho do `uvx`:

   ```shell
   where uvx
   ```

4. No `claude_desktop_config.json` (Passo 4.3), usa este bloco, trocando o caminho pelo que
   apareceu no item 3 (com barras duplas):

   ```json
   {
     "mcpServers": {
       "garmin": {
         "command": "C:\\Users\\SEUNOME\\.local\\bin\\uvx.exe",
         "args": [
           "--python",
           "3.12",
           "--from",
           "git+https://github.com/Taxuspt/garmin_mcp",
           "garmin-mcp"
         ]
       }
     }
   }
   ```

5. Reinicia o Claude Desktop (Passo 4.6) e testa (Parte 5).

O README também cita uma **extensão de desktop** (arquivo `.dxt`) publicada na página de
[Releases](https://github.com/Taxuspt/garmin_mcp/releases), que dá pra instalar em
**Configurações → Extensões**. Quando este documento foi revisado, a página de releases não
mostrava nenhum arquivo publicado, então confira antes.

---

## É seguro?

**Razoavelmente, com ressalvas.** O conector roda **no seu PC**:

- Sua senha do Garmin é enviada **só pro login oficial da Garmin**, uma vez, no `garmin-mcp-auth`.
  Ela não fica salva; o que fica é um token.
- O token fica em `C:\Users\SEUNOME\.garminconnect\garmin_tokens.json`. **Trate esse arquivo como
  uma senha:** quem tiver ele acessa sua conta Garmin sem precisar de senha nem MFA.
- Os dados são lidos direto do servidor oficial do Garmin Connect.
- **Correção do original:** os dados que o Claude consulta (treinos, sono, HRV etc.) **são
  enviados pra Anthropic** como parte da conversa, igual a qualquer coisa que você digita no
  Claude. O conector não manda dados pra mais ninguém, mas eles saem do seu PC.
- **Falha de segurança CVE-2026-54447** (publicada em julho/2026): a `garminconnect` até a versão
  **0.3.4** podia salvar o arquivo de tokens com permissão de leitura pra outros usuários do
  computador (problema de sistemas tipo Linux/macOS). O `garmin_mcp` usa a 0.3.2, mas **aplica a
  mesma correção por conta própria**: depois do login ele restringe a pasta de tokens ao seu
  usuário. No Windows, a pasta `C:\Users\SEUNOME` já é privada do seu usuário por padrão. Mesmo
  assim, num PC compartilhado, não deixe outras pessoas usarem a sua conta do Windows.
- É uma biblioteca **não oficial**: a própria Garmin não aprova nem dá suporte a esse acesso.

Se algum dia quiser desconectar:

- Deleta a pasta `C:\Users\SEUNOME\garmin_mcp_server`
- Deleta a pasta `C:\Users\SEUNOME\.garminconnect`
- Tira o bloco `garmin-local` do config do Claude
- Apagar a pasta **não cancela** o token na Garmin. Se suspeitar que o token vazou, troque a senha
  da conta Garmin (isso derruba as sessões existentes).

---

## Bônus: Configurando o Projeto "Treinador"

Pra deixar o Claude ainda mais útil, configura **instruções personalizadas** no seu projeto. Sugestão:

```text
Você é meu treinador pessoal de corrida.

CONTEXTO SOBRE MIM:
- Idade: [SUA IDADE]
- Sexo: [M/F]
- Peso: [SEU PESO]
- Anos correndo: [QUANTOS ANOS]
- Melhor tempo recente em [DISTÂNCIA]: [TEMPO]
- Objetivo principal: [EX: maratona em outubro]
- Frequência semanal de treinos: [EX: 4x na semana]

COMO QUERO QUE VOCÊ ME AJUDE:
- Use meus dados reais do Garmin (via MCP) sempre que possível
- Foque em análises práticas, não teoria genérica
- Aponte sinais de overtraining ou subtreino
- Sugira ajustes específicos com base no que vê nos meus dados
- Seja direto, sem rodeios

EVITE:
- Conselhos genéricos do tipo "treine com consistência"
- Recomendações sem considerar meus dados
- Linguagem motivacional vazia
```

Cola isso nas **instruções do projeto** (dentro do projeto Treinador, na área de instruções).

---

## Extra: briefing diário automático (posts do @jppace__)

O @jppace__ publicou no Instagram posts mostrando um **briefing diário gerado
automaticamente** às 9h (projeção de maratona, HRV, sono, carga/ACWR, VO2max e uma "leitura do
dia"). A receita dele tem 3 peças:

1. **Uma IA que roda no seu computador e executa tarefas** (ele usa o Claude Code)
2. **O conector do seu relógio** (Garmin, ou Apple Health / Strava / Polar / Zepp, conforme o repo
   dele: `github.com/joaojardimp/briefing-automatico`)
3. **Um agendador:** launchd no Mac, Agendador de Tarefas no Windows, cron no Linux

O próprio post avisa: *"Garmin usa lib não-oficial. Pode quebrar a qualquer update."*

Prompt sugerido por ele (adapte os colchetes):

```text
Quero conectar meu [Garmin / Apple Watch / Coros / Polar / Amazfit] a você e receber um briefing
automático todo dia.

1. Use o conector [garmin_mcp / Apple Health / Strava OAuth / Polar Accesslink / Zepp via Apple
   Health] pra baixar os dados do dia: HRV, sono, readiness, carga, VO2max, peso e previsão de prova.

2. Gere um resumo curto com semáforos (verde/amarelo/vermelho) pra recuperação, carga e
   performance, fechando com a ação do dia.

3. Agende pra rodar sozinho todo dia às 9h e me avisar.

4. Separa em 2 camadas: script determinístico que CALCULA os números (pra não alucinar) e você
   interpretando por cima.

Me guia do zero, passo a passo. Não sei programar.
```

> Não verifiquei o repositório `joaojardimp/briefing-automatico`. Leia o código antes de rodar
> qualquer script agendado com acesso aos seus tokens.

---

## O que mudou em relação ao original

| # | Original | Atualizado | Por quê |
|---|----------|------------|---------|
| 1 | Python **3.11** | Python **3.12+** (recomendado 3.13) | O README do `garmin_mcp` pede 3.12+. O 3.11 ainda instala hoje (`garminconnect` fixada em 0.3.2, que aceita 3.10+), mas a `garminconnect` 0.3.5+ já exige 3.12. |
| 2 | Caixa "Add Python to PATH" | "Add python.exe to PATH" + nota sobre o Python Install Manager | Nome atual da opção. O instalador tradicional deixa de existir no Python 3.16. |
| 3 | `git-scm.com/download/win` | `git-scm.com/install/windows` (+ `winget`) | O link antigo agora redireciona. |
| 4 | Prompt de Comando "como administrador" em tudo | Prompt normal | Não é necessário, e como admin o terminal abre em `C:\Windows\System32`, o que confunde. |
| 5 | `env` com `GARMIN_COOKIES_FILE` | Removido; `GARMINTOKENS` opcional | `GARMIN_COOKIES_FILE` não existe no `garmin_mcp`. A variável suportada é `GARMINTOKENS` (padrão `~/.garminconnect`). |
| 6 | "Apaga tudo" no config | Apaga só se estiver vazio; senão, mescla | O arquivo pode ter outros servidores e preferências do Claude. |
| 7 | Abrir config via `Win + R` | **Configurações → Desenvolvedor → Editar configuração** (Win + R como alternativa) | Caminho oficial; cria o arquivo e acerta a pasta em qualquer tipo de instalação. |
| 8 | Logs em "Configurações → Servidores MCP locais → Ver Logs" | **Configurações → Desenvolvedor → Abrir pasta de logs**; `%APPDATA%\Claude\logs\mcp-server-garmin-local.log` | Nome atual da tela e local documentado dos logs. |
| 9 | — | Passo 4.7 (conferir "running" e o menu "+" → Conectores) | Forma atual de verificar se o servidor conectou. |
| 10 | — | `garmin-mcp-auth --verify` e `--force-reauth` | Opções atuais do comando. |
| 11 | — | Nota sobre MFA não funcionar dentro do Claude | Documentado no README (erro "no interactive terminal available"). |
| 12 | — | Tokens antigos incompatíveis; arquivo agora é `garmin_tokens.json` | Mudança de formato na `garminconnect` 0.3.0. |
| 13 | "Nenhum servidor de terceiros tem acesso aos seus dados" | Esclarecido que os dados consultados vão pra Anthropic | Tudo que o conector devolve entra na conversa com o Claude. |
| 14 | — | Nota sobre a CVE-2026-54447 | Afeta a `garminconnect` <= 0.3.4 (o `garmin_mcp` usa a 0.3.2), mas o `garmin_mcp` restringe as permissões dos tokens por conta própria (`secure_token_dir`). |
| 15 | — | Seções "Como atualizar", "Caminho alternativo com uv" e aviso de lib não oficial | O README atual recomenda `uv`/`uvx`; o próprio autor dos posts avisa que a lib pode quebrar. |
| 16 | "Extensões de nome de arquivo" em Exibir | Caminho separado pra Windows 10 e 11 | No Windows 11 a opção fica em Exibir → Mostrar. |
| 17 | Seção "Curtiu? Retribui assim" | Removida | O texto estava cortado no screenshot. |

### Fontes consultadas (11/10/2026)

- [Taxuspt/garmin_mcp: README](https://github.com/Taxuspt/garmin_mcp) e
  [pyproject.toml](https://raw.githubusercontent.com/Taxuspt/garmin_mcp/main/pyproject.toml)
- [cyberjunky/python-garminconnect: README](https://github.com/cyberjunky/python-garminconnect)
- [garminconnect no PyPI](https://pypi.org/project/garminconnect/) (0.3.2 aceita Python >= 3.10;
  0.3.5+ exige >= 3.12; versão mais recente: 0.3.17)
- [garmin_mcp: uv.lock e token_utils.py](https://github.com/Taxuspt/garmin_mcp/tree/main/src/garmin_mcp)
  (`garminconnect==0.3.2`; permissões dos tokens restritas ao dono)
- [garth no PyPI](https://pypi.org/project/garth/) ("Garth is deprecated and no longer maintained")
- [Advisory CVE-2026-54447 (garminconnect <= 0.3.4)](https://feed.craftedsignal.io/products/garminconnect--0.3.4/feed.xml)
- [MCP: Connect to local MCP servers (Claude Desktop, config e logs)](https://modelcontextprotocol.io/docs/develop/connect-local-servers)
- [Python Install Manager (python.org)](https://www.python.org/downloads/latest/pymanager)
- [Git for Windows](https://git-scm.com/install/windows)
- [Fórum Garmin: pedido de interface MCP oficial (não existe)](https://forums.garmin.com/apps-software/mobile-apps-web/f/garmin-connect-web/437504/plea-add-an-official-mcp-interface-to-garmin)
