## My First Project in n8n
# Workflow Overview

### Telegram Trigger
Starts when a user sends a message or presses a button.

### Context Normalizer
Cleans raw Telegram data into a consistent format.
- Extracts: `chatId`, `userId`, `username`, `type` (message or callback), `text`, and `callback`.

### Google Sheets (Append or Update)
Ensures the user exists in the sheet.
- If new: adds user with `Idle` state.
- If existing: keeps row.

### Get Row(s) in Sheet
Reads the user’s current state from the sheet.

### Context Manager (Code node)
Updates or calculates user state:
- If first time: set to `Idle`.
- If `/start`: switch to `Active`.
- If pressing `custom_command`: switch to `Custom Command`.
- Otherwise: default to `Active`.

### Switch Node
Routes flow depending on `userStatus`:
- **Idle** → Waits for `/start`
- **Active** → Executes server commands
- **Custom Command** → Asks for user input

### Idle + Start Command (IF node)
Handles `/start`:
- Updates sheet with `Active`
- Sends welcome message with command buttons.

### If Node (exit check)
Handles exit command:
- If `exit`: resets status to `Idle`
- Else: passes command to execution.

### Execute a Command (SSH)
Runs server command from user.

### Send Messages (Telegram nodes)
Respond to user with results, prompts, or errors.

### Google Sheets Updates
Context is written back every time the state changes, so next run continues correctly.

---

# Google Sheets Structure

| userID | status    | user-name |
|--------|-----------|-----------|
| 12345  | Active    |   Active  |
| 67890  | Idle      |   Idle    |

- **userID**: Telegram user’s ID → used as unique key.
- **user-name**: Telegram username.
- **status**: Bot context for this user (`Idle`, `Active`, `Custom Command`).

---

# How Context Works
- **New user** → Added to sheet with `Idle`.
- **Idle + /start** → Switch to `Active` and send command menu.
- **Active** → Bot listens for server commands and executes via SSH.
- **Custom Command** → User must type a custom command; then bot runs it.
- **Exit** → Resets status back to `Idle`.

---
