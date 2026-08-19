# Atlassian Data Center — MCP

Minimální repo pro práci s **self-hosted Jira Data Center a Confluence
Data Center** z Claude Code, čistě přes MCP server
[mcp-atlassian](https://github.com/sooperset/mcp-atlassian). Žádné vlastní
CLI ani přímá API volání — jen konfigurace serveru a skill.

Změřená fakta a pasti Data Center jsou ve skillu `atlassian-dc`, který si Claude
načte sám.

## Nastavení

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
