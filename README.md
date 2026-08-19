# Atlassian Data Center — MCP

Minimální repo pro práci s **self-hosted Jira Data Center a Confluence
Data Center** čistě přes MCP server
[mcp-atlassian](https://github.com/sooperset/mcp-atlassian). Žádné vlastní
CLI ani přímá API volání — jen konfigurace serveru a skill.

## Co repo umožňuje

- **Číst i psát Jira DC**: issues, JQL, sprinty, boardy, epics, worklogy,
  verze, komentáře — všech 98 nástrojů mcp-atlassian
  ([přehled nástrojů](https://mcp-atlassian.soomiles.com/docs/tools-reference.md)).
- **Číst i psát Confluence DC**: stránky, CQL, komentáře, přílohy, štítky.
- **Změřená fakta o DC API** (skill `atlassian-dc`): čím se Data Center liší
  od Cloudu, které cesty vrací 403/404 a proč, pasti createmeta/epic —
  změřeno proti živým instancím, ne opsáno z dokumentace.

Auth je Personal Access Token — pro Server/DC jediná podporovaná cesta
([mcp-atlassian: authentication](https://mcp-atlassian.soomiles.com/docs/authentication.md)).

## Kde to použiješ

| Nástroj | Co z repa využiješ | Jak |
|---|---|---|
| **Claude Code** | MCP server + skill | setup níže; `.mcp.json` je projektová MCP konfigurace ([docs: MCP](https://code.claude.com/docs/en/mcp)), skill se načítá z `.claude/skills/` ([docs: Skills](https://code.claude.com/docs/en/skills)) |
| **Claude Desktop** | MCP server | tentýž `uvx` příkaz v `claude_desktop_config.json` ([návod](https://modelcontextprotocol.io/quickstart/user)) — snippet níže |
| **claude.ai / Claude Desktop (Chat, Cowork)** | skill | složku `.claude/skills/atlassian-dc/` zabal do zipu a nahraj v *Customize → Skills* ([návod](https://support.claude.com/en/articles/12512180-use-skills-in-claude)); vyžaduje zapnutý code execution. Skilly fungují ve všech třech módech desktopu — Chat, Cowork i Claude Code ([přehled](https://claude.com/resources/tutorials/navigating-the-claude-desktop-app)) |
| **Cursor, VS Code a jiní MCP klienti** | MCP server | stejná `mcpServers` konfigurace ([mcp-atlassian: configuration](https://mcp-atlassian.soomiles.com/docs/configuration.md)) |

Nechceš klonovat? Plugin (skill + MCP jedním příkazem) je v sesterském repu
[atlassian-dc-plugin](https://github.com/chocholous/atlassian-dc-plugin) —
funguje v Claude Code i jako plugin marketplace pro claude.ai a Claude
Desktop/Cowork
([návod](https://support.claude.com/en/articles/13837440-use-plugins-in-claude)).

## Nastavení (Claude Code)

```bash
cp .env.example .env && chmod 600 .env    # doplň URL a tokeny
cp .mcp.example.json .mcp.json            # nahraď cestu k .env absolutní cestou
```

`.mcp.json` nemá `--enabled-tools` schválně — všech 98 nástrojů je k dispozici.
Ruční allowlist se neudržuje, protože tichá absence nástroje je horší než širší
kontext.

Personal Access Token si vytvoř ve webovém UI (v DC má povinnou expiraci):
`<JIRA_URL>/secure/ViewProfile.jspa` a
`<CONFLUENCE_URL>/users/viewmyprofile.action` → *Personal access tokens*.

Po změně `.env` nebo `.mcp.json` restartuj Claude Code.

## Nastavení (Claude Desktop)

Do `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)
resp. `%APPDATA%\Claude\claude_desktop_config.json` (Windows) — soubor otevřeš
přes *Settings → Developer → Edit Config*
([návod](https://modelcontextprotocol.io/quickstart/user)):

```json
{
  "mcpServers": {
    "atlassian-dc": {
      "command": "uvx",
      "args": ["mcp-atlassian@0.23.0", "--env-file", "/absolutni/cesta/k/.env"]
    }
  }
}
```

Desktop nespouští servery ve tvém shellu — když server nenastartuje, zadej
absolutní cestu k `uvx` (`which uvx`).

mcp-atlassian umí i HTTP transport (SSE / streamable-http) pro vzdálené
nasazení ([docs](https://mcp-atlassian.soomiles.com/docs/http-transport.md)) —
prakticky ale DC bývá v interní síti, takže lokální stdio je obvyklá cesta.

## Ověření

`claude mcp list` musí ukázat `atlassian-dc ✔ Connected` — ale pozor,
**„connected" není důkaz přístupu**: server startuje i s prázdnými tokeny
a selže až první volání. Skutečný důkaz je úspěšné první čtecí volání
(např. `jira_get_user_profile` nebo `jira_search`).

## Co se necommituje

`.env` (tokeny) a `.mcp.json` (absolutní cesta).

## Poznámka k oficiálnímu `acli`

Proti Data Center nefunguje — je Cloud-only, jede nad `/rest/api/3`, které DC
nemá (ověřeno: 0 cest pod `/api/3`, 252 pod `/api/2`). Proto MCP.

## Sesterská repa

| Repo | Obsah |
|---|---|
| [atlassian-dc-ops](https://github.com/chocholous/atlassian-dc-ops) | vše — MCP + skill + CLI skripty (referenční) |
| [atlassian-dc-plugin](https://github.com/chocholous/atlassian-dc-plugin) | totéž + marketplace s pluginem (skill + MCP) pro Claude Code, claude.ai i Claude Desktop/Cowork |
| **atlassian-dc-mcp** (tady) | jen MCP + skill, bez CLI |
| [atlassian-dc-cli](https://github.com/chocholous/atlassian-dc-cli) | jen CLI/REST + skill, bez MCP |
