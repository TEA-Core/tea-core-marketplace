# tea-core-marketplace

Custom Claude Code plugin marketplace for David Vennei's TEA-Core skill corpus.

Install on any Claude Code host:

```bash
/plugin marketplace add TEA-Core/tea-core-marketplace
/plugin install lovable-prompt-library@tea-core
```

## What's in this folder

This is the bootstrap area that lives in the Obsidian vault. It produces two things:

1. The **marketplace repo** at `TEA-Core/tea-core-marketplace` — public manifest listing every TEA-Core skill.
2. One **`.claude-plugin/plugin.json`** inside each of the 9 TEA-Core skill repos so they're installable plugins.

| File | Purpose |
|---|---|
| `marketplace.json` | The hand-built v0.1.0 manifest. Gets copied into the marketplace repo at `.claude-plugin/marketplace.json`. |
| `plugin-templates/<skill>/.claude-plugin/plugin.json` | Per-repo plugin manifest. Gets copied into the matching TEA-Core skill repo. |
| `generate_marketplace.py` | Regenerates `marketplace.json` from the Obsidian vault's `skills/*.md` frontmatter — use this when you add/change/remove a skill instead of hand-editing. |
| `ship.sh` | One-shot script that creates the marketplace repo (if needed), copies in `marketplace.json`, and pushes `.claude-plugin/plugin.json` to each of the 9 skill repos. Run it on your Mac with `gh` authed — the Cowork sandbox lacks credentials to push. |

## Architecture

The system has three sources of truth, each with one job:

| What | Source of truth | Consumes |
|---|---|---|
| Skill *content* | TEA-Core repos (`TEA-Core/<skill>`) | — |
| Registry *index* | Obsidian vault (`mooncatcher/skills/*.md`) | Used by `skills.base` view, also by `generate_marketplace.py` |
| Marketplace *manifest* | Generated artifact, not a third source | Committed to `TEA-Core/tea-core-marketplace/.claude-plugin/marketplace.json` |

The marketplace manifest is derived from the vault — regenerate it with `generate_marketplace.py` after any change to a `skills/*.md` entry. This keeps the marketplace from drifting out of the registry.

## Shipping it (first time)

Run on your Mac (not the Cowork sandbox):

```bash
cd ~/vaults/mooncatcher/tea-core-marketplace   # or wherever the vault lives
gh auth status                                  # confirm auth to TEA-Core org
./ship.sh --dry-run                             # preview the commits + pushes
./ship.sh                                       # execute
```

The script is idempotent — safe to re-run. If a plugin.json is already in place and unchanged, the step is a no-op.

## Regenerating `marketplace.json`

After you add, rename, or retire a skill in the vault:

```bash
cd ~/vaults/mooncatcher/tea-core-marketplace
python3 generate_marketplace.py \
  --vault-skills-dir ../skills \
  --out marketplace.json
./ship.sh                                       # pushes the regenerated manifest
```

### Opting a skill out of the marketplace

The generator filters on `status: stable` + `claude_code_compatible: true` + `doc_type: skill_entry` + `repo` prefix `https://github.com/TEA-Core/`. To keep an entry in the vault registry but exclude it from the marketplace, flip one of those fields (usually `status`) in the vault frontmatter until it's ready.

## Known caveats

- **`skills: ["./"]` vs a `skills/<name>/` subfolder.** Each plugin.json currently declares `"skills": ["./"]`, relying on Claude Code's path behavior rule that a directory path containing `SKILL.md` resolves to a single skill (name taken from SKILL.md frontmatter). If a future CC version rejects this shape, the fix is to move each repo's contents into `skills/<skill-name>/` and update the plugin.json to `"skills": ["skills/<skill-name>"]`.
- **Skill card drift.** `tea-core-skill-registry`'s "Quick reference — current registry" table listed 6 skills and marked `lovable-tech-stack-workflow` as TODO. That table is stale — the vault (canonical per invariant 2) says all 9 are migrated and stable. Consider refreshing the table on the next pass through the registry skill.
- **Private repos need auth.** Because TEA-Core repos are private, every client that runs `/plugin marketplace add` needs GitHub auth with read access to the org — either via `gh auth`, an SSH key, or `GITHUB_TOKEN`/`GH_TOKEN` env var.

## Current skill roster

| Skill | TEA-Core repo |
|---|---|
| checkout-handoff | https://github.com/TEA-Core/checkout-handoff |
| lovable-deployment-ops | https://github.com/TEA-Core/lovable-deployment-ops |
| lovable-notion-automation-workflow | https://github.com/TEA-Core/lovable-notion-automation-workflow |
| lovable-project-generator | https://github.com/TEA-Core/lovable-project-generator |
| lovable-prompt-library | https://github.com/TEA-Core/lovable-prompt-library |
| lovable-secrets-extractor | https://github.com/TEA-Core/lovable-secrets-extractor |
| lovable-tech-stack-workflow | https://github.com/TEA-Core/lovable-tech-stack-workflow |
| lovable-verification-workflow | https://github.com/TEA-Core/lovable-verification-workflow |
| tea-core-skill-registry | https://github.com/TEA-Core/tea-core-skill-registry |
