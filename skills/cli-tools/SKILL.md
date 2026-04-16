---
name: cli-tools
description: Provides instructions for using Obsidian CLI, Atlassian CLI (Jira + Confluence), so the agent can interact with these services via command-line tools instead of MCP servers. Use this skill when the user asks about Jira tickets, Confluence pages, or Obsidian notes, or when MCP servers are unavailable. Triggers include mentions of Jira issues, Confluence pages, Obsidian notes, or direct requests to use these CLIs.
---

# CLI Tools for Obsidian and Atlassian

You have access to the following command-line tools for interacting with external services. Use them via the Bash tool whenever the user asks about Jira issues, Confluence pages, or Obsidian notes.

**You do NOT need the user to say "use the CLI"** -- just use these tools naturally when the context calls for it. If the user says "what's the status of PROJ-1234?", run the Jira CLI. If they say "find my notes on X", run the Obsidian CLI.

---

## Obsidian CLI

Command: `obsidian`

Interacts with the user's Obsidian vault. Supports reading, searching, creating, and modifying notes.

### Read Operations (safe, no approval needed)

```bash
# Search for notes
obsidian search query="search terms"
obsidian search query="search terms" format=json

# Search with context (shows matching lines)
obsidian search:context query="search terms"

# Read a note's contents
obsidian read file="Note Name"
obsidian read path="folder/note.md"

# List files
obsidian files
obsidian files folder="subfolder"
obsidian files total

# List folders
obsidian folders

# Show file info
obsidian file file="Note Name"

# List tags
obsidian tags
obsidian tags file="Note Name"
obsidian tags counts sort=count

# Show note outline/headings
obsidian outline file="Note Name"

# List backlinks to a note
obsidian backlinks file="Note Name"

# List outgoing links from a note
obsidian links file="Note Name"

# List tasks
obsidian tasks
obsidian tasks todo
obsidian tasks done
obsidian tasks file="Note Name"

# Read daily note
obsidian daily:read
obsidian daily:path

# List bookmarks
obsidian bookmarks

# Read a property value
obsidian property:read name="property_name" file="Note Name"

# List properties
obsidian properties file="Note Name"

# List templates
obsidian templates

# Read template content
obsidian template:read name="template_name"

# Vault info
obsidian vault
obsidian vaults
```

### Write Operations (require approval)

```bash
# Create a new note
obsidian create name="Note Name" content="content here"
obsidian create path="folder/note.md" content="content here"
obsidian create name="Note Name" template="template_name"

# Append content to a note
obsidian append file="Note Name" content="content to append"

# Prepend content to a note
obsidian prepend file="Note Name" content="content to prepend"

# Append/prepend to daily note
obsidian daily:append content="content here"
obsidian daily:prepend content="content here"

# Set a property
obsidian property:set name="prop" value="val" file="Note Name"

# Remove a property
obsidian property:remove name="prop" file="Note Name"

# Delete a note
obsidian delete file="Note Name"

# Move/rename a note
obsidian move file="Note Name" to="new/path"
obsidian rename file="Note Name" name="New Name"

# Toggle/update a task
obsidian task file="Note Name" line=5 toggle
obsidian task file="Note Name" line=5 done

# Add a bookmark
obsidian bookmark file="path/to/note.md"
```

### Notes

- File resolution works like wikilinks: `file="Note Name"` finds by name, `path="folder/note.md"` is exact.
- Quote values with spaces: `name="My Note Title"`
- Use `\n` for newlines in content values.
- Target a specific vault with `vault="Vault Name"`.
- Most commands default to the active file when file/path is omitted.

---

## Atlassian CLI (Jira)

Command: `acli jira`

Interacts with Jira Cloud. Supports viewing, searching, creating, and editing work items (issues).

### Read Operations (safe, no approval needed)

```bash
# View a Jira issue
acli jira workitem view PROJ-1234
acli jira workitem view PROJ-1234 --json
acli jira workitem view PROJ-1234 --fields "summary,status,assignee,description,comment"
acli jira workitem view PROJ-1234 --fields "*all" --json

# Search issues with JQL
acli jira workitem search --jql "project = PROJ AND status = 'In Progress'"
acli jira workitem search --jql "assignee = currentUser() AND resolution = Unresolved"
acli jira workitem search --jql "project = PROJ" --fields "key,summary,status,assignee" --limit 20
acli jira workitem search --jql "project = PROJ" --json
acli jira workitem search --jql "project = PROJ" --count

# List projects
acli jira project list

# List boards
acli jira board list

# List sprints
acli jira sprint list --board-id 123

# View a filter
acli jira filter view FILTER_ID

# View fields
acli jira field list
```

### Write Operations (require approval)

```bash
# Create an issue
acli jira workitem create --project PROJ --type Task --summary "Issue title" --description "Description"
acli jira workitem create --project PROJ --type Bug --summary "Bug title" --priority High

# Edit an issue
acli jira workitem edit PROJ-1234 --summary "Updated title"
acli jira workitem edit PROJ-1234 --assignee "user@example.com"

# Transition an issue (change status)
acli jira workitem transition PROJ-1234 --transition "In Progress"
acli jira workitem transition PROJ-1234 --transition "Done"

# Add a comment
acli jira workitem comment add PROJ-1234 --body "Comment text"

# Assign an issue
acli jira workitem assign PROJ-1234 --assignee "user@example.com"

# Link issues
acli jira workitem link add --inward PROJ-1234 --outward PROJ-5678 --type "Blocks"

# Delete an issue
acli jira workitem delete PROJ-1234

# Clone an issue
acli jira workitem clone PROJ-1234
```

---

## Atlassian CLI (Confluence)

Command: `acli confluence`

Interacts with Confluence Cloud. Currently limited to viewing pages.

### Read Operations (safe, no approval needed)

```bash
# View a page
acli confluence page view PAGE_ID
acli confluence page view PAGE_ID --json

# List spaces
acli confluence space list
```

### Notes

- The Confluence CLI is limited -- it only supports viewing pages. For creating or editing pages, you would need to use the Confluence REST API directly with `curl`.
- Page IDs can be found in Confluence page URLs.

---

## General Usage Notes

- Always use `--json` when you need to parse structured output or extract specific fields.
- For Jira JQL queries, common fields: `project`, `status`, `assignee`, `priority`, `issuetype`, `summary`, `created`, `updated`, `resolution`.
- The Obsidian CLI requires Obsidian to be running on the machine.
- All write operations go through bash permission checks -- read operations are whitelisted for convenience.
