---
tags:
  - meta
  - guidelines
created: 2025-01-23
updated: 2025-01-23
status: active
---

# Knowledge Base Maintenance Guide

## Directory Structure

```
ob/
├── projects/           # Company project documentation
│   ├── mikoto/
│   ├── ishin/
│   ├── polunga/
│   └── rosetta/
├── tech/               # Technical knowledge
│   ├── languages/      # Programming languages (elixir/, ruby/)
│   ├── infra/          # Infrastructure (aws/, kubernetes/, terraform/)
│   ├── databases/      # Databases (mysql/, redis/, bigquery/)
│   ├── devops/         # CI/CD, tools
│   └── ai/             # AI/ML tools
├── operations/         # SRE & operations
├── business/           # Business planning
├── journal/            # Work logs by year
│   ├── 2023/
│   ├── 2024/
│   └── 2025/
├── _archive/           # Historical notes (2021-2022)
├── _templates/         # Obsidian templates
├── _index.md           # Main knowledge base index
└── CONTRIBUTING.md     # This file
```

## Naming Conventions

### Permanent Documents (Reference)
- Use lowercase with hyphens: `topic-name.md`
- Examples: `sre-playbook.md`, `global-database.md`

### Journal Entries (Dated)
- Format: `MM-DD-topic.md`
- Place in `journal/YYYY/` directory
- Examples: `journal/2025/04-01-tech-sharing.md`

### Index Files
- Use `README.md` for directory indexes
- Main index is `_index.md`

## Creating New Notes

### 1. Choose the Right Location

| Content Type | Location |
|--------------|----------|
| Project-specific work | `projects/{project}/` |
| General tech knowledge | `tech/{category}/` |
| Daily work logs | `journal/{year}/` |
| SRE/Operations | `operations/` |
| Business planning | `business/` |

### 2. Use Templates

Templates are in `_templates/`:
- `daily-note.md` - Work journal entries
- `tech-note.md` - Technical documentation
- `project-note.md` - Project-specific notes

### 3. Required Front Matter

Every file must have YAML front matter:

```yaml
---
tags:
  - primary-tag
  - secondary-tag
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active  # active | draft | archived
---
```

### 4. Use Wiki Links

Use Obsidian wiki-link format:
- Same directory: `[[filename]]`
- Different directory: `[[path/to/file|Display Name]]`

## Maintenance Schedule

### Weekly
- Update `status` and `updated` fields for modified files
- Review and clean up draft notes

### Monthly
- Update directory README files
- Check for broken wiki links

### Quarterly
- Review and archive old journal entries
- Consolidate similar topics
- Update `_index.md`

### Annually
- Move previous year's dated notes to archive if outdated
- Review archived content for deletion candidates

## Classification Guidelines

### When to Create Project Notes
- Implementation details specific to one project
- Project-specific decisions and rationale
- Debugging sessions for project issues

### When to Create Tech Notes
- Reusable knowledge applicable across projects
- Reference documentation
- Best practices and patterns

### When to Use Journal
- Time-sensitive work logs
- Meeting notes
- One-time event documentation

### When to Archive
- Notes older than 2 years with no recent updates
- Superseded documentation
- Historical records no longer actively used

## Tags Convention

### Category Tags
- `project`, `tech`, `operations`, `business`, `journal`

### Project Tags
- `mikoto`, `ishin`, `polunga`, `rosetta`

### Technology Tags
- `elixir`, `ruby`, `aws`, `mysql`, `redis`, `kubernetes`

### Status Tags
- `active`, `draft`, `archived`

## Tips

1. **Prefer editing over creating** - Add to existing notes before creating new ones
2. **Link liberally** - Use `[[wiki-links]]` to connect related content
3. **Keep titles concise** - 3-5 words maximum
4. **One topic per file** - Split large topics into multiple files
5. **Update the index** - Add new permanent docs to `_index.md`
