# Changelog

All notable changes to this project are listed here. Versions follow [Semantic Versioning](https://semver.org/).

## [1.1.0] - 2026-10-31

### Changed
- Cost estimates take model prices from the vendor's official pricing page only (Anthropic, OpenAI and Google pages listed), fetched directly instead of found through a web search.
- Looked-up prices are reused for `pricing.max_age_days` days (default 7) from the new `pricing.cache` setting, so the page is fetched about once a week instead of for every note.

## [1.0.0] - 2026-09-28

### Added
- Skill `save-conversation-to-obsidian` for Claude, ChatGPT, Gemini and coding agents.
- First-run setup interview; configuration stored in memory, in the vault or in the skill.
- Five write routes: local file access, linked computer, cloud drive via Make, download, copyable Markdown.
- Project routing based on the Claude Project, ChatGPT Project, Gemini Gem or repo a chat belongs to.
- Note format with frontmatter, assistant tags, commit table, Mermaid diagrams and usage estimate.
- Make scenario for OneDrive (importable blueprint) and guide for Google Drive.
- Claude Code plugin and marketplace manifests; release workflow that builds the skill ZIP.
