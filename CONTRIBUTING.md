# Contributing

## Linting

All Markdown files are linted with
[markdownlint](https://github.com/DavidAnson/markdownlint), configured in
[`.markdownlint.jsonc`](.markdownlint.jsonc). The
[Lint workflow](.github/workflows/lint.yml) runs it on every push to `main` and
every pull request that is not a draft, and fails if it finds any error. Run it
locally before pushing:

```bash
npx markdownlint-cli2 "**/*.md"
```

Wrap prose and list items at 80 characters. Headings, tables and code blocks
are exempt; SQL examples follow the guide's own 120-character limit.

## Rules

- Each rule is a `##` heading in its group's file under [`guide/`](guide/),
  starting with its code (for example `## LAY-03 ...`), followed by its
  explanation, a `### Example` and, when known, a `### Source breakdown`.
- A new rule takes the next free number in its group. Codes are never reused:
  when a rule is removed, its code is retired.
- Keep the rule checklist in [`guide/SKILL.md`](guide/SKILL.md) in sync with
  the guide, citing the codes each bullet covers.
