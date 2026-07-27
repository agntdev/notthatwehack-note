# Admin Note Reminder — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for admin-only access to manage a single note/reminder with audit logging. Only allowlisted admins can view/set the note, with a 90-day retention policy for logs. All interactions are direct and human-friendly.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram admins

## Success criteria

- Only allowlisted admins receive responses
- Note text is set/viewed correctly
- Audit log shows 20 most recent changes with pagination
- Unauthorized users get 'access denied' message

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu for admins
- **/view** (command, actor: user, command: /view) — Display current note text
- **/set** (command, actor: user, command: /set) — Replace note with new text (requires free-form input)
  - inputs: <text>
  - outputs: Confirmation message
- **/history** (command, actor: user, command: /history) — Show audit log entries with pagination
  - outputs: Paginated log entries

## Flows

### main_menu
_Trigger:_ /start

1. Display greeting and available commands

_Data touched:_ Admin

### view_note
_Trigger:_ /view

1. Display current note text

_Data touched:_ Note

### set_note
_Trigger:_ /set

1. Request new text input
2. Update note
3. Log change to audit

_Data touched:_ Note, Audit log entry

### view_history
_Trigger:_ /history

1. Fetch recent 20 log entries
2. Display paginated list

_Data touched:_ Audit log entry

### unauthorized
_Trigger:_ any message from non-admin

1. Send access denied message

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Admin** _(retention: persistent)_ — Telegram user with access to the bot
  - fields: telegram_id
- **Note** _(retention: persistent)_ — Current admin note/reminder text
  - fields: title, content
- **Audit log entry** _(retention: 90 days)_ — Record of note changes
  - fields: admin_id, timestamp, previous_content

## Integrations

- **Telegram** (required) — Admin-only messaging and command handling
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Set allowlisted admin IDs
- Configure initial note text
- Adjust audit log retention period

## Permissions & privacy

- All interactions restricted to allowlisted Telegram IDs
- Audit logs stored securely for 90 days

## Edge cases

- Non-admin users attempting commands
- Note text exceeding Telegram message limits
- Empty note content after /set
- Pagination beyond 20 history entries

## Required tests

- Verify unauthorized users get access denied
- Confirm /set updates note and creates audit log
- Validate /history shows 20 most recent entries with pagination

## Assumptions

- Owner's language is English
- Audit log retention defaults to 90 days
- Note content is a single string
