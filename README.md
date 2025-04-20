# DiscordFileServ

An IRC inspired Discord bot designed for browsing files and folders on the host server directly through Direct Messages (DMs). It uses interactive messages with buttons for navigation and selection, handling Discord's file size limits by temporarily copying/moving oversized items and providing a static link (if configured) with automatic cleanup.

## Features

*   **DM Interaction:** All commands and browsing happen exclusively in DMs with the bot.
*   **Paginated File Listing:** Uses the `!filelist` command to initiate browsing with paginated results (`BROWSE_ITEMS_PER_PAGE` items per page).
*   **Interactive Navigation:** Buttons for moving up directories (`Up`), navigating pages (`Prev`/`Next`), and cancelling the session (`Cancel`).
*   **Item Selection:** Select folders to navigate into or files to retrieve using the `Choose Item` button followed by typing the item number.
*   **New Message on Navigation:** Each folder navigation (up or down) sends a *new* message, preserving browse history within the DM channel. Old navigation messages have their buttons disabled.
*   **File Sending:** Sends individual files directly if they are under Discord's limit (25MB).
*   **Folder Zipping:** Zips the contents (files only) of the current directory on demand (`Get Folder (ZIP)` button) and sends the archive.
*   **Oversized Item Handling (Optional):**
    *   If a file or generated ZIP exceeds the 25MB limit, it's copied (files) or moved (ZIPs) to a configured temporary location (`FILE_LIMIT_FOLDER`).
    *   A message notifies the user about the oversized item and the temporary location.
    *   If a `FILE_SERV_URL` is configured, a static link based on this URL is provided for user access.
    *   **Requires `FILE_LIMIT_FOLDER` to be set in `.env`.**
*   **Persistent Auto-Deletion (Optional):**
    *   The *containing folder* created within `FILE_LIMIT_FOLDER` for an oversized item is automatically scheduled for deletion 2 hours after the item was copied/moved.
    *   Deletion schedules persist across bot restarts via `file_records.json`.
    *   **Requires `FILE_LIMIT_FOLDER` to be set and valid in `.env`.**
*   **Robust Error Handling:** Provides feedback for common issues like invalid paths, permissions errors, and ZIP failures. Only sends a final follow-up message on ZIP failure if the initial creation/send process had an issue.

## Prerequisites

*   [Node.js](https://nodejs.org/) (v16.x or higher recommended)
*   npm (usually included with Node.js)
*   A Discord Bot Token (create an application and bot user on the [Discord Developer Portal](https://discord.com/developers/applications))
    *   Ensure the bot has the `MESSAGE CONTENT` and `DIRECT MESSAGES` Privileged Gateway Intents enabled in the Developer Portal.

## Setup & Installation

1.  **Clone the repository:**
    ```bash
    git clone <your-repository-url>
    cd <repository-directory>
    ```

2.  **Install dependencies:**
    ```bash
    npm install
    ```

3.  **Configure the environment:**
    *   Copy the example environment file:
        ```bash
        cp .env.example .env
        ```
        *(If you don't have `.env.example`, create `.env` manually based on the example below)*
    *   Edit the `.env` file with your specific settings:
        ```dotenv
        # --- Required ---
        DISCORD_BOT_TOKEN=YOUR_BOT_TOKEN_HERE
        FILE_BROWSE_ROOT=/path/to/your/shareable/files

        # --- Optional (Enable Oversized Handling & Auto-Delete) ---
        # Needs write access for the bot.
        FILE_LIMIT_FOLDER=/path/to/temp/storage/for/large/files
        # Public URL mapping to FILE_LIMIT_FOLDER (if served by webserver)
        FILE_SERV_URL=https://your.domain.com/discord_temp
        ```
    *   **Important:**
        *   Replace `YOUR_BOT_TOKEN_HERE` with your actual bot token.
        *   Set `FILE_BROWSE_ROOT` to the directory you want users to browse. **Ensure this path is secure and doesn't expose sensitive system files.**
        *   If you want oversized file handling and auto-deletion:
            *   Set `FILE_LIMIT_FOLDER` to a valid path where the bot can create temporary subdirectories and files. **The bot needs write permissions here.** This directory should ideally be served by a web server if you want to use `FILE_SERV_URL`.
            *   Set `FILE_SERV_URL` to the corresponding public URL *only if* `FILE_LIMIT_FOLDER` is set and web-accessible.

4.  **Run the bot:**
    ```bash
    node index.js
    ```
    For development, you might use `nodemon`:
    ```bash
    npm install -g nodemon # If not already installed
    nodemon index.js
    ```
    For production, consider using a process manager like `pm2`:
    ```bash
    npm install -g pm2 # If not already installed
    pm2 start index.js --name discord-file-browser
    ```

## Usage

1.  **Start Browsing:** Send a Direct Message (DM) to the bot with the command:
    ```
    !filelist
    ```
2.  **Interact:** Use the buttons provided in the bot's message:
    *   `⬆️ Up`: Navigate to the parent directory.
    *   `⬅️ Prev` / `➡️ Next`: Navigate between pages of items in the current directory.
    *   `✅ Choose Item`: Prompts you to type the number corresponding to the item (file or folder) you want to select.
        *   Selecting a folder navigates into it.
        *   Selecting a file sends it (or handles it as oversized).
    *   `📦 Get Folder (ZIP)`: Creates a ZIP archive of all *files* in the *current* directory and sends it (or handles it as oversized). Does not work on the root directory.
    *   `✖️ Cancel`: Ends the current browsing session.

## Configuration Details (`.env`)

*   `DISCORD_BOT_TOKEN` (Required): Your bot's login token.
*   `FILE_BROWSE_ROOT` (Required): The base directory accessible by the bot. Path traversal outside this root is prevented.
*   `FILE_LIMIT_FOLDER` (Optional): Path for storing oversized items temporarily. **Enables oversized item handling and 2-hour auto-deletion.** Requires write permissions.
*   `FILE_SERV_URL` (Optional): Base URL for accessing items in `FILE_LIMIT_FOLDER`. Only useful if `FILE_LIMIT_FOLDER` is set and served by a web server.

## Persistence

*   The bot uses a `file_records.json` file in its root directory to keep track of scheduled deletions for oversized items placed in the `FILE_LIMIT_FOLDER`. This allows deletions to occur even if the bot restarts. This file is automatically created and managed.

## Security Considerations

*   **`FILE_BROWSE_ROOT`:** Choose this path carefully. **Never set it to `/`, `C:\`, or other sensitive system-wide directories.** The bot has checks against path traversal (`../`), but limiting the initial scope is the best security practice.
*   **`FILE_LIMIT_FOLDER`:** Ensure this path is secure and ideally outside of critical system areas. If using `FILE_SERV_URL`, configure your web server securely to only serve files from this directory.
*   **Bot Token:** Keep your `DISCORD_BOT_TOKEN` secret. Do not commit it to version control (hence the `.gitignore` entry for `.env`).
