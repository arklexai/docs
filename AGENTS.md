# Documentation project instructions

## About this project

- This is the Arkdock user documentation site built on [Mintlify](https://mintlify.com)
- Pages are MDX files with YAML frontmatter
- Configuration lives in `docs.json`
- Run `mint dev` to preview locally
- Run `mint broken-links` to check links

## Terminology

- Use "Owner" not "Admin" for the highest-privilege role
- Use "simulation" not "test run"
- Use "evaluation" not "assessment" or "review"
- Use "scenario" not "persona" (the persona is a field within a scenario)

## Style preferences

- Use active voice and second person ("you")
- Keep sentences concise — one idea per sentence
- Use sentence case for headings
- Bold for UI elements: Click **Settings**
- Code formatting for file names, commands, paths, and code references
- No em dashes or en dashes; use hyphens only where grammatically required

## Content boundaries

- Document only features visible in the current production UI
- If a feature is behind a feature flag, add a Note component explaining the availability condition
- Do not document backend implementation details unless they directly affect user behavior
