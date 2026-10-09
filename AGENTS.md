# Documentation project instructions

## About this project

- This is the Arklex Platform user documentation site built on [Mintlify](https://mintlify.com)
- Pages are MDX files with YAML frontmatter
- Configuration lives in `docs.json`
- Run `mint dev` to preview locally
- Run `mint broken-links` to check links
- Limits that appear on more than one page live in `snippets/limits.mdx`. Import the variable instead of typing the number, so every page stays in sync

## Terminology

- The product name is "Arklex Platform". Use the full name in titles, descriptions, and the first mention on a page, then "Arklex"
- Use "Owner" not "Admin" for the highest-privilege role
- Use "simulation" not "test run"
- Use "evaluation" not "assessment" or "review"
- Use "scenario" not "persona" (the persona is a field within a scenario)

## Style preferences

- Use active voice and second person ("you")
- Keep sentences concise: one idea per sentence
- Use sentence case for headings
- Bold for UI elements: Click **Settings**
- Code formatting for file names, commands, paths, and code references
- No em dashes or en dashes; use hyphens only where grammatically required

## Content boundaries

- Document only features visible in the current production UI
- If a feature is behind a feature flag, add a Note component explaining the availability condition
- Do not document backend implementation details unless they directly affect user behavior
