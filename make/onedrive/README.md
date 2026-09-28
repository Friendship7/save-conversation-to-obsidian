# Make scenario for OneDrive

The OneDrive version is a thin "API shell": it forwards a Microsoft Graph request and returns the answer. This is the version I use daily.

## Modules

| # | Module | Settings |
| --- | --- | --- |
| 1 | **Scenarios › Start scenario** | receives the inputs below |
| 2 | **OneDrive › Make an API Call** | URL `{{1.path}}`, Method `{{1.method}}`, Headers `{{1.header}}`, Query string `{{1.qs}}`, Body `{{1.body}}` |
| 3 | **Scenarios › Return output** | `data` = `{{2.body}}` |

Scenario inputs:

| Name | Type | Required |
| --- | --- | --- |
| `path` | Text | yes |
| `method` | Text | yes |
| `header` | Array of `key` / `value` (text) | no |
| `qs` | Array of `key` / `value` (text) | no |
| `body` | Any | no |

Scenario output: `data` (Any).

## Quick start: import the blueprint

1. In Make, create a new scenario and open the **⋯** menu › **Import blueprint**.
2. Choose [`blueprint.json`](blueprint.json) from this folder.
3. In module 2, select your OneDrive connection.
4. Define the scenario inputs and output from the tables above ([`interface.json`](interface.json) lists them in machine-readable form).
5. Continue with the [scenario settings](../../docs/make-setup.md#step-4-scenario-settings).

The blueprint already has **Data is confidential** switched on, so note contents aren't kept in the run history.

## Calls the skill makes

The assistant makes Graph calls like these (you never type them yourself):

```http
GET /v1.0/me/drive/root:/Obsidian/MyVault/Projects:/children?$select=name,folder
```

```http
PUT /v1.0/me/drive/root:/Obsidian/MyVault/Projects/MyApp/Conversations/2026-09-27 Sync Conflict Fix.md:/content?@microsoft.graph.conflictBehavior=fail
Content-Type: text/markdown; charset=utf-8
```

## Optional safety filter

The API shell can reach your whole OneDrive. To restrict it to the vault, add a filter between modules 1 and 2: *`path` starts with `/v1.0/me/drive/root:/Obsidian/MyVault`* (plus a second condition for `/v1.0/me/drive/items/` if you want the automatic rename fix described in [Troubleshooting](../../docs/troubleshooting.md)).

## Configuration

Set `make.variant: onedrive-api-shell` in the configuration.

Back to the [Make setup guide](../../docs/make-setup.md).
