# Nama ERP Support Knowledge Base

Ask questions about Nama ERP and get answers from the official documentation and data model,
using [Claude Code](https://claude.com/claude-code).

This repo bundles two knowledge sources as git submodules:

- **`namaerp-docs/`** — the user documentation ([docs.namasoft.com](https://docs.namasoft.com))
- **`namaerp-dm/`** — the database schema ([dm.namasoft.com](https://dm.namasoft.com))

## Setup

Clone **with submodules** — the `--recurse-submodules` flag is required, otherwise the two
folders come down empty:

```bash
git clone --recurse-submodules https://github.com/ahmedqasid/namaerp-support-kb.git
cd namaerp-support-kb
```

Already cloned without it? Fix it with:

```bash
git submodule update --init --recursive
```

The download is around 250 MB and takes a few minutes the first time.

## Using it

Start Claude Code in the repo folder:

```bash
claude
```

Then just ask, in Arabic or English:

```
كيف أقفل الفترة المحاسبية؟
Which table stores customer credit limits?
Customer says the sales invoice isn't generating a journal entry — what should I check?
What parameters does the EAGenJournalEntry entity flow take?
```

Claude reads `CLAUDE.md` automatically, which tells it how to search these docs and how to
answer: it grounds every answer in an actual documentation page, gives you the menu path to
click, and includes a `docs.namasoft.com` link you can forward to the customer.

## Keeping it up to date

The submodules are pinned to a specific commit, so everyone on the team sees the same content.
To pull the latest documentation:

```bash
git pull
git submodule update --init --recursive
```

To move the pins forward to the newest docs (then commit and push so the team gets them):

```bash
git submodule update --remote --merge
git commit -am "Update docs submodules"
```

## Notes

- The submodules are **read-only mirrors**. Don't edit files inside them — documentation fixes
  go through the `namaerp-docs` and `namaerp-dm` repos directly.
- You don't need Node.js or `npm install`. Claude reads the markdown and JSON directly; there's
  no site to build.
