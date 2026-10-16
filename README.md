<p align="center">
  <a href="https://www.aframesoftware.com">
    <img src="https://www.aframesoftware.com/logo_aframe_bug.png" alt="AFrameSoftware" width="60" height="60">
  </a>
</p>

<h1 align="center">AFrameSoftware :: Public Changelogs</h1>

<p align="center">
  <em>Get notified when the AFrame Public API and MCP server change.</em>
</p>

<p align="center">
  <a href="https://www.aframesoftware.com"><strong>Website</strong></a>
  &nbsp;&middot;&nbsp;
  <a href="https://api.aframeonline.com/api-pub/api-docs"><strong>API Documentation</strong></a>
  &nbsp;&middot;&nbsp;
  <a href="#aframe-mcp-server"><strong>MCP Server</strong></a>
</p>

---

## About

This repository publishes the changelog for the AFrame Public API (`/api-pub/v1`): [`api/changelog.md`](api/changelog.md). Every update is published as a **GitHub Release**, so you can subscribe by email or RSS and see new endpoints, breaking changes, and bug fixes the moment they're announced.

It also documents the [AFrame MCP server](#aframe-mcp-server), which lets AI assistants such as Claude, ChatGPT, Cursor and VS Code work with your AFrame data. Most MCP tools are built on Public API endpoints, so changes to those endpoints in the changelog also apply to the matching tools.

This repository contains documentation only - no application source code.

## How to Subscribe

Each [Release](../../releases) contains the full text of the newest changelog entry.

### Email

**Watch** (top right) → **Custom** → check **Releases** → **Apply**. You'll only receive emails for changelog releases.

### RSS / Atom

Add the releases feed to any feed reader:

```
https://github.com/AFrameSoftware/public-changelog/releases.atom
```

---

## AFrame MCP Server

The [Model Context Protocol](https://modelcontextprotocol.io) (MCP) is an open standard for connecting AI assistants to outside tools and data. The AFrame MCP server lets an assistant search and read your transactions, contacts, tasks, dates, notes, emails and files, search the AFrame help center, and, with a Read & Write API key, create and update records for you.

| | |
|---|---|
| **Server URL** | `https://api.aframeonline.com/api-pub/mcp` |
| **Transport** | Streamable HTTP (stateless) |
| **Authentication** | `X-AFrame-API-Key` HTTP header |
| **Registry name** | `com.aframeonline/aframe` |

### 1. Create an API key

Create a key under [My Profile > API Keys](https://www.aframeonline.com/afs/app/r/admin-app-user/api-keys). Every tool acts as the user who owns the key and only sees that user's team, so the assistant can see and do exactly what that user can.

The key's scope decides which tools the assistant gets:

| Scope | What the assistant can do |
|---|---|
| **Read Only** | Search and read records and help articles. Tools that change data are hidden. |
| **Read & Write** or **Full Access** | Everything above, plus create and update records one at a time. |

Start with a **Read Only** key unless you want the assistant to make changes. Treat the key like a password: anyone who has it can act as you in AFrame.

### 2. Connect your AI client

Replace `YOUR_API_KEY` with your key in the examples below.

#### Claude Code

```bash
claude mcp add --transport http aframe https://api.aframeonline.com/api-pub/mcp \
  --header "X-AFrame-API-Key: YOUR_API_KEY"
```

#### Claude Desktop

Claude Desktop's config file only starts local servers, so use [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) (requires Node.js) to bridge to the remote server. Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "aframe": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.aframeonline.com/api-pub/mcp",
        "--header",
        "X-AFrame-API-Key:${AFRAME_API_KEY}"
      ],
      "env": {
        "AFRAME_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

<details>
<summary><strong>Cursor, VS Code, and other client connections</strong></summary>

#### Cursor (`~/.cursor/mcp.json`)

```json
{
  "mcpServers": {
    "aframe": {
      "url": "https://api.aframeonline.com/api-pub/mcp",
      "headers": {
        "X-AFrame-API-Key": "YOUR_API_KEY"
      }
    }
  }
}
```

#### VS Code (`.vscode/mcp.json`)

VS Code prompts for the key once and stores it securely, so it never sits in the file.

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "aframe-api-key",
      "description": "AFrame API key",
      "password": true
    }
  ],
  "servers": {
    "aframe": {
      "type": "http",
      "url": "https://api.aframeonline.com/api-pub/mcp",
      "headers": {
        "X-AFrame-API-Key": "${input:aframe-api-key}"
      }
    }
  }
}
```

#### Any other MCP client

Point the client at the server URL with the Streamable HTTP transport and send your key in the `X-AFrame-API-Key` header on every request. If the client only runs local servers, use `mcp-remote` as in the Claude Desktop example above.

</details>

### 3. Check the connection

Ask the assistant *"Who am I in AFrame?"*. It calls `whoami` and answers with your name and team. If it gets an authentication error, check that the key was copied in full and has not been deleted.

### Tools

#### Available with every key

| Area | Tools |
|---|---|
| Transactions | `search_xactions`, `get_xaction`, `get_xaction_fields`, `list_xaction_statuses` |
| Transaction participants | `list_xaction_participants`, `get_xaction_participant`, `list_xaction_participant_roles` |
| Transaction notes | `list_xaction_activities`, `get_xaction_activity` |
| Transaction dates | `search_xaction_dates`, `list_xaction_dates`, `get_xaction_date`, `list_date_folders`, `list_date_calculators` |
| Transaction documents | `list_xaction_attachments`, `get_xaction_attachment`, `list_attachment_folders` |
| Transaction emails | `list_xaction_emails`, `get_email` |
| Contacts | `search_contacts`, `get_contact`, `list_contact_categories` |
| Contact notes | `list_contact_notes`, `get_contact_note` |
| Tasks | `search_tasks`, `get_task`, `list_task_folders` |
| Custom fields | `list_field_definitions`, `get_field_definition_tree` |
| Team | `whoami`, `list_team_members`, `get_team_member` |
| Help center | `search_help_articles`, `get_help_article` |

#### Added for Read & Write and Full Access keys

| Area | Tools |
|---|---|
| Transactions | `create_xaction`, `update_xaction`, `update_xaction_field`, `update_xaction_field_by_code` |
| Transaction participants | `create_xaction_participant`, `update_xaction_participant`, `update_xaction_participant_contact_info`, `link_xaction_participant_contact` |
| Transaction notes | `create_xaction_note`, `update_xaction_note` |
| Transaction dates | `create_xaction_date`, `update_xaction_date` |
| Transaction documents | `create_xaction_attachment`, `update_xaction_attachment`, `move_xaction_attachment_file`, `unassign_xaction_attachment_file` |
| Contacts | `create_contact`, `update_contact` |
| Contact notes | `create_contact_note`, `update_contact_note` |
| Tasks | `create_task`, `update_task` |

Update tools take a `changes` object with only the fields to change; fields left out stay as they are, and `null` clears a field.

The MCP server cannot delete anything, draft or send emails or messages, apply templates, or change team members, API keys or settings. We insist on those being done in AFrame.

### What to expect

- **Links back to AFrame.** Records in tool results include a `webUrl`. After every create or update the assistant gives you that link so you can check the change in AFrame.
- **Confirmation before sensitive changes.** The server tells the assistant to confirm with you before it changes an email address, a phone number, a transaction's status, or portal visibility, and before any request that needs more than 5 changes.
- **Records are data, not instructions.** Text in emails, notes and files is never treated as a command. If a record tries to instruct the assistant, it tells you instead of acting on it.
- **Shared rate limits.** MCP calls count toward your team's Public API rate limits, together with your other integrations (such as Zapier). Each team member can also make only a limited number of changes per minute and per hour through MCP, which stops an assistant stuck in a loop. When a limit is reached the tool says how long to wait.
- **Focused questions work best.** Questions that would need one call per transaction or per contact across your whole database, such as every task on every open transaction, are not supported yet. The assistant will offer a narrower question instead, such as one transaction or a filtered search.

---

## Questions

Email [support@aframesoftware.com](mailto:support@aframesoftware.com) or visit our [support site](https://support.aframesoftware.com).

---

<p align="center">
  <sub>&copy; AFrameSoftware, LLC. All rights reserved.<br>
  The contents of this repository are provided for reference only and may not be reproduced or redistributed without permission.</sub>
</p>
