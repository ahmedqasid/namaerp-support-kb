# Nama ERP — AI Support Knowledge Base

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

**Find which entities reference a given entity** (the item master is `InvItem`, not `Item`):

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

## Keeping content fresh

The submodules are pinned to a commit. If an answer looks stale, or the user says "this was
added in a recent release", update them:

```bash
git submodule update --remote --merge
```
