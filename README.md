# garmin-claude-mcp-guide

Guia passo a passo, em português, para conectar sua conta **Garmin Connect** ao **Claude Desktop**
no **Windows** usando o servidor MCP da comunidade
[Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp). Com ele, o Claude passa a ler seus
treinos, sono, HRV, Body Battery, VO2max e outros dados, e pode atuar como seu "treinador" de
corrida.

Duas versões do guia:

- **[Tutorial com `uv`](garmin-claude-tutorial-uv.md)** (recomendado): menos passos, não precisa
  instalar o Python nem mexer com `venv`.
- **[Tutorial com `pip`](garmin-claude-tutorial.md)**: o fluxo clássico com Python + Git + `venv`,
  mais próximo do tutorial original.

## Para quem é

Para quem nunca programou. O guia explica cada comando e traz uma seção de solução de problemas
para os erros mais comuns no Windows.

## O que você vai fazer

1. Instalar o Claude Desktop e criar um projeto "Treinador"
2. Instalar Python 3.12+ e Git
3. Baixar e instalar o conector `garmin_mcp`
4. Fazer login na Garmin (com suporte a MFA) e gerar os tokens
5. Configurar o `claude_desktop_config.json`
6. Testar com perguntas como *"Quais foram meus últimos 5 treinos?"*

Tempo estimado: 20 a 30 minutos.

Na [versão com `uv`](garmin-claude-tutorial-uv.md), os passos 2 e 3 viram "instalar o `uv`", que é
o caminho recomendado hoje pelo README do `garmin_mcp`.

## Avisos importantes

- **Não oficial.** O `garmin_mcp` e a biblioteca
  [python-garminconnect](https://github.com/cyberjunky/python-garminconnect) não são da Garmin. Uma
  mudança no login da Garmin pode quebrar o conector a qualquer momento.
- **Privacidade.** O conector roda no seu PC, mas os dados que o Claude consulta são enviados para
  a Anthropic como parte da conversa.
- **Tokens.** O arquivo `%USERPROFILE%\.garminconnect\garmin_tokens.json` dá acesso à sua conta
  Garmin. Trate como uma senha e nunca compartilhe.
- **Revisado em 11/10/2026.** Versões, telas e comandos mudam. Se algo não bater, confira o
  [README do garmin_mcp](https://github.com/Taxuspt/garmin_mcp) e abra uma issue aqui.

## O que mudou em relação ao tutorial original

Esta é uma versão revisada de um tutorial que circulou no Instagram. As correções principais:

- **Python 3.12+** em vez de 3.11, como pede o README atual do `garmin_mcp`.
- Removida a variável `GARMIN_COOKIES_FILE`, que não existe no `garmin_mcp`.
- Caminhos atuais do Claude Desktop para editar a configuração e ver os logs.
- Nota sobre a falha de segurança CVE-2026-54447 e como o `garmin_mcp` lida com ela.
- Seções novas: como atualizar, caminho com `uv`, MFA e verificação dos tokens.

A lista completa, com as fontes, está no fim do
[tutorial](garmin-claude-tutorial.md#o-que-mudou-em-relação-ao-original).

## Créditos

- Tutorial original: [@jp.runningclub](https://www.instagram.com/jp.runningclub/). O texto e a
  estrutura deste guia derivam do trabalho dele; o conteúdo original continua sendo do autor.
- Ideia do briefing diário automático: [@jppace__](https://www.instagram.com/jppace__/).
- Servidor MCP: [Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp).
- Biblioteca Garmin: [cyberjunky/python-garminconnect](https://github.com/cyberjunky/python-garminconnect).

Se você é autor do conteúdo original e quer que algo seja ajustado ou removido, abra uma issue.

## Contribuindo

Achou um passo desatualizado ou um erro novo? Abra uma issue ou um pull request com o print do erro
(sem senhas, emails ou tokens) e a versão do Windows.

## Licença

[MIT](LICENSE), válida para as revisões e o material novo deste repositório. Garmin, Garmin Connect
e Claude são marcas de seus respectivos donos; este projeto não tem vínculo com a Garmin nem com a
Anthropic.
