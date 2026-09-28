# Make setup guide (for the assistant)

Read this when the user wants to set up the Make route (route C), or when a Make save fails and you need to debug the scenario. Guide the user **one step at a time**: explain the step, wait until they confirm it's done, then continue. Adapt to their storage provider and to the device they're on (setting up Make is much easier on a computer; suggest that if they're on a phone).

## What to tell the user first

- Make is an automation service. Here it acts as a private bridge: the assistant calls a Make scenario, and the scenario writes the note into the cloud drive the vault syncs with. Their normal sync then delivers it to every device.
- **Cost:** Make's free plan is enough. It currently includes 1,000 credits per month and up to 2 active scenarios. One save uses a handful of credits, so that covers roughly a hundred saves or more per month. Plans and limits can change; see make.com/pricing.
- **They need:** a Make account, a Make connection to their cloud drive, one scenario, and the **Make connector** (sometimes called an app or integration) enabled in their AI assistant.
- **Who does what:** the user builds the scenario in Make's editor. If you have Make management tools available (they require a paid Make plan's extra scopes), you may offer to create the scenario for them instead.
- **Privacy:** note contents pass through Make. Recommend turning on **Data is confidential** in the scenario settings so contents aren't kept in the run history.

Only iCloud Drive and Obsidian Sync can't be reached this way (no Make integration). For those, say so and stay with routes B, D and E.

## Step 1: Make account

Sign up at make.com. The free plan is fine.

## Step 2: Connection to the cloud drive

In Make: **Connections › Add**, choose OneDrive, Google Drive or Dropbox, sign in with the account the vault syncs with.

## Step 3: Build the scenario

Pick the variant for their provider. Both start with **Scenarios › Start scenario** (receives the inputs) and end with **Scenarios › Return output** (sends the outputs back).

### OneDrive: `onedrive-api-shell`

A thin pass-through for Microsoft Graph requests. The repository has an importable blueprint at `make/onedrive/blueprint.json` (scenario editor › **⋯** menu › **Import blueprint**); after importing, the user selects their OneDrive connection in module 2 and defines the inputs and output.

| # | Module | Settings |
| --- | --- | --- |
| 1 | Scenarios › Start scenario | receives the inputs below |
| 2 | OneDrive › Make an API Call | URL `{{1.path}}`, Method `{{1.method}}`, Headers `{{1.header}}`, Query string `{{1.qs}}`, Body `{{1.body}}` |
| 3 | Scenarios › Return output | `data` = `{{2.body}}` |

Inputs: `path` (text, required), `method` (text, required), `header` (array of key/value text, optional), `qs` (array of key/value text, optional), `body` (any, optional). Output: `data` (any).

Optional safety filter between modules 1 and 2: `path` starts with `/v1.0/me/drive/root:<cloud_path>` (plus `/v1.0/me/drive/items/` if the `%20` rename fix should work). It limits the scenario to the vault.

Config: `make.variant: onedrive-api-shell`.

### Google Drive: `vault-bridge`

Google Drive works with folder IDs, so this scenario resolves the path itself and offers two actions.

| # | Module | Settings |
| --- | --- | --- |
| 1 | Scenarios › Start scenario | inputs `action` (text, required), `folder` (text, required), `filename` (text), `content` (text) |
| 2 | Google Drive › Get a Folder ID for a Path | Drive: My Drive; Folder Path: `<cloud_path>/{{1.folder}}` |
| 3 | Router | route "list": filter `action` = `list`; route "write": filter `action` = `write` |
| 4 list | Google Drive › Search for Files/Folders | Enter manually; Retrieve: Folders; Folder ID `{{2.id}}` |
| 5 list | Array aggregator → Scenarios › Return output | `status` = `ok`, `items` = folder names |
| 4 write | Google Drive › Search for Files/Folders | Enter manually; Retrieve: Files; Folder ID `{{2.id}}`; search names, exact term `{{1.filename}}`; limit 1 |
| 5 write | Router or filters | found → Return output `status` = `exists`; not found → continue |
| 6 write | Google Drive › Create a File from Text | My Drive; location mapped to `{{2.id}}`; File Name `{{1.filename}}`; Content `{{1.content}}`; **Convert to Google Docs: No** |
| 7 write | Scenarios › Return output | `status` = `ok`, `name` = `{{6.name}}` |

Outputs: `status` (text), `name` (text), `items` (array of text), `message` (text).

Pitfalls to mention:
- The target folders must exist; the user can create the `Conversations` folders once in Obsidian.
- "Convert to Google Docs" must be **No**, or Obsidian gets a Google Doc instead of a `.md` file.
- Search modules that return nothing must not stop the run: enable continuing on empty results, or filter on the bundle count.

Config: `make.variant: vault-bridge`.

### Dropbox: `vault-bridge`

Same inputs and outputs as Google Drive, but simpler because Dropbox uses paths: a folder-listing module for `list`, an upload module with overwriting turned off for `write` (a conflict → `status: exists`).

## Step 4: Scenario settings

- Scheduling: **On demand**. Only on-demand scenarios are exposed to AI assistants.
- The scenario must be **active** (switched on).
- Inputs and outputs defined exactly as above.
- A clear **description**, e.g. "Lists folders and writes Markdown notes in my Obsidian vault on OneDrive." You'll see it as the tool description.
- A short scenario name; it becomes part of the tool name.
- **Data is confidential** switched on.

## Step 5: Connect Make to the assistant

- Claude: *Settings › Connectors*, add **Make**, sign in, choose the organization. The scope "Run your scenarios" is enough (it's what the free plan offers). Then enable the connector for the chat.
- ChatGPT: add Make under apps/connectors, sign in, enable it for the conversation.
- Other assistants with MCP connector support can use Make's MCP server the same way.

The assistant may ask for permission before each run; that's expected. After connecting, the user may need to start a new chat before the scenario appears as a tool.

## Step 6: Test and record

1. Call the scenario's list action (or a Graph `GET` of the vault folder) and show the user the folder names you got back.
2. Save a short test note to the default folder.
3. Ask the user to check it appears in Obsidian on another device.
4. Update the configuration: `make.enabled: true`, `make.scenario: "<name>"`, `make.variant`, `sync.cloud_path`.

## Debugging

| Symptom | Cause and fix |
| --- | --- |
| No Make tool visible | Connector not enabled in this chat, or scenario not active / not on demand. Start a new chat after changing it. |
| Filename contains a literal `%20` (OneDrive) | The name was encoded twice. Pass names with plain spaces; rename via `PATCH /v1.0/me/drive/items/<id>`. |
| Graph: empty JSON payload on PATCH | Pass the body as a JSON string, not an object. |
| `409 Conflict` (OneDrive) | The name exists. Retry with ` 2`, ` 3`, … |
| Google Drive creates a Google Doc | "Convert to Google Docs" is on; switch it off. |
| Google Drive: folder not found | Path is case-sensitive and must exist; check `cloud_path`. |
| Tool call times out | Make answers within about 25 seconds over OAuth; the run may still finish. List the folder to check before retrying, so you don't create duplicates. |
