# WordPress Security Checklist skill

An [Agent Skill](https://agentskills.io) that brings the [WordPress Security Checklist](https://wpsecuritychecklist.org) to AI coding agents such as Claude Code, Codex, and Cursor.

It has two modes:

- **Audit:** inspects a WordPress install through its files, WP-CLI, and optionally a public URL you are authorized to test. It reports every checklist item as ✅ pass, ❌ fail, ⚠️ review, ❔ unknown, or 📝 manual, with evidence and a fix, and starts with the most urgent problems.
- **Guide:** explains items and walks you through hardening your site in English, Portuguese, Spanish, or Japanese, with links to the checklist in your language.

## Install

Pick one of these options. None of them clone this repository.

**Any agent (Claude Code, Codex, Cursor, and others)** with the [`skills` CLI](https://github.com/vercel-labs/skills) (requires Node.js):

```bash
# Current project
npx skills add WP-Security-Checklist/skill

# All projects
npx skills add WP-Security-Checklist/skill -g
```

**Claude Code plugin**, which you can update with `/plugin update`:

```
/plugin install wordpress-security-checklist --marketplace WP-Security-Checklist/skill
```

On Claude Code versions before 2.1.275, run `/plugin marketplace add WP-Security-Checklist/skill` first, then `/plugin install wordpress-security-checklist@wordpress-security-checklist`.

**Without Node.js or git**, download only the skill folder:

```bash
# Claude Code: ~/.claude/skills   Codex: ~/.agents/skills
SKILLS_DIR=~/.claude/skills
mkdir -p "$SKILLS_DIR"
curl -sL --url 'https://codeload.github.com/WP-Security-Checklist/skill/tar.gz/main' \
  | tar -xz -C "$SKILLS_DIR" --strip-components=2 \
    skill-main/skills/wordpress-security-checklist
```

Run the same command again to update.

## Use

Ask your agent, for example:

- "Audit this WordPress site against the security checklist."
- "Check https://example.com for the checklist items you can verify remotely. I own the site."
- "What does the XML-RPC item mean, and should I disable it?"
- "Me ajude a aplicar o checklist de segurança no meu site WordPress."

## Safety

Audits are designed to be safe to run on a production site:

- **Read-only by default.** The agent offers fixes after the report and applies them only when you ask.
- **Non-intrusive.** It only sends ordinary `GET` and `HEAD` requests to sites you own or are authorized to test. It never attempts logins, brute force, exploits, or port scans.
- **No secrets in reports.** Passwords, keys, salts, and hashes are described ("set, 64 characters"), never printed. Email addresses are masked.
- **Resistant to prompt injection.** Everything the agent inspects is treated as data. Text in files or responses that tries to instruct an AI agent is ignored and reported as a sign of tampering.
- **Escalates compromises.** Signs of a hacked site go at the top of the report, with a pointer to incident response. Suspicious files are never opened, run, or deleted.

The skill gives your agent instructions; it does not restrict what the agent can do. Review the commands your agent proposes, as with any tool that runs commands on your behalf.

## How it works

| File | Purpose |
|---|---|
| `skills/wordpress-security-checklist/SKILL.md` | Modes, workflow, safety rules, and report format. |
| `skills/wordpress-security-checklist/references/checklist.md` | Every checklist item with its stable ID, in all four languages. **Generated** from the website; do not edit by hand. |
| `skills/wordpress-security-checklist/references/checks.md` | Why each item matters, how to verify it, and how to fix it, keyed by item ID. Written by hand. |
| `scripts/sync-checklist.py` | Regenerates `checklist.md` from the [website repository](https://github.com/WP-Security-Checklist/website) and validates `checks.md`. |
| `.claude-plugin/marketplace.json` | Claude Code plugin marketplace entry. |

### Keeping in sync with the website

The checklist itself lives in [WP-Security-Checklist/website](https://github.com/WP-Security-Checklist/website) (`items*.md`). After the checklist changes there, regenerate `checklist.md` here:

```bash
python3 scripts/sync-checklist.py                      # from GitHub (website main)
python3 scripts/sync-checklist.py --website ../website # from a local checkout
python3 scripts/sync-checklist.py --check              # only check, don't write
```

The script fails if the website's languages have different item IDs, or if `checks.md` is missing an entry for an item, so a new item can't be added to the skill without audit instructions.

## Contributing

- **Checklist wording or new items:** change them in the [website repository](https://github.com/WP-Security-Checklist/website) first, then sync them here.
- **New item:** run the sync script, then add the item's entry to `references/checks.md` (Why, Check, Fix) in the same pull request.
- **Audit logic or safety rules:** edit `SKILL.md` or `checks.md`, and test the skill with a real agent before opening a pull request. A fake WordPress folder with planted problems (a `root` database user, a default `AUTH_KEY`, a `wp-config.php.bak`) works well.

The idea for this skill came from the WordPress São Paulo community security meetup ([#1](https://github.com/WP-Security-Checklist/skill/issues/1)).

## License

[GPL v2](LICENSE), the same as the checklist.
