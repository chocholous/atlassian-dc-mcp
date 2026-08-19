---
name: atlassian-dc
description: Práce s Atlassian Jira Data Center a Confluence Data Center (self-hosted). Použij při práci s issues, epics, JQL, sprinty, boardy, stránky, CQL, komentáře.
---

# Atlassian Data Center

Všechno jde přes MCP `atlassian-dc`. URL instancí a tokeny jsou v `.env` (gitignored)
— **tokeny nikdy nevypisuj** do transkriptu, commitů ani souborů.

## Než začneš

Problem je v konfiguraci, dokud neplatí následující:

1. **`claude mcp list | grep atlassian-dc` ukazuje `✔ Connected`.**
2. **První čtecí volání projde.** „Connected" není důkaz — server startuje
   i s prázdnými tokeny a selže až první volání. Ověř např. přes
   `jira_get_user_profile` nebo ručně (token na stdin, nikdy přes `-H`,
   a nikdy `-v`/`-sv`/`--trace*`/`--libcurl`):

```bash
set -a; . .env; set +a
printf 'header = "Authorization: Bearer %s"\n' "$JIRA_PERSONAL_TOKEN" \
  | curl -sS -K - --fail-with-body -H "Accept: application/json" \
    "$JIRA_URL/rest/api/2/myself" | jq .
```

## Fakta

Změřeno proti živým DC instancím (Jira 10.3, Confluence 9.2). Kde je tvrzení
neověřené, je to napsané.

| | |
|---|---|
| API | Jira `/rest/api/2` a `/rest/agile/1.0`, Confluence `/rest/api` |
| `/rest/api/3` | na DC **neexistuje** (302 na login); spec má 0 cest pod `/api/3`, 252 pod `/api/2` |
| Confluence vs Cloud | `/rest/api`, **ne** `/wiki/rest/api` |
| Confluence `serverInfo` | **neexistuje** (404); verze jen z `/rest/applinks/1.0/manifest`, tedy pod jiným basem |
| `createmeta` v Jiře 10 | `/issue/createmeta/{key}/issuetypes/{typeId}`; starý `?projectKeys=` dává 404 |
| Epic | vyžaduje `customfield_10002` (Epic Name), jinak 400 |
| Ostatní typy | povinné jen `summary`, `issuetype`, `reporter`, `project`; Sub-task navíc `parent` |
| Globální seznamy lžou | `/priority` vrátilo 27 hodnot, ale konkrétní projekt+typ jen 5 → ber `allowedValues` z `jira_get_create_fields` |
| `jira_search` | vrací `total` přímo v odpovědi |
| `confluence_search` | strop 50, **nestránkuje**, počet nevrací → při 50 výsledcích říkej „nejméně 50" |
| `jira_get_comments` | **neexistuje**; komentáře přes `jira_get_issue` + `comment_limit`. Neověřeno na datech — absenci klíče `comments` neber za „issue je nemá" |
| `expand="changelog"` | v odpovědi je klíč **`changelogs`** (množné číslo) |
| Běžný uživatelský token | **65 čtecích cest vrací 403**: schémata, skupiny, `application-properties`, `monitoring`, `cluster`, `audit`, `backup-restore`, `webhooks`, `screens`, `settings`, `upgrade`. 403 není chyba konfigurace, je to odpověď |
| 404 rozlišuj podle **těla** | `{"message":"HTTP 404 Not Found","sub-code":-1}` = endpoint chybí; `errorMessages` nebo konkrétní věta = endpoint je, chybí objekt. Kontrast 403 vs 404 spolehlivý **není** |
| Confluence + špatný typ id | vrací **500**, ne 400 — chyba je na vstupu |
| Těžké endpointy | `/api/2/worklog/deleted` a `/updated` (chybí `since`), `/rest/api/label/recent` (chybí `limit`) — bez parametrů timeout |
| MCP „connected" | **není důkaz** — server startuje i s prázdnými tokeny a selže až první volání |
| MCP „Pending approval" | server je v .mcp.json, ale ještě nebyl schválený → **žádný `mcp__atlassian-dc__*` nástroj v session není**. `ToolSearch` na jejich jména vrací prázdno. Schválit interaktivně přes `claude` |
| PAT | povinná expirace → náhlé 401 napříč vším je vypršelý token. Naopak timeout na jednom endpointu (jiné se stejným PAT prošly) = transient, ne token |
| Destruktivní nástroje | `jira_delete_issue`, `confluence_delete_page`, `confluence_delete_attachment`, `jira_remove_issue_link`, `jira_remove_watcher` |
| `acli` (oficiální) | Cloud-only, proti DC nefunguje |

## Zápisy

Před **každým** zápisem vypiš přesné volání se všemi parametry (u Confluence i `id`,
`title`, `space`, `version`) a čekej na potvrzení v chatu; souhlas platí na jedno
volání, ne na dávku. Po zápisu objekt přečti zpět a cituj změněnou hodnotu —
prázdná odpověď ani `rc=0` nejsou důkaz.
