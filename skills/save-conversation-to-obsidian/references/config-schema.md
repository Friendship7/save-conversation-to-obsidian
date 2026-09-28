# Configuration reference

The configuration is a small YAML block named `obsidian-conversation-config`. It holds only names, paths and switches, never passwords, tokens or API keys.

It can live in three places; the skill reads them in this order and uses the first complete one:

1. The **Configuration** block at the end of `SKILL.md`.
2. The assistant's **persistent memory**, custom instructions or project knowledge, under the name `obsidian-conversation-config`.
3. A note `save-conversation-config.md` in the **vault root**, with the YAML in a fenced `yaml` block.

## Fields

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `vault_name` | text | | Name of the Obsidian vault, used in messages and download hints. |
| `timezone` | text | from context | IANA timezone for the note's date and time, e.g. `Europe/Zurich`. |
| `vault_roots.windows` | path | | Vault folder on Windows, e.g. `C:\Users\me\Obsidian\MyVault`. |
| `vault_roots.macos` | path | | Vault folder on macOS, e.g. `/Users/me/Documents/MyVault`. |
| `vault_roots.linux` | path | | Vault folder on Linux. |
| `vault_roots.android` | path | | Vault folder on Android, e.g. `/storage/emulated/0/Documents/MyVault`. Used for download hints and local agents (Termux). |
| `vault_roots.ios` | path | | Vault location on iOS, e.g. `iCloud Drive/Obsidian/MyVault`. Used for download hints. |
| `sync.method` | enum | | `onedrive`, `google-drive`, `dropbox`, `icloud`, `obsidian-sync`, `syncthing`, `git` or `none`. Decides whether the Make route is possible. |
| `sync.cloud_path` | path | | The vault folder inside the cloud drive, starting with `/`, e.g. `/Obsidian/MyVault`. |
| `sync.connector_verified` | bool | `false` | `true` only after the assistant's own storage connector was tested to create plain `.md` files in the right folder. |
| `make.enabled` | bool | `false` | `true` once a Make scenario is set up and tested. |
| `make.declined` | bool | `false` | `true` if the user doesn't want the Make route; the skill then stops offering it. |
| `make.scenario` | text | | Scenario or tool name to call, e.g. `Obsidian vault: OneDrive API shell`. |
| `make.variant` | enum | | `onedrive-api-shell` or `vault-bridge` (see `make-setup.md`). |
| `folders.default` | path | `Conversations` | Vault-relative folder for notes that don't belong to a project. |
| `folders.projects_root` | path | empty | Vault-relative parent folder with one subfolder per project. Empty turns project routing off. |
| `folders.project_subfolder` | text | `Conversations` | Subfolder inside a project folder where notes go. |
| `folders.known_projects` | list | `[]` | Last known project folder names; used when the vault can't be listed live (download and copy routes). The skill refreshes it when it can list the folder. |
| `note.filename` | pattern | `YYYY-MM-DD {Topic}.md` | Filename pattern. |
| `note.usage` | bool | `true` | Add the session usage / cost estimate section and frontmatter fields. |
| `note.diagrams` | bool | `true` | Add Mermaid diagrams where they help understanding. |
| `note.extra_tags` | list | `[]` | Tags added to every note. |

All folder paths are relative to the vault root, use forward slashes and have no leading slash, except `sync.cloud_path`, which is a path inside the cloud drive.

## Example

```yaml
obsidian-conversation-config:
  vault_name: "MyVault"
  timezone: "Europe/Zurich"
  vault_roots:
    windows: 'C:\Users\me\Obsidian\MyVault'
    android: "/storage/emulated/0/Documents/MyVault"
  sync:
    method: "onedrive"
    cloud_path: "/Obsidian/MyVault"
    connector_verified: false
  make:
    enabled: true
    declined: false
    scenario: "Obsidian vault: OneDrive API shell"
    variant: "onedrive-api-shell"
  folders:
    default: "Inbox/Conversations"
    projects_root: "Projects"
    project_subfolder: "Conversations"
    known_projects: ["MyApp", "Writing", "Research"]
  note:
    filename: "YYYY-MM-DD {Topic}.md"
    usage: true
    diagrams: true
    extra_tags: []
```

Windows paths go in single quotes so YAML doesn't treat the backslashes as escapes.
