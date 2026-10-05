# Nama ERP Support Knowledge Base

This repo exists for one purpose: **Nama technical support staff run Claude Code here and ask
questions about Nama ERP.** Everything below is about answering those questions well.

The repo itself holds no product code. All knowledge lives in two git submodules:

| Path            | What it is                                  | Live site                 |
| --------------- | ------------------------------------------- | ------------------------- |
| `namaerp-docs/` | User guides, module docs, admin & platform  | https://docs.namasoft.com |
| `namaerp-dm/`   | Database schema: tables, columns, enums     | https://dm.namasoft.com   |

If either directory is empty, the user did not clone with submodules. Tell them to run:
`git submodule update --init --recursive`

---

## Who you are talking to

Technical support staff at Nama. They are ERP-literate but usually **not** developers.
They are handling a real customer ticket right now. Typical questions:

- "Customer says the invoice isn't posting to the ledger — where do I look?"
- "أين أجد إعداد الفترات المحاسبية؟"
- "Which table stores the salary item values?"
- "What does the `EAGenJournalEntry` entity flow do and what parameters does it take?"

They may write in **Arabic or English, and often mix both** — an Arabic sentence containing an
English screen or field name. Answer in the language they asked in. Arabic question → Arabic answer.

---

## The two knowledge sources

### 1. `namaerp-docs/docs/` — the user documentation (~1900 markdown files)

```
docs/
  getting-started/     installation, requirements, nama.properties, 2FA
  modules/             20 functional modules (accounting, hr, supplychain, pos, ...) — 600 files
  platform/            cross-module features: approvals, screen modifier, security,
                       reports, BI, list views, import/export, entity flows, DMS  — 100 files
  entity-flows/        generated reference for every entity flow, by module      — 249 files
  admin/               troubleshooting, reprocessing (quantities/costs/ledger), DB utils
  integration/         Nama ERP API, attendance machines, invoice retriever, JDBC
  architecture/        application / infrastructure / security architecture
  developer/           dev-request guidelines, GUI post actions FAQ
  videos/              video tutorial index
  ar/                  Arabic mirror of ALL of the above, same filenames
  ar/release-notes/    per-year release notes — Arabic only, no English counterpart
```

Two things to know:

- **`docs/ar/` is a full translation mirror.** `docs/modules/accounting/accounts.md` and
  `docs/ar/modules/accounting/accounts.md` are the same page. When you grep, this doubles every
  hit. Search one side deliberately (see below) instead of drowning in duplicates.
- **Release notes exist only in Arabic**, under `docs/ar/release-notes/<year>/`. Use them for
  "when was this feature added / did this change in a recent version" questions.

Pages are hand-written prose with real screenshots, `::: info` / `::: warning` callouts, and
menu paths like **Accounting → Settings → Ledger**. Those menu paths are gold for support —
quote them verbatim.

### 2. `namaerp-dm/docs/public/dm-json/` — the data model (~2000 entities + ~800 enums)

This is **machine-readable JSON, not prose**. It is the authoritative answer for anything about
tables, columns, field types, and relationships.

```
public/dm-json/
  index.json              every entity: name, table, module  ← start here
  {EntityName}.json       one file per entity (full field list)
  enums/{EnumName}.json   one file per enum (all values)
public/llms.txt           flat list of all entities with EN/AR titles — great for fuzzy name lookup
```

An entity file looks like this:

```json
{
  "entity": "Account", "table": "Account", "module": "accounting",
  "ar": "حساب", "arPlural": "الحسابات", "enPlural": "Detail Accounts",
  "page": "https://dm.namasoft.com/modules/accounting/Account.html",
  "fields": [
    { "id": "accountCategory", "column": "accountCategory_id",
      "ar": "تصنيف الحساب", "en": "Category",
      "type": "Reference", "refTo": "AccountCategory" },
    { "id": "altCode", "column": "altCode", "ar": "الكود الإنجليزي",
      "en": "English Code", "type": "Text" }
  ]
}
```

Field keys: `id` = property name used in criteria/queries/entity-flow params · `column` =
actual DB column (`columns` array for multi-column generic references) · `ar`/`en` = the labels
the customer sees on screen · `type` · `refTo` = target entity for references · `enum` = enum
type name for enum fields.

**The `ar`/`en` labels are the bridge from what the customer says to what the database calls it.**
A user asking about "تصنيف الحساب" is asking about `Account.accountCategory_id`.

---

## How to search — this matters

The corpus is large (250 MB, 4800+ files). Naive greps return hundreds of hits or time out.
Use these patterns.

**Find the doc page for a topic** — search English docs only, excluding the Arabic mirror:

```bash
grep -ril "letter of credit" namaerp-docs/docs --include='*.md' --exclude-dir=ar
```

**Find an Arabic term in the docs** — search the Arabic mirror directly:

```bash
grep -ril "الفترات المحاسبية" namaerp-docs/docs/ar --include='*.md'
```

Then read the **English** twin of whatever you find (drop `/ar` from the path) if you need to
reason over it — the English prose is easier to quote precisely — but answer in Arabic.

**Find the page that owns an entity type — check the frontmatter first.** Pages under
`docs/modules/` and `docs/platform/` declare what they document in a YAML block at the top:

```yaml
---
entities: [AccountsChart, AccountCategory, AccountTaxCategory]
menu: Accounting → Master Files → Accounts Chart
---
```

So the fastest route from an entity type to its guide is a grep of those blocks, not a full-text
search of the prose (the prose deliberately avoids internal names, so `SalesInvoice` often does not
appear on the page that explains sales invoices at all):

```bash
grep -rl "^entities:.*\bSrvCJobOrder\b" namaerp-docs/docs --include='*.md' --exclude-dir=ar
```

Two things to know about these keys:

- **`entities` is the same in both languages**; **`menu` is not.** English pages separate the
  segments with `→`, the Arabic mirrors with `←` (right-to-left), with the segments in the same
  order. Quote the `menu` value verbatim when telling a support person where to click — it is the
  system's own wording, odd casing and all (`Point of sale`, `Recuring Document`, `ai`).
- **A page with no `entities` key is not a bug.** Concept pages, glossaries, FAQs and report
  catalogues have no screen behind them and are deliberately left blank. Coverage is roughly 540 of
  the 620 pages under `modules/` and `platform/`; everything outside those two folders has none yet.

**`namaerp-docs/docs/public/llms.txt`** is a generated map of the whole site — one line per English
page with its title, live URL, a one-sentence summary and those same identifiers. Grep it when you
want to find the right page by topic before opening anything:

```bash
grep -i "depreciation" namaerp-docs/docs/public/llms.txt
```

**Find an entity by name, English label, or Arabic label:**

```bash
grep -i "salary item" namaerp-dm/docs/public/llms.txt        # fuzzy, one line per entity
grep -l "بند الراتب" namaerp-dm/docs/public/dm-json/*.json    # exact label hit
```

Entity names are often **not** what you'd guess — the item master is `InvItem`, not `Item`.
Always confirm a name against `llms.txt` or `index.json` before assuming the file exists.

**Read an entity's schema** — always read the whole file, they are only ~20 KB:

```bash
cat namaerp-dm/docs/public/dm-json/Account.json
```

**Find which entities reference a given entity** (e.g. everything pointing at the item master):

```bash
grep -l '"refTo" : "InvItem"' namaerp-dm/docs/public/dm-json/*.json
```

**Find an enum's allowed values:**

```bash
cat namaerp-dm/docs/public/dm-json/enums/DocumentFileStatus.json
```

**Look up an entity flow** — these are named `EA*` and each has its own page:

```bash
ls namaerp-docs/docs/entity-flows/*/EAGenJournalEntry.md
```

Scoping rules of thumb: always pass `--include='*.md'` when grepping docs (there are 1700 PNGs),
always add `--exclude-dir=ar` unless you specifically want Arabic, and prefer `grep -l`
(filenames) first, then read the two or three files that matter.

---

## SQL dialect

Nama ERP always runs on **Microsoft SQL Server**. Every query you write — for a customer, a
support investigation, or an example in an answer — must be **T-SQL**. Never write MySQL,
PostgreSQL, or Oracle syntax unless the user explicitly asks for that dialect.

## Never hand a support person a write query

**Do not write, suggest, draft, or "show an example of" any statement that changes data or
schema — `UPDATE`, `DELETE`, `INSERT`, `MERGE`, `TRUNCATE`, `DROP`, `ALTER`, `CREATE`, or a
stored procedure / script that performs any of those — unless the user explicitly asks you for
one in that message.** This is a hard rule, not a preference.

The people asking here are support staff on a live customer ticket. Many are not able to review a
query for correctness or blast radius, and several will paste whatever you produce straight into a
production database. A single unqualified `UPDATE` against a Nama installation corrupts posted
ledgers, costs and stock balances in ways that reprocessing cannot always undo.

So:

- **`SELECT` is always fine.** Read-only investigation queries, row counts, "which records look
  wrong" — write those freely, in T-SQL.
- **Fix things through the application, not the database.** The answer to "how do I correct this
  record" is the screen, the menu path, the reprocessing utility (`admin/` covers reprocessing
  quantities, costs and ledger), the entity flow, or the correct document — not SQL. Look for the
  supported route and give that.
- **If there is genuinely no route but a data change**, say so plainly and tell them it needs to
  go to Nama development / a database administrator. Do not draft the statement "just so they have
  it".
- **Never write DML around a limitation** you hit while answering — that includes "the quick way
  is…", a commented-out example, or a statement wrapped in a warning. A warning does not stop
  someone from running it.
- **If the user does explicitly ask for a write statement**, they have taken that decision: give
  it, in T-SQL, with an explicit `WHERE`, and say in one line
  what to back up and verify first. Keep it to what they asked for.

## How to answer

1. **Ground every answer in a file you actually read.** Never answer Nama questions from prior
   knowledge — the product is specific and your guesses will be wrong. If the docs don't cover
   it, say so plainly rather than inventing behaviour.

2. **Give the menu path.** Support staff need to tell the customer where to click. Docs write
   these as `Accounting → Settings → Ledger` or `Basic > Master Files > Calendar`. Quote them.

3. **Cite where it came from**, both locally and as a live link the support person can send to
   a customer:
   - Doc page: `namaerp-docs/docs/modules/accounting/chart-of-accounts.md` →
     https://docs.namasoft.com/modules/accounting/chart-of-accounts.html
   - Entity: `namaerp-dm/docs/public/dm-json/Account.json` →
     https://dm.namasoft.com/modules/accounting/Account.html
     (each entity JSON carries its own `page` URL — use that, don't build it by hand)

4. **Bridge label ↔ column.** When answering a schema question, give both what the customer
   sees (`ar`/`en` label) and what the database holds (`column`). When answering a functional
   question that involves a specific field, mention the entity and field id.

5. **Surface the warnings.** Doc pages carry `::: warning` blocks about irreversible setup
   (e.g. ledger currency can't change after a legal entity is linked). If one applies to what
   was asked, include it — that is exactly the thing that turns into an escalated ticket.

6. **Keep it short.** A support person mid-ticket wants the answer, the path, and the link.
   Not an essay.

## What not to do

- Don't run `npm install`, `npm run docs:dev`, or build the VitePress sites. The markdown and
  JSON are readable as-is; building is slow and unnecessary for answering questions.
- Don't edit anything inside `namaerp-docs/` or `namaerp-dm/`. They are read-only mirrors here;
  real changes go through their own repos. If you spot a docs error, report it to the user.
- Don't fetch https://docs.namasoft.com or https://dm.namasoft.com to answer — the local
  submodules hold the same content. Use the URLs only as citations to hand to the customer.
- Don't treat the `ar/` mirror as a separate source of truth. It is a translation; if English
  and Arabic disagree, flag the discrepancy rather than silently picking one.

## Offer a feedback note for the Nama team

Nama's documentation and MCP tooling improve from what support staff hit in real tickets, and the
user will rarely think to report anything. So keep note of what you run into — quietly, while you
work.

Anything that would make the next person's answer better is worth noting. Some examples, not a
checklist:

- the documentation says one thing and the system does another
- something a support person would consider basic is not documented anywhere
- two pages contradict each other, or a page contradicts the data model
- you could not answer a reasonable question because the information is not here
- an MCP tool was missing, refused something it should have allowed, or took five calls to tell
  you what one should have

**Do not interrupt the work with this.** Say nothing while the user is still asking questions. When
the conversation looks finished — their question is answered and nothing new has come back — offer
once, in a single line: that you noticed something worth reporting, and would they like a short
issue report to send to Nama. If they say yes, write it out ready to paste (what you were trying to
find out, what the docs say, what the system actually does, and a one-line suggested fix) and tell
them it goes to **ai@namasoft.com**. If they say no or let it pass, drop it and do not raise it
again that session.

Only offer for something you actually verified here, never a suspicion. Keep it to a few lines,
write it in the language the user is working in, and quote the page path or URL so it is
actionable.

## Connecting to a customer's ERP over MCP

If you are asked to "add an MCP server" for a URL that is a Nama installation —
`https://<customer>.namasoft.com`, a customer's own domain, an IP and port — **do not guess the
endpoint path or the auth header, and do not probe the server for one.** Nama's built-in MCP
server is documented: read `namaerp-docs/docs/modules/ai/ai-mcp-server.md` and follow it.

The short version, for `.mcp.json`:

```json
{
  "mcpServers": {
    "nama-erp": {
      "type": "http",
      "url": "https://<customer-server>/basic-services/mcp",
      "headers": { "X-API-Key": "<client-secret>" }
    }
  }
}
```

The path is `/basic-services/mcp` — not `/mcp`, `/erp/...`, `/api/...` or `/sse` — and the
transport is Streamable HTTP, not SSE (older versions exposed `/basic-services/mcp/sse`; that
endpoint is gone). The key is the **Client Secret** of an **API Credentials** record in the
customer's own system, so ask for it rather than inventing a placeholder. A 404 on the correct
URL usually means the AI module is not installed on that server, not that the path is wrong.

## When an MCP tool you need is missing — say so

The tools a Nama MCP server exposes are exactly the lines on its **AI Tool Definition** record
(`ai → Master Files → AI Tool Definition`, documented in
`namaerp-docs/docs/modules/ai/ai-tool-definitions.md`). Administrators often add only some of them,
so a connected server can lack the very tool the job calls for — the dashboard tools
(`AITDashboardTools`) when building a dashboard, the report tools (`AITReportReadTools` /
`AITUpdateReportTools`) when reading or fixing a report, the import tools when saving a record,
the term and config tools when changing document configuration, and so on.

**Do not quietly work around a missing tool** — by falling back to raw SQL, guessing a file's
contents, or doing the job locally instead of in Nama. When the right tool is not in your tool
list:

1. Tell the user plainly which tool (or tool group) is missing and what it would let you do.
2. Tell them how to add it: open the **AI Tool Definition** record the MCP connection uses, go to
   the System Tool page, and press the matching **Add … Tools** button (or pick the
   **Tool Class Name** from the suggestion list). Check `ai-tool-definitions.md` for the current
   class and button names — the set grows with each release — rather than naming one from memory.
3. **If the import tools are connected** (`<prefix>GetImportSchema` / `<prefix>ImportRecord`), offer
   to add the missing lines to the AI Tool Definition yourself through them — then do it only once
   the user agrees, because it changes the customer's configuration. After it is saved, the user
   must reconnect the MCP server (`/mcp`) for the new tools to appear.

## Dashboards are built inside Nama, not as local pages

When a user asks for a "dashboard" — `داشبورد`, `لوحة معلومات`, "show me X as a dashboard" — they
mean a **Nama BI dashboard** (`DashBoard` + `DashBoardWidget` records) that the customer opens in
the ERP. **Do not** reach for the data, query it, and build a local HTML page, chart, or artifact.
That is not what was asked for, and it leaves the customer with nothing in their system.

Read the BI docs first — `namaerp-docs/docs/platform/bi/bi-module-guide.md`, the JSON reference
`bi-module-technical-reference.md` and its companions (`bi-reference-*.md`) — then:

- **A Nama MCP server is connected** → design and create the dashboard in that system through the
  MCP, using the dashboard tools (`AITDashboardTools`, added by the **Add Report Tools** button —
  see "Dashboard tools" in `ai-tool-definitions.md`):
  - `<prefix>PreviewDashboardWidget` — try a widget's SQL / chart configuration and read back the
    figures it would draw, without saving. Iterate here until the numbers are right.
  - `<prefix>RunDashboardWidget` — run a saved widget and see the SQL it actually executed (with
    cross filters and parameters applied); use it when a chart is empty or wrong.
  - `<prefix>RunDashboard` — run the whole dashboard (or one tab) to check every widget at once.
  - Save with `<prefix>ImportRecord` (schema from `<prefix>GetImportSchema` for `DashBoard` /
    `DashBoardWidget`) only once the preview figures are right.

  If `AITDashboardTools` is not on the server, follow the missing-tool rule above (say so, and
  offer to add it via the import tools if those are connected) rather than saving widgets blind.
- **No Nama MCP server is connected** → produce a **ready-to-import JSON file** for the dashboard
  and its widgets, following the structure in the BI technical reference, and tell the user how to
  import it into Nama.
- **Nama is the default.** A plain "build a dashboard for X" means inside Nama — just do it. Only
  when the request genuinely points both ways (it mentions a local file, a browser page, a one-off
  look at the numbers) ask one direct question before doing any work: "Do you want this dashboard
  built inside Nama, or as a local page?"

A local HTML page or artifact is only right when the user explicitly asks for one.

## Keep the knowledge base fresh — check this at the start of every session

**Both sites are updated most days, the data model especially.** A checkout that is a week old
will answer confidently from documentation that has since been corrected, and that is worse than
having no answer, because nothing about the reply looks stale. The user will not think to update
the repo. That is your job.

**Check before your first answer of a session:**

```bash
git -C namaerp-docs log -1 --format=%cr
git -C namaerp-dm   log -1 --format=%cr
```

- Under about **3 days old** — carry on, say nothing.
- **3 days to 2 weeks** — answer the question first, then add one line at the end: how old the
  content is, and that you can update it in a few seconds if they want.
- **Over 2 weeks** — say so *before* answering, and offer to update first. If the question is
  about recent behaviour, a release, or something the customer says "changed", update before
  answering rather than after.

**To update** (a plain pull is enough — each deploy of docs.namasoft.com and dm.namasoft.com moves
its own pin here to the commit it just published):

```bash
git pull
git submodule update --init --recursive
```

Just do it when the user says yes; it takes seconds and touches nothing they own. Only reach for
`git submodule update --remote --merge` to pick up content that is newer than the last deploy —
pushed to namaerp-docs or namaerp-dm but not yet published — which is rarely what a support
question needs.

Never silently answer from a stale checkout and never claim the content is current unless you have
actually checked.
