# Troubleshooting

| Symptom | Cause and fix |
| --- | --- |
| The assistant says it has no Make tool | The Make connector isn't enabled in this chat, or the scenario isn't active and on demand. |
| Scenario not visible in the assistant | Scheduling must be **On demand** and the scenario **active**; reconnect the connector after creating it. |
| OneDrive file name contains `%20` | The filename was URL-encoded twice. Pass names with normal spaces. The skill renames such files automatically via `PATCH /v1.0/me/drive/items/<id>`. |
| Graph reports an empty JSON payload on PATCH | The API shell passed the body as an object; pass it as a JSON string. |
| `409 Conflict` on upload | A note with that name exists. The skill retries with ` 2`, ` 3`, … |
| Google Drive creates a Google Doc | Set **Convert to Google Docs** to No. |
| Google Drive: folder not found | The folder path is case-sensitive and must exist; check the vault path in module 2 (see [Google Drive scenario](../make/google-drive/README.md)). |
| Tool call times out | Make answers within about 25 seconds over OAuth; the scenario still finishes in the background. Ask the assistant to check whether the file was created before retrying. |
| Note landed in the default folder | No project folder matched. Name the project ("save it to MyApp") or check `projects_root`. |
| Automate flow stops at "Expression true?" | See [Android: file downloads automatically](android-automate.md): no `{4}` or backslashes in the pattern. |
| The assistant asks the setup questions again | It found no stored configuration. Check that memory is on, or fill in the Configuration block in `SKILL.md`, or keep `save-conversation-config.md` in the vault root. See [configuration reference](../skills/save-conversation-to-obsidian/references/config-schema.md). |

Back to the [README](../README.md).
