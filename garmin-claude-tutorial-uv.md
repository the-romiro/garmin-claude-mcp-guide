# Conecte seu Garmin no Claude (Windows) — versão com `uv`

> Variante do [tutorial principal](garmin-claude-tutorial.md) que usa o
> [`uv`](https://docs.astral.sh/uv/) no lugar de `pip` + `venv`. Revisado em **11/10/2026** contra o
> README do [garmin_mcp](https://github.com/Taxuspt/garmin_mcp), que recomenda o `uv` como forma
> padrão de instalação.

## Por que `uv`?

- **Não precisa instalar o Python.** O `uv` baixa sozinho a versão certa (3.12) e deixa ela isolada,
  sem mexer em nenhum Python que você já tenha.
- **Não tem "Add to PATH", `venv` nem `activate`.** Menos passos, menos erros.
- **Na opção A, também não precisa do Git.**
- É o caminho que o próprio `garmin_mcp` documenta.

**Aviso:** o `garmin_mcp` é um projeto da comunidade, **não oficial**. Uma mudança no login da
Garmin pode quebrar o conector a qualquer momento. Detalhes em
[É seguro?](garmin-claude-tutorial.md#é-seguro)

## Qual opção escolher

| | Opção A: `uvx` (recomendada) | Opção B: cópia local com `uv sync` |
|---|---|---|
| Precisa de Git | Não | Sim |
| Onde fica o código | No cache do `uv` (você não mexe) | Numa pasta sua (`garmin_mcp_server`) |
| Atualizar | `uv cache clean` | `git pull` + `uv sync` |
| Indicado para | Quem só quer usar | Quem quer ver/alterar o código ou travar uma versão |

Na dúvida, vá de **Opção A**.

---

## Antes de tudo (vale para as duas opções)

### Passo 0.1 — Claude Desktop e projeto "Treinador"

Faça os passos [1](garmin-claude-tutorial.md#1-instala-o-claude-desktop) e
[2](garmin-claude-tutorial.md#2-cria-seu-projeto-treinador-no-claude) do tutorial principal:
instalar o Claude Desktop (**https://claude.ai/download**) e criar o projeto "Treinador".

### Passo 0.2 — Instalar o `uv`

1. Aperta `Tecla Windows`, digita `cmd` e abre o **Prompt de Comando** (não precisa ser como
   administrador)
2. Cola:

```shell
winget install --id=astral-sh.uv -e
```

3. **Fecha e abre o Prompt de Comando de novo** (senão ele não acha o comando novo)
4. Confere:

```shell
uvx --version
```

Se aparecer um número de versão (ex.: `0.x.x`), deu certo!

> **Sem `winget`?** Use o instalador oficial, colando no Prompt de Comando:
>
> ```shell
> powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
> ```
>
> Ele instala em `C:\Users\SEUNOME\.local\bin`. Feche e abra o Prompt de Comando depois.
>
> Se o Windows 11 tiver o **Smart App Control** ligado, o README do `garmin_mcp` recomenda
> instalar pelo `winget` (ou `scoop install main/uv`), porque o instalador via PowerShell pode ser
> bloqueado.

### Passo 0.3 — Descobrir o caminho do `uv`

O Claude Desktop nem sempre enxerga o PATH, então vamos usar o caminho completo:

```shell
where uvx
```

Vai aparecer algo como um destes:

```text
C:\Users\joaopaulo\AppData\Local\Microsoft\WinGet\Links\uvx.exe
C:\Users\joaopaulo\.local\bin\uvx.exe
```

**Anota o caminho que apareceu** (se aparecer mais de um, usa o primeiro). Na Opção A você vai
precisar dele.

---

## Opção A: `uvx` (recomendada)

### A.1 — Conectar com sua conta Garmin

No Prompt de Comando, cola (é uma linha só):

```shell
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth
```

Na primeira vez demora um pouco: o `uv` baixa o Python 3.12, o conector e as dependências. Depois:

1. Vai pedir seu **email do Garmin Connect**
2. Vai pedir sua **senha do Garmin** (não aparece nada quando digita, é normal!)
3. Se você tem **MFA** ativado, vai pedir o código. Isso **tem que** ser feito aqui no terminal: o
   Claude Desktop não consegue pedir o código de MFA.

Quando terminar, aparece uma mensagem de autenticação bem-sucedida com o seu nome. Os tokens ficam
em `C:\Users\SEUNOME\.garminconnect` e valem **cerca de 6 meses**.

Esse passo também deixa o conector guardado no cache do `uv`, então o Claude abre ele rápido
depois.

Pra conferir os tokens a qualquer momento:

```shell
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --verify
```

### A.2 — Configurar o Claude Desktop

1. No Claude Desktop: **Configurações → Desenvolvedor → Editar configuração** (Settings →
   Developer → Edit Config)
2. Abre o `claude_desktop_config.json` com o **Bloco de Notas**
3. Se o arquivo estiver vazio ou só tiver `{}`, apaga tudo e cola o bloco abaixo. Se já tiver
   outras configurações, só adiciona o `"garmin": { ... }` dentro do `"mcpServers"` (ver
   [Passo 4.4](garmin-claude-tutorial.md#passo-44--substituir-o-conteúdo) do tutorial principal).

```json
{
  "mcpServers": {
    "garmin": {
      "command": "C:\\Users\\SEUNOME\\AppData\\Local\\Microsoft\\WinGet\\Links\\uvx.exe",
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

4. **Troca o caminho do `"command"`** pelo que apareceu no [Passo 0.3](#passo-03--descobrir-o-caminho-do-uv),
   **com barras duplas** `\\`. Exemplo: `C:\Users\joaopaulo\.local\bin\uvx.exe` vira
   `"C:\\Users\\joaopaulo\\.local\\bin\\uvx.exe"`.
5. Salva com `Ctrl + S` (tipo "Todos os arquivos", codificação UTF-8, sem virar `.json.txt`)

Agora pula para [Reiniciar e testar](#reiniciar-e-testar).

---

## Opção B: cópia local com `uv sync`

Esta é a tradução direta do tutorial principal: em vez de `python -m venv` + `pip install`, o `uv`
cria o ambiente e instala tudo usando as versões travadas no `uv.lock` do projeto.

### B.1 — Instalar o Git

Faz o [Passo 1.3](garmin-claude-tutorial.md#passo-13--instalar-o-git) do tutorial principal
(ou `winget install --id Git.Git -e --source winget`). Fecha e abre o Prompt de Comando depois.

### B.2 — Baixar o programa

No Prompt de Comando, um comando de cada vez:

```shell
cd %USERPROFILE%
```

```shell
git clone https://github.com/Taxuspt/garmin_mcp.git garmin_mcp_server
```

```shell
cd garmin_mcp_server
```

### B.3 — Instalar com `uv`

```shell
uv sync --python 3.12 --no-dev
```

Isso baixa o Python 3.12 (se precisar), cria a pasta `.venv` dentro de `garmin_mcp_server` e
instala o conector. Não precisa ativar nada.

### B.4 — Conectar com sua conta Garmin

```shell
uv run garmin-mcp-auth
```

Mesmo fluxo da [Opção A](#a1--conectar-com-sua-conta-garmin): email, senha, código de MFA.
Pra conferir depois: `uv run garmin-mcp-auth --verify`.

### B.5 — Descobrir o caminho do conector

```shell
echo %CD%\.venv\Scripts\garmin-mcp.exe
```

Vai aparecer algo como:

```text
C:\Users\joaopaulo\garmin_mcp_server\.venv\Scripts\garmin-mcp.exe
```

### B.6 — Configurar o Claude Desktop

Abre o `claude_desktop_config.json` como na [Opção A](#a2--configurar-o-claude-desktop) e usa:

```json
{
  "mcpServers": {
    "garmin-local": {
      "command": "C:\\Users\\SEUNOME\\garmin_mcp_server\\.venv\\Scripts\\garmin-mcp.exe"
    }
  }
}
```

Troca o caminho pelo do Passo B.5, **com barras duplas** `\\`. Repare que a pasta é `.venv` (com
ponto), diferente do tutorial com `pip`.

> O README do `garmin_mcp` mostra outra forma, com `"command": "uv"` e
> `"args": ["--directory", "C:\\Users\\SEUNOME\\garmin_mcp_server", "run", "garmin-mcp"]`. Também
> funciona, mas apontar direto pro `.exe` evita depender do `uv` toda vez que o Claude abre.

---

## Reiniciar e testar

1. **Sai do Claude Desktop completamente:** bandeja do sistema (perto do relógio) → botão direito
   no ícone do Claude → **Quit / Sair**. Fechar no X não basta.
2. Abre o Claude Desktop de novo.
3. Em **Configurações → Desenvolvedor**, o servidor (`garmin` ou `garmin-local`) deve aparecer como
   **running**.
4. No projeto **Treinador**, pergunta:

> *"Quais foram meus últimos 5 treinos?"*

Aprova a permissão quando o Claude pedir. Se ele responder com seus dados reais, **DEU CERTO!**

Ideias de perguntas e instruções de treinador:
[Perguntas matadoras](garmin-claude-tutorial.md#perguntas-matadoras-pra-começar) e
[Bônus](garmin-claude-tutorial.md#bônus-configurando-o-projeto-treinador).

---

## Como atualizar

**Opção A:**

```shell
uv cache clean
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --verify
```

O `uv cache clean` apaga a cópia guardada; o segundo comando baixa a versão mais nova e já confere
os tokens. Depois reinicia o Claude Desktop.

**Opção B:**

```shell
cd %USERPROFILE%\garmin_mcp_server
git pull
uv sync --python 3.12 --no-dev
```

Depois reinicia o Claude Desktop.

**Atualizar o próprio `uv`:** `winget upgrade --id astral-sh.uv` (se instalou pelo `winget`) ou
`uv self update` (se usou o instalador do PowerShell).

---

## Tokens expiraram (depois de ~6 meses)

**Opção A:**

```shell
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --force-reauth
```

**Opção B:**

```shell
cd %USERPROFILE%\garmin_mcp_server
uv run garmin-mcp-auth --force-reauth
```

Depois reinicia o Claude Desktop.

---

## Solução de problemas

### "uv" ou "uvx" não é reconhecido como comando

Fecha **todas** as janelas do Prompt de Comando e abre de novo. Se continuar, reinstala o `uv`
(Passo 0.2) e reinicia o computador.

### Servidor aparece como "failed" / "Server disconnected"

1. Confere o caminho do `"command"`: tem que ser **exatamente** o do Passo 0.3 (Opção A) ou B.5
   (Opção B), com **barras duplas** `\\`.
2. Erro **"Failed to spawn process"** ou **"No such file or directory"** no log: o Claude não achou
   o `uvx`. Usa o caminho completo, como no Passo 0.3, em vez de só `"uvx"`.
3. Log com **"MFA authentication required but no interactive terminal available"**: roda o comando
   de autenticação (A.1 ou B.4) no Prompt de Comando, digita o código e reinicia o Claude.
4. Testa o servidor fora do Claude, colando no Prompt de Comando:
   - Opção A: `uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp`
   - Opção B: o caminho do Passo B.5

   Se aparecer erro, é ele que precisa ser resolvido. Se ficar parado sem mostrar nada, está
   funcionando (fecha com `Ctrl + C`).
5. Logs: **Configurações → Desenvolvedor → Abrir pasta de logs**, arquivo
   `mcp-server-garmin.log` (Opção A) ou `mcp-server-garmin-local.log` (Opção B).

### Smart App Control bloqueia o `uv` / `uvx`

No Windows 11 com Smart App Control ligado, o `uv.exe`/`uvx.exe` pode ser bloqueado (no
Visualizador de Eventos aparece "CodeIntegrity Error 3077"). O README do `garmin_mcp` sugere:

- reinstalar o `uv` pelo `winget` ou `scoop` (Passo 0.2) e usar o caminho completo do `uvx.exe`; ou
- usar a **Opção B**, apontando o Claude direto pro `garmin-mcp.exe`; ou
- rodar via Docker (ver README do projeto).

### Erro de login (401, 403, "login failed")

- Atualiza o conector ([Como atualizar](#como-atualizar)) e tenta `--force-reauth`.
- Se continuar, olha as [issues do garmin_mcp](https://github.com/Taxuspt/garmin_mcp/issues):
  quando a Garmin muda o login, costuma aparecer lá primeiro.

### O arquivo de config virou `.json.txt`

Veja [esta seção](garmin-claude-tutorial.md#o-arquivo-de-config-foi-salvo-como-jsontxt) do tutorial
principal.

### Outro erro

Use a [dica de ouro](garmin-claude-tutorial.md#outro-erro-tenta-isso-antes-de-pedir-ajuda): abra
uma conversa nova no Claude, diga que está usando o `garmin_mcp` com `uv` no Windows e cole o erro
(sem senhas, emails ou tokens).

---

## Como desinstalar

1. Tira o bloco `garmin` / `garmin-local` do `claude_desktop_config.json` e reinicia o Claude
2. Deleta a pasta `C:\Users\SEUNOME\.garminconnect` (seus tokens)
3. Opção A: `uv cache clean`. Opção B: deleta a pasta `C:\Users\SEUNOME\garmin_mcp_server`
4. (Opcional) Remove o `uv`: `winget uninstall --id astral-sh.uv`, ou apaga `uv.exe`, `uvx.exe` e
   `uvw.exe` de `C:\Users\SEUNOME\.local\bin` se usou o instalador do PowerShell

Apagar os arquivos **não cancela** o token na Garmin. Se achar que ele vazou, troque a senha da
conta Garmin.

---

## Fontes (11/10/2026)

- [garmin_mcp: README](https://github.com/Taxuspt/garmin_mcp) (instalação com `uvx`, `--python 3.12`,
  Smart App Control, MFA) e [`uv.lock`](https://github.com/Taxuspt/garmin_mcp/blob/main/uv.lock)
- [uv: instalação no Windows](https://docs.astral.sh/uv/getting-started/installation/)
- [uv: ferramentas e cache do `uvx`](https://docs.astral.sh/uv/concepts/tools/)
- [MCP: Connect to local MCP servers (Claude Desktop)](https://modelcontextprotocol.io/docs/develop/connect-local-servers)
